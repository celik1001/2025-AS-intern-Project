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

```{note}
這支程式會使用 `requests`、`Beautiful Soup`這個套件與 Wayback Machine 做互動。
在開始之前，請先安裝：
    ````
        pip install requests BeautifulSoup
    ````
```

```python
import os
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
# settings
INPUT_CSV = "example.csv"  # Input file with article URLs
URL_FIELD = "uri"
ID_FIELD = "id"
THREADS = 5
N_LIMIT = 10                    # None = all rows
OUTPUT_PREFIX = "output_"
SKIP_FILE_NAME = "skipped_urls.txt"
```

```python
# Utility functions
def _only_date(s: str) -> str:
    """
    Normalize different datetime formats into YYYY-MM-DD.
    Supported inputs include:
    - '20240207123456' (CDX timestamp)
    - '2024-02-07T12:34:56+08:00'
    - '2024-02-07 12:34:56'
    - '2024-02-07'
    Returns empty string if parsing fails.
    """
    if not s:
        return ""
    s = s.strip()
    if s.isdigit() and len(s) == 14:  # CDX format
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
    """
    Return the first path segment of a URL.
    Example: /local/20200729/... -> 'local'
    """
    try:
        p = urlparse(raw_url)
        parts = [seg for seg in p.path.split("/") if seg]
        return parts[0] if parts else ""
    except Exception:
        return ""


def _ensure_dir(path: str):
    os.makedirs(path, exist_ok=True)
```

```python
# Wayback Scraper
class WaybackScraper:
    def __init__(self, img_root_dir):
        self.session = requests.Session()
        self.session.headers.update({'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'})
        self.img_root_dir = img_root_dir
        _ensure_dir(self.img_root_dir)

    def get_latest_snapshot(self, url):
        """
        Query the CDX API for the latest snapshot of a given URL.
        Returns timestamp, original URL, and constructed Wayback URL.
        """
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
                    'timestamp': row[1],
                    'original_url': row[2],
                    'wayback_url': f"https://web.archive.org/web/{row[1]}/{row[2]}"
                }
        except Exception as e:
            print(f"[CDX error] {url} → {e}, params={params}")
        return None

    def _extract_published_date(self, soup, wayback_ts: str) -> str:
        """
        Try to extract a published date from <meta property='article:published_time'>
        or <time datetime>. Fall back to Wayback timestamp if none are found.
        """
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

    def scrape_article(self, wayback_url: str, raw_url: str, wayback_ts: str):
        """
        Parse a Wayback snapshot page to extract:
        - title
        - body text (concatenated <p> tags)
        - publication date
        - category (from URL path)
        - image URLs (first = cover, rest = others)
        """
        try:
            resp = self.session.get(wayback_url, timeout=20)
            resp.raise_for_status()
            soup = BeautifulSoup(resp.content, 'html.parser')

            # Title
            title_tag = soup.find("h1") or soup.find("title")
            title = title_tag.get_text(strip=True) if title_tag else ""

            # Body text
            bodies = "\n".join(
                p.get_text().strip()
                for p in soup.find_all("p")
                if p.get_text().strip()
            )

            # Dates
            firstcreated = self._extract_published_date(soup, wayback_ts)
            contentcreated = firstcreated
            versioncreated = _only_date(wayback_ts)

            # Collect images
            all_img_urls = []
            for img in soup.find_all("img"):
                src = img.get("src")
                if not src:
                    continue
                if src.startswith("//"):
                    src = "https:" + src
                elif not src.startswith("http"):
                    src = f"https://web.archive.org{src}"
                if src not in all_img_urls:
                    all_img_urls.append(src)

            subject = _first_path_segment(raw_url)

            return {
                "title": title,
                "bodies": bodies,
                "firstcreated": firstcreated,
                "versioncreated": versioncreated,
                "contentcreated": contentcreated,
                "subject": subject,
                "all_img_urls": all_img_urls
            }
        except Exception as e:
            print(f"[Parse error] {wayback_url} (raw={raw_url}) → {e}")
            return None

    def download_images(self, urls, item_id):
        """
        Download all images for an article.
        Saved to images/{item_id}/img/
        Filenames: {item_id}_cover_1.jpg (first), {item_id}_other_2.jpg (rest).
        Returns a list of filenames aligned with input URLs.
        """
        saved_files = []
        item_img_dir = os.path.join(self.img_root_dir, item_id, "img")
        _ensure_dir(item_img_dir)

        for i, url in enumerate(urls, 1):
            try:
                ext = os.path.splitext(url)[1].split("?")[0].lower()
                if ext not in ['.jpg', '.jpeg', '.png', '.gif', '.webp']:
                    ext = '.jpg'
                label = "cover" if i == 1 else "other"
                fname = f"{item_id}_{label}_{i}{ext}"
                fpath = os.path.join(item_img_dir, fname)

                r = self.session.get(url, timeout=20)
                if r.status_code == 200:
                    with open(fpath, "wb") as f:
                        f.write(r.content)
                    saved_files.append(fname)
                else:
                    print(f"[Download failed {r.status_code}] {url}")
                    saved_files.append("")
            except Exception as e:
                print(f"[Download error] {url} → {e}")
                saved_files.append("")
        return saved_files
```
`````
