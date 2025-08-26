# 使用 BeautifulSoup、requests 來擷取 Wayback Machine 上的網頁內容並存檔

## 簡介

這支程式的目的是協助使用者擷取特定年月、類別的文章列表，
透過 Wayback CDX API 與`requests`、`BeautifulSoup`取得文章 url、Wayback 快取的網址、文章 id，能夠初步查看總共有哪些文章。

為了加快速度，使用了多執行緒來加快程式運作。

```{tip}
以下皆是在Python3中執行。
在開始之前，建議先開一個虛擬環境，避免衝突。
```

    python3 -m venv .venv  # 建立虛擬環境 (資料夾名稱可自訂，一般用 .venv 或 venv)
    source .venv/bin/activate # 啟動虛擬環境 macOS / Linux
    .venv\Scripts\activate # Windows

`````
```

````{note}
這支程式會使用 `requests`、`Beautiful Soup`這個套件與 Wayback Machine 做互動。
在開始之前，請先安裝：
    ````
        pip install requests BeautifulSoup
    ````
````

```python
import os
import re
import time
import json
import zipfile
import requests
import pandas as pd
from bs4 import BeautifulSoup
from datetime import datetime
from concurrent.futures import ThreadPoolExecutor, as_completed
from urllib.parse import urlparse
```

```python
# basic setting
INPUT_CSV = "example.csv"
URL_FIELD = "uri"
ID_FIELD = "id"
THREADS = 5
N_LIMIT = 5                  # None == all
OUTPUT_PREFIX = "output_"
SKIP_FILE_NAME = "skipped_urls.txt"
```

```python
# tools
def _only_date(s: str) -> str:
    """Normalize time string to YYYY-MM-DD. Return '' if parsing fails."""
    if not s:
        return ""
    s = s.strip()
    if s.isdigit() and len(s) == 14:  # CDX: YYYYmmddHHMMSS
        try:
            return datetime.strptime(s, "%Y%m%d%H%M%S").strftime("%Y-%m-%d")
        except Exception:
            pass
    for fmt in ("%Y-%m-%dT%H:%M:%S%z",
                "%Y-%m-%dT%H:%M:%S",
                "%Y-%m-%d %H:%M:%S",
                "%Y-%m-%d"):
        try:
            dt = datetime.strptime(s.replace("Z", "+0000"), fmt)
            return dt.strftime("%Y-%m-%d")
        except Exception:
            continue
    if len(s) >= 10 and s[4] == "-" and s[7] == "-":
        return s[:10]
    return ""

def _first_path_segment(raw_url: str) -> str:
    """Return first URL path segment (e.g., /local/20200729/... -> local)."""
    try:
        p = urlparse(raw_url)
        parts = [seg for seg in p.path.split("/") if seg]
        return parts[0] if parts else ""
    except Exception:
        return ""

def _norm_img_url(src: str) -> str:
    """Normalize to absolute Wayback URL."""
    if not src:
        return ""
    src = src.strip()
    if src.startswith("//"):
        return "https:" + src
    if src.startswith("/"):
        return "https://web.archive.org" + src
    if not src.startswith("http"):
        return "https://web.archive.org" + ("/" + src if not src.startswith("/") else src)
    return src

def _content_type_from_ext(filename: str) -> str:
    """Map file extension to MIME type."""
    ext = os.path.splitext(filename)[1].lower().lstrip(".")
    if ext in ("jpg", "jpeg"):
        return "image/jpeg"
    if ext in ("png", "gif", "webp", "bmp", "svg"):
        return f"image/{ext}"
    return "image/jpeg"

def _normalize_text(s: str) -> str:
    """Light normalization for pattern matching (unify slashes, remove extra spaces)."""
    if not s:
        return ""
    s = s.replace("／", "/").replace("　", " ").strip()
    s = re.sub(r"\s+", " ", s)
    return s

def _extract_by_loc_text(text: str) -> tuple[str, str]:
    """
    Extract 'by' (reporter) and 'located' (place/desk) from text such as:
      - 記者周庭慶／台中報導
      - 地方中心周庭慶／台中報導
      - 周庭慶／台中報導
      - （…）變體皆可；允許 / 與 ／；報導/報道 都接受
    Returns (by, located) or ("","") if not found.
    """
    t = _normalize_text(text)

    # Common patterns
    patterns = [
        # Optional prefix + NAME / PLACE 報導|報道
        r'(?:記者|地方中心|採訪中心|特派|特約)?\s*([\u4e00-\u9fa5A-Za-z·．\.\s]{2,20})\s*/\s*([\u4e00-\u9fa5A-Za-z·\.\s]{1,10})\s*報[導道]',
        # NAME / PLACE 報導|報道
        r'([\u4e00-\u9fa5A-Za-z·．\.\s]{2,20})\s*/\s*([\u4e00-\u9fa5A-Za-z·\.\s]{1,10})\s*報[導道]',
    ]
    for pat in patterns:
        m = re.search(pat, t)
        if m:
            by = m.group(1).strip(" ，,。.!?（）()")
            located = m.group(2).strip(" ，,。.!?（）()")
            return by, located

    # Fallback: only "(PLACE報導)" without a clear name
    m = re.search(r'([\u4e00-\u9fa5A-Za-z·\.\s]{1,10})\s*報[導道]', t)
    if m:
        return "", m.group(1).strip(" ，,。.!?（）()")

    return "", ""

```

```python
# Wayback Scraper
class WaybackScraper:
    def __init__(self, img_root_dir):
        self.session = requests.Session()
        self.session.headers.update({'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'})
        self.img_root_dir = img_root_dir
        os.makedirs(self.img_root_dir, exist_ok=True)

    def get_latest_snapshot(self, url):
        """Query CDX for the latest successful snapshot."""
        api = "https://web.archive.org/cdx/search/cdx"
        params = {
            'url': url,
            'output': 'json',
            'filter': 'statuscode:200',
            'sort': 'reverse',
            'limit': 1
        }
        try:
            r = self.session.get(api, params=params, timeout=20)
            r.raise_for_status()
            data = r.json()
            if len(data) > 1:
                row = data[1]
                return {
                    'timestamp': row[1],  # YYYYmmddHHMMSS
                    'original_url': row[2],
                    'wayback_url': f"https://web.archive.org/web/{row[1]}/{row[2]}"
                }
        except Exception as e:
            print(f"[CDX error] {url} → {e}")
        return None

    def _extract_published_date(self, soup, wayback_ts: str) -> str:
        """Use meta[article:published_time] or <time>; fallback to Wayback timestamp."""
        meta = soup.find("meta", attrs={"property": "article:published_time"})
        if meta and meta.get("content"):
            d = _only_date(meta.get("content"))
            if d:
                return d
        t = soup.find("time")
        if t:
            d = _only_date(t.get("datetime") or t.get_text(strip=True))
            if d:
                return d
        return _only_date(wayback_ts)

    def scrape_article_payload(self, wayback_url, raw_url, item_id, wayback_ts):
        """
        Parse snapshot:
          - title
          - body text (joined <p>)
          - published date
          - subject (first path segment)
          - images: FIRST <img> = cover, remaining = other
          - image alts map
          - by / located extraction (author/byline zones → fallback to title+first body chunk)
        """
        try:
            resp = self.session.get(wayback_url, timeout=20)
            resp.raise_for_status()
            # prefer lxml; fallback to html.parser
            parser = "lxml"
            try:
                import lxml  # noqa
            except Exception:
                parser = "html.parser"
            soup = BeautifulSoup(resp.content, parser)

            # title
            h1 = soup.find("h1") or soup.find("title")
            title = h1.get_text(strip=True) if h1 else ""

            # bodies
            paragraphs = [p.get_text().strip() for p in soup.find_all("p") if p.get_text().strip()]
            body_text = "\n".join(paragraphs)

            # date
            published_date = self._extract_published_date(soup, wayback_ts)

            # collect all <img> in order; first is cover, rest are other
            all_imgs, seen = [], set()
            for img in soup.find_all("img"):
                src = _norm_img_url(img.get("src"))
                if not src or src in seen:
                    continue
                if "." not in os.path.basename(src):  # crude filter: must look like a file
                    continue
                all_imgs.append(src)
                seen.add(src)
            cover_urls = all_imgs[:1]
            other_urls = all_imgs[1:]

            # alts
            img_alts = {}
            for img in soup.find_all("img"):
                src = _norm_img_url(img.get("src"))
                if not src:
                    continue
                img_alts[src] = (img.get("alt") or "").strip()

            # by/located: search in byline zones first
            meta_zone_texts = []
            for sel in [
                {"name": "span", "class_": re.compile("author|byline")},
                {"name": "div",  "class_": re.compile("author|byline")},
                {"name": "p",    "class_": re.compile("author|byline")},
            ]:
                for el in soup.find_all(sel["name"], class_=sel["class_"]):
                    meta_zone_texts.append(el.get_text(" ", strip=True))
            by, located = _extract_by_loc_text("  ".join(meta_zone_texts))

            # fallback: try title + first body chunk if still empty
            if not by and not located:
                head_and_lead = (title + " " + " ".join(paragraphs[:3]))[:800]
                by, located = _extract_by_loc_text(head_and_lead)

            subject_name = _first_path_segment(raw_url)

            return {
                "title": title,
                "body_text": body_text,
                "published_date": published_date,
                "cover_urls": cover_urls,
                "other_urls": other_urls,
                "img_alts": img_alts,
                "subject_name": subject_name,
                "by": by,
                "located": located,
            }
        except Exception as e:
            print(f"[failed capture] {wayback_url} → {e}")
            return None

    def download_images(self, urls, item_id, label, start_at=1):
        """
        Download images to images/{item_id}/img/
        Filename format: {item_id}_{label}_{N}.{ext}
          - cover uses start_at=1  → {item_id}_cover_1.png
          - other can start at 2   → {item_id}_other_2.png (if a cover exists)
        Returns: list of (url, filename)
        """
        saved = []
        item_img_dir = os.path.join(self.img_root_dir, item_id, "img")
        os.makedirs(item_img_dir, exist_ok=True)

        for i, url in enumerate(urls, 1):
            try:
                ext = os.path.splitext(url)[1].split("?")[0].lower()
                if ext not in [".jpg", ".jpeg", ".png", ".gif", ".webp", ".bmp", ".svg"]:
                    ext = ".jpg"
                seq = start_at + i - 1
                fname = f"{item_id}_{label}_{seq}{ext}"
                fpath = os.path.join(item_img_dir, fname)

                r = self.session.get(url, timeout=20)
                if r.status_code == 200:
                    with open(fpath, "wb") as f:
                        f.write(r.content)
                    saved.append((url, fname))
                else:
                    print(f"[Download {r.status_code}] {url}")
            except Exception as e:
                print(f"[Download error] {url} → {e}")
        return saved
```

```python
# per-row
def process_row(row, scraper: WaybackScraper, skip_file_path: str):
    raw_url = row[URL_FIELD]
    item_id = str(row[ID_FIELD])

    snap = scraper.get_latest_snapshot(raw_url)
    if not snap:
        with open(skip_file_path, "a", encoding="utf-8") as f:
            f.write(raw_url + "\n")
        return None

    parsed = scraper.scrape_article_payload(
        wayback_url=snap['wayback_url'],
        raw_url=raw_url,
        item_id=item_id,
        wayback_ts=snap['timestamp']
    )
    if not parsed:
        with open(skip_file_path, "a", encoding="utf-8") as f:
            f.write(raw_url + "\n")
        return None

    # download images with desired numbering
    cover_saved = scraper.download_images(parsed["cover_urls"], item_id, "cover", start_at=1)
    other_start = 2 if cover_saved else 1
    other_saved = scraper.download_images(parsed["other_urls"], item_id, "other", start_at=other_start)

    # build associations: first = "cover"; others = "other_1", "other_2", ...
    associations = []

    def _mk_assoc(name_label, url_fname_tuple):
        url, fname = url_fname_tuple
        href = f"./images/{item_id}/img/{fname}"
        ctype = _content_type_from_ext(fname)
        return {
            "name": name_label,
            "uri": url,
            "type": "picture",
            "headlines": [{"value": parsed["img_alts"].get(url, "")}],
            "renditions": [{"href": href, "contentType": ctype}]
        }

    if cover_saved:
        associations.append(_mk_assoc("cover", cover_saved[0]))
    for i, tup in enumerate(other_saved, 1):
        associations.append(_mk_assoc(f"other_{i}", tup))

    # dates
    firstcreated = parsed["published_date"] or _only_date(snap['timestamp'])
    versioncreated = _only_date(snap['timestamp'])  # Wayback snapshot date
    contentcreated = firstcreated
    subjects = [{"name": parsed["subject_name"]}] if parsed["subject_name"] else []

    ninjs_obj = {
        "uri": raw_url,
        "standard": {
            "name": "ninjs",
            "version": "3.0",
            "schema": "https://www.iptc.org/std/ninjs/ninjs-schema_3.0.json"
        },
        "firstcreated": firstcreated,
        "versioncreated": versioncreated,
        "contentcreated": contentcreated,
        "type": "text",
        "language": "zh-Hant-TW",
        "headlines": [{"role": "main", "value": parsed["title"]}],
        "subjects": subjects,
        "bodies": [{"role": "main", "contentType": "text/plain", "value": parsed["body_text"]}],
        "associations": associations,
        "by": parsed.get("by", ""),          # <-- filled from extractor
        "located": parsed.get("located", ""),# <-- filled from extractor
        "altids": [{"role": "internal", "value": item_id}]
    }
    return ninjs_obj
```

```python
#  main
if __name__ == "__main__":
    t0 = time.time()
    print("activetime:", datetime.now().strftime("%Y-%m-%d %H:%M:%S"))

    ts = datetime.now().strftime("%Y%m%d_%H%M%S")
    out_dir = f"{OUTPUT_PREFIX}{ts}"
    os.makedirs(out_dir, exist_ok=True)

    img_root_dir = os.path.join(out_dir, "images")
    skip_file_path = os.path.join(out_dir, SKIP_FILE_NAME)
    os.makedirs(img_root_dir, exist_ok=True)
    if os.path.exists(skip_file_path):
        os.remove(skip_file_path)

    df = pd.read_csv(INPUT_CSV)
    if N_LIMIT:
        df = df.head(N_LIMIT)

    print("output_index:", out_dir)
    scraper = WaybackScraper(img_root_dir=img_root_dir)

    ninjs_list = []
    with ThreadPoolExecutor(max_workers=THREADS) as executor:
        tasks = {executor.submit(process_row, row, scraper, skip_file_path): row for _, row in df.iterrows()}
        for i, f in enumerate(as_completed(tasks), 1):
            obj = f.result()
            if obj:
                ninjs_list.append(obj)
                print(f"[{i}] ✔ {obj['uri']}")
            else:
                print(f"[{i}] ✘ skipped")

    # ninjs.json（array）
    if ninjs_list:
        with open(os.path.join(out_dir, "ninjs.json"), "w", encoding="utf-8") as f:
            json.dump(ninjs_list, f, ensure_ascii=False, indent=2)
        print("already output ninjs.json")
    else:
        print("failed")

    # zip all files
    zip_name = out_dir + ".zip"
    with zipfile.ZipFile(zip_name, 'w', zipfile.ZIP_DEFLATED) as z:
        for root, _, files in os.walk(out_dir):
            for file in files:
                fp = os.path.join(root, file)
                z.write(fp, os.path.relpath(fp, start=os.path.dirname(out_dir)))
    print(f"zipped: {zip_name}")

    # skip count
    skip_count = 0
    if os.path.exists(skip_file_path):
        with open(skip_file_path, encoding="utf-8") as f:
            skip_count = len([l for l in f if l.strip()])
    print(f"total {skip_count} URL skipped")
    print("cost: %.2f s" % (time.time() - t0))

```
`````
