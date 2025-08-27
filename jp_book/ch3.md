# 使用 BeautifulSoup、requests 來擷取 Wayback Machine 上的網頁內容並存檔

## 簡介

這支程式的目的是協助使用者擷取特定年月、類別的文章列表，
透過 Wayback CDX API 與`requests`、`BeautifulSoup`取得文章 url、Wayback 快取的網址、文章 id，能夠初步查看總共有哪些文章。

為了加快速度，使用了多執行緒來加快程式運作。

```{tip}
以下程式碼皆是在Python3環境中執行。
在開始之前，建議先開一個虛擬環境，避免衝突。

```

```
python3 -m venv .venv # 建立虛擬環境 (資料夾名稱可以自訂，一般用 .venv 或 venv)
source .venv/bin/activate # 啟動虛擬環境 macOS / Linux
.venv\Scripts\activate # Windows
```

```{note}
請先透過命令列或anacoda 安裝jupyter notebook。
另外，這支程式會使用`requests`、`Beautiful Soup`這個套件與 Wayback Machine 做互動，在開始之前，也請先安裝。
```

最後會輸出成這樣的檔案結構

```
├─ images/                          # 存放所有文章的圖片
│  ├─ 4C4Q3Y...CSQI/
│  │  └─ img/                       # 這篇文章的圖片子資料夾
│  ├─ 23YFPL...TW5L4/
│  │  └─ img/
│  ├─ C4AZKG...RUHDY/
│  │  └─ img/
│  ├─ C5GNIZ...HJYDI/
│  │  └─ img/
│  └─ DUS6LU...6YCCY/
│     └─ img/
└─ ninjs.json                       # 匯出的新聞資料（JSON 陣列）

```
