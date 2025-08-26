# 使用 CDX api 查找 時間範圍內的網址內容

## 簡介

這支程式的目的是協助使用者獲得特定年月、類別的文章列表，
透過 Wayback CDX API 與`requests`取得文章 url、Wayback 快取的網址、文章 id，能夠初步查看總共有哪些文章。

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
這支程式會使用 `requests` 這個套件與 Wayback Machine 做互動。
在開始之前，請先安裝：
    ```
    pip install requests
    ```
`````

```python
import requests, csv, time, random
from concurrent.futures import ThreadPoolExecutor, as_completed
from threading import Lock
import logging
import os
from collections import defaultdict
```

```python
# set log
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(message)s')
logger = logging.getLogger(__name__)
```

```python
API = "https://web.archive.org/cdx/search/cdx"
class MultiCategoryWaybackScraper:
    def __init__(self, max_workers=2, max_retries=3, timeout=60):
        self.max_workers = max_workers
        self.max_retries = max_retries
        self.timeout = timeout
        self.session = requests.Session()
        self.session.headers.update({"Accept-Encoding": "gzip"})
        self.lock = Lock()
        self.stats = defaultdict(lambda: {"success": 0, "failed": 0, "total_items": 0})

    def fetch_with_retry(self, params, info=""):
        """retry request"""
        for attempt in range(self.max_retries):
            try:
                response = self.session.get(API, params=params, timeout=self.timeout)
                response.raise_for_status()
                data = response.json()

                if not data:
                    logger.warning(f"{info} - Empty response on attempt {attempt + 1}")
                    continue

                header, *items = data
                logger.info(f"{info} - Success: {len(items)} items")
                return items

            except requests.exceptions.RequestException as e:
                wait_time = random.uniform(2, 5) * (attempt + 1)
                logger.warning(f"{info} - Attempt {attempt + 1} failed: {e}")

                if attempt < self.max_retries - 1:
                    logger.info(f"{info} - Waiting {wait_time:.1f}s before retry")
                    time.sleep(wait_time)
                else:
                    logger.error(f"{info} - All attempts failed")

            except Exception as e:
                logger.error(f"{info} - Unexpected error: {e}")
                break

        return []

    def fetch_single_day(self, date, category):
        """catch single day"""
        params = {
            "url": f"www.appledaily.com.tw/{category}/{date}/*", # set catagory and date
            "output": "json",
            "fl": "timestamp,original",
            "filter": ["statuscode:200", "mimetype:text/html"],
            "collapse": "original",
            "limit": 5000,
        }

        info = f"[{category}] {date}"
        items = self.fetch_with_retry(params, info)

        with self.lock:
            if items:
                self.stats[category]["success"] += 1
                self.stats[category]["total_items"] += len(items)
            else:
                self.stats[category]["failed"] += 1

        results = []
        for ts, orig in items:
            try:
                clean = orig.split('?', 1)[0]
                article_id = clean.rstrip('/').split('/')[-1]

                if not article_id or article_id == category:
                    continue

                article_url = f"https://www.appledaily.com.tw/{category}/{date}/{article_id}/"
                results.append((date, article_id, article_url, category))
            except Exception as e:
                logger.warning(f"{info} - Error processing item {orig}: {e}")

        return results

    def fetch_category_data(self, months, category):
        """catch all month of category"""
        logger.info(f"Starting category [{category}] for months: {months}")
        category_start = time.time()

        # generate all dates
        all_dates = []
        for month in months:
            for day in range(1, 32):
                all_dates.append(f"{month}{day:02d}")

        all_rows = []
        failed_requests = []

        # multiThreads
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            future_to_info = {
                executor.submit(self.fetch_single_day, date, category): (date, category)
                for date in all_dates
            }

            completed = 0
            total = len(future_to_info)

            for future in as_completed(future_to_info):
                date, cat = future_to_info[future]
                completed += 1

                try:
                    results = future.result()
                    if results:
                        all_rows.extend(results)
                    else:
                        failed_requests.append((date, cat))

                    # Progress bar process per 50
                    if completed % 50 == 0:
                        logger.info(f"[{category}] Progress: {completed}/{total} ({completed/total*100:.1f}%)")

                except Exception as e:
                    logger.error(f"[{category}] {date} - Future exception: {e}")
                    failed_requests.append((date, cat))

        # retry failed request
        if failed_requests:
            logger.info(f"[{category}] Retrying {len(failed_requests)} failed requests")
            retry_count = 0
            for date, cat in failed_requests[:]:  # copy failed lists
                time.sleep(random.uniform(3, 6))  # set retry timeout
                retry_results = self.fetch_single_day(date, cat)
                if retry_results:
                    all_rows.extend(retry_results)
                    failed_requests.remove((date, cat))
                    retry_count += 1

                # Retry progress
                if retry_count % 10 == 0:
                    logger.info(f"[{category}] Retry progress: {retry_count}")

        category_elapsed = time.time() - category_start
        logger.info(f"[{category}] Completed in {category_elapsed:.2f}s, "
                   f"got {len(all_rows)} articles, {len(failed_requests)} failed")

        return all_rows, failed_requests

    def save_category_to_csv(self, rows, category, months):
        """saved sing. category"""
        if not rows:
            logger.warning(f"[{category}] No data to save")
            return 0

        # unique articles
        unique_articles = {}
        for date, article_id, url, cat in rows:
            key = f"{cat}_{article_id}"
            if key not in unique_articles:
                unique_articles[key] = (date, article_id, url, cat)
            else:
                existing_date = unique_articles[key][0]
                if date < existing_date:
                    unique_articles[key] = (date, article_id, url, cat)

        # sorted and write into file
        sorted_articles = sorted(unique_articles.values())

        month_range = f"{months[0]}-{months[-1]}"
        filename = f"appledaily_{category}_{month_range}.csv"

        with open(filename, "w", newline="", encoding="utf-8") as f:
            writer = csv.writer(f)
            writer.writerow(["date", "article_id", "url", "category"])
            for row in sorted_articles:
                writer.writerow(row)

        logger.info(f"[{category}] Saved {len(sorted_articles)} unique articles to {filename}")
        return len(sorted_articles), filename

    def save_all_categories_to_csv(self, all_data, months):
        """save all categories to csv"""
        if not all_data:
            logger.warning("No data to save to combined file")
            return 0, ""

        # combine all data and output to csv
        unique_articles = {}
        for rows in all_data.values():
            for date, article_id, url, category in rows:
                key = f"{category}_{article_id}"
                if key not in unique_articles:
                    unique_articles[key] = (date, article_id, url, category)
                else:
                    existing_date = unique_articles[key][0]
                    if date < existing_date:
                        unique_articles[key] = (date, article_id, url, category)

        # sorted and write into file
        sorted_articles = sorted(unique_articles.values())

        month_range = f"{months[0]}-{months[-1]}"
        filename = f"appledaily_ALL_CATEGORIES_{month_range}.csv"

        with open(filename, "w", newline="", encoding="utf-8") as f:
            writer = csv.writer(f)
            writer.writerow(["date", "article_id", "url", "category"])
            for row in sorted_articles:
                writer.writerow(row)

        logger.info(f"Saved {len(sorted_articles)} unique articles (all categories) to {filename}")
        return len(sorted_articles), filename
```

```python
def main():
    # === set variables ===
    months = ["202201", "202202", "202203", "202204", "202205"]
    categories = [
        "local",        # 地方新聞
        "realtime",     # 即時新聞
        "entertainment", # 娛樂新聞
        "sports",       # 體育新聞
        "international", # 國際新聞
        "finance",      # 財經新聞
        "life",         # 生活新聞
        "forum"         # 論壇
    ]

    # scraper
    scraper = MultiCategoryWaybackScraper(
        max_workers=2,      # 2workers
        max_retries=3,      # retry times
        timeout=60          # timeout:60s
    )

    # start scrapping
    overall_start = time.time()
    all_category_data = {}
    category_files = []

    logger.info("=" * 80)
    logger.info(f"STARTING MULTI-CATEGORY SCRAPING")
    logger.info(f"Months: {months}")
    logger.info(f"Categories: {categories}")
    logger.info("=" * 80)

    for i, category in enumerate(categories, 1):
        logger.info(f"\n{'='*20} CATEGORY {i}/{len(categories)}: {category.upper()} {'='*20}")

        category_rows, failed = scraper.fetch_category_data(months, category)
        all_category_data[category] = category_rows

        # saved sing. category
        if category_rows:
            count, filename = scraper.save_category_to_csv(category_rows, category, months)
            category_files.append((category, count, filename))
        else:
            logger.warning(f"[{category}] No data found!")
            category_files.append((category, 0, "No file"))

        # sleep between both category
        if i < len(categories):
            rest_time = random.uniform(30, 60)
            logger.info(f"Resting {rest_time:.1f}s before next category...")
            time.sleep(rest_time)

    # saved all news
    total_count, combined_file = scraper.save_all_categories_to_csv(all_category_data, months)

    # final report
    total_time = time.time() - overall_start

    logger.info("\n" + "=" * 80)
    logger.info("FINAL REPORT")
    logger.info("=" * 80)
    logger.info(f"Total execution time: {total_time:.2f} seconds ({total_time/60:.1f} minutes)")
    logger.info(f"Months processed: {', '.join(months)}")
    logger.info(f"Categories processed: {len(categories)}")

    logger.info("\nPer-category results:")
    total_articles = 0
    for category, count, filename in category_files:
        logger.info(f"  {category:>15}: {count:>6} articles → {filename}")
        total_articles += count

    logger.info(f"\nCombined results:")
    logger.info(f"  Total raw articles: {total_articles}")
    logger.info(f"  Unique articles: {total_count}")
    logger.info(f"  Combined file: {combined_file}")

    logger.info("\nSuccess rates by category:")
    for category in categories:
        stats = scraper.stats[category]
        if stats["success"] + stats["failed"] > 0:
            success_rate = stats["success"] / (stats["success"] + stats["failed"]) * 100
            logger.info(f"  {category:>15}: {success_rate:>5.1f}% "
                       f"({stats['success']}/{stats['success'] + stats['failed']} requests, "
                       f"{stats['total_items']} items)")
        else:
            logger.info(f"  {category:>15}: No requests made")

    # create SUMMARY file
    summary_file = f"SUMMARY_appledaily_{months[0]}-{months[-1]}.txt"
    with open(summary_file, "w", encoding="utf-8") as f:
        f.write(f"Apple Daily Archive Summary\n")
        f.write(f"Period: {months[0]} - {months[-1]}\n")
        f.write(f"Execution time: {total_time:.2f} seconds\n")
        f.write(f"Total unique articles: {total_count}\n\n")
        f.write("Files created:\n")
        for category, count, filename in category_files:
            if filename != "No file":
                f.write(f"  {filename}\n")
        f.write(f"  {combined_file}\n")

    logger.info(f"\nSummary saved to: {summary_file}")
    logger.info("SCRAPING COMPLETED!")
```

```python
if __name__ == "__main__":
    main()
```
