# 使用 CDX api 查找 時間範圍內的網址內容

## 簡介

這支程式的目的是協助使用者獲得特定年月、類別的文章列表，
透過 Wayback CDX API 與`requests`取得文章 url、Wayback 快取的網址、文章 id，能夠初步查看總共有哪些文章。

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
這支程式會使用 `requests`、`Beautiful Soup`這個套件與 Wayback Machine 做互動。
在開始之前，請透過命令列先安裝以上套件/環境
```

## Input

## 輸出結果
