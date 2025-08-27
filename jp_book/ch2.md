# 使用 CDX api 查找 時間範圍內的網址內容

## 簡介

這支程式的目的是協助使用者獲得特定年月、類別的文章列表，
透過 Wayback CDX API 與`requests`取得文章 url、文章 id，能夠初步查看像是https://www.appledaily.com.tw/local/20220801/ 這個網站架構下的所有文章。

為了加快速度，使用了多執行緒來加快程式運作。

```{tip}
以下皆是在Python3中執行。
在開始之前，建議先開一個虛擬環境，避免衝突。
```

```
python3 -m venv .venv  # 建立虛擬環境 (資料夾名稱可自訂，一般用 .venv 或 venv)
source .venv/bin/activate # 啟動虛擬環境 macOS / Linux
.venv\Scripts\activate # Windows
```

```{note}
請先透過命令列或anacoda 安裝jupyter notebook。
這支程式會使用 `requests`這個套件與 Wayback Machine 做互動。
在開始之前，請透過命令列先安裝以上套件/環境
```

## 輸出結果

```python
activetime: 2025-08-26 20:36:48
output_index: output_20250826_203648
[1] ✔ https://www.appledaily.com.tw/local/20220101/4C4Q3YRHTZGGBN2YAHLJH4CSQI/
[2] ✔ https://www.appledaily.com.tw/local/20220101/C4AZKGYVYZDVTPWZ7RPWARUHDY/
[3] ✔ https://www.appledaily.com.tw/local/20220101/DUS6LUBV2FEHBHZ5R47BG6YCCY/
[4] ✔ https://www.appledaily.com.tw/local/20220101/C5GNIZT4DNA4DAFUVA5NHKJYDI/
[5] ✔ https://www.appledaily.com.tw/local/20220101/23YFPLCSYZCZPANVI4E2LTW5L4/
already output ninjs.json
zipped: output_20250826_203648.zip
total 0 URL skipped
```

```python
cost: 44.86 s
```

最後會輸出成這樣的檔案結構:

```
./
├─ appledaily_local_202201-202205.csv
├─ appledaily_realtime_202201-202205.csv
├─ appledaily_entertainment_202201-202205.csv
├─ appledaily_sports_202201-202205.csv
├─ appledaily_international_202201-202205.csv
├─ appledaily_finance_202201-202205.csv
├─ appledaily_life_202201-202205.csv
├─ appledaily_forum_202201-202205.csv
├─ appledaily_ALL_CATEGORIES_202201-202205.csv
└─ SUMMARY_appledaily_202201-202205.txt

```
