# 研究問題、方法及流程

## 研究問題概述

透過 SQL 以及 Google LOOK Studio 資料視覺化來查看實驗室先前整理的新聞資料內容，可以看到：

- 90 多萬篇新聞卻只有 60 多萬篇
- 發現 2018 年之後，新聞數量大幅下跌
- 有些種類的新聞很少，可能是早期新聞的分類 ex. daily, news
- 日期只到 2022 的八月十幾號 -> 但網頁 31 號才停止更新
- 有沒有可能有資料沒整理到？

<iframe width="800" height="600" src="https://lookerstudio.google.com/embed/reporting/892b2c90-c9a4-4488-a6f7-559739d5a64c/page/crMUF" frameborder="0" style="border:0" allowfullscreen sandbox="allow-storage-access-by-user-activation allow-scripts allow-same-origin allow-popups allow-popups-to-escape-sandbox"></iframe>

那麼，有哪些內容沒被收到呢？
以==https://www.appledaily.com.tw/life/20220101/2DFNWSUWLBBGPKTMETNIG4KIZQ/==，嘗試觀察 appledailyTW 的網址結構，可以知道網站結構可能是：

```{mermaid}
flowchart TD
  ROOT[www.appledaily.com.tw] --> HOME[/首頁/]

  ROOT --> SECTIONS[分類（Sections）]
  SECTIONS --> life[/life/]
  SECTIONS --> local[/local/]
  SECTIONS --> international[/international/]
  SECTIONS --> entertainment[/entertainment/]
  SECTIONS --> politics[/politics/]
  SECTIONS --> finance[/finance/]
  SECTIONS --> society[/society/]
  SECTIONS --> property[/property/]
  SECTIONS --> game[/game/]
  %% 依實際需要可增減

  %% 日期列表
  life --> lifeDate[/life/YYYYMMDD/]
  local --> localDate[/local/YYYYMMDD/]
  international --> intlDate[/international/YYYYMMDD/]
  entertainment --> entDate[/entertainment/YYYYMMDD/]
  politics --> polDate[/politics/YYYYMMDD/]
  finance --> finDate[/finance/YYYYMMDD/]
  society --> socDate[/society/YYYYMMDD/]
  property --> propDate[/property/YYYYMMDD/]
  game --> gameDate[/game/YYYYMMDD/]

  %% 文章頁
  lifeDate --> lifeArt[/life/YYYYMMDD/ARTICLE_ID/]
  localDate --> localArt[/local/YYYYMMDD/ARTICLE_ID/]
  intlDate --> intlArt[/international/YYYYMMDD/ARTICLE_ID/]
  entDate --> entArt[/entertainment/YYYYMMDD/ARTICLE_ID/]
  polDate --> polArt[/politics/YYYYMMDD/ARTICLE_ID/]
  finDate --> finArt[/finance/YYYYMMDD/ARTICLE_ID/]
  socDate --> socArt[/society/YYYYMMDD/ARTICLE_ID/]
  propDate --> propArt[/property/YYYYMMDD/ARTICLE_ID/]
  gameDate --> gameArt[/game/YYYYMMDD/ARTICLE_ID/]

```

可以看出 appledaily 的網頁架構是具備一定結構的，因此試著補回原資料集，是本次的專題目標。

因為能夠做的時間有限，希望能夠快速檢驗文章的目錄收集數量以及未收錄的內容。
又因 ArchiveTeam 的存檔內容大部分可經由 IA 的 Wayback Machine 存取，因此這邊使用 Wayback CDX API 與 Wayback Machine 互動，透過 BeautifulSoup 來解析、進一步擷取 Wayback Machine 上的網頁存檔內容。

是以，透過這樣的實驗，希望：

1. 建立《蘋果新聞網》文章快取的 編目流程
2. 以 2022 年 1 月至 8 月本土版（local）為案例，建立一份具體的文章索引
3. 接著比對 CDX API 快取清單 與 ArchiveBot WARC，檢視存檔完整性差異。
4. 探討差異成因（跨日抓取、robots.txt 限制、種子設定、Save Page Now 介入）。

## 流程概要

- CDX API 抽取www.appledaily.com.tw/*，指定版面、日期區間。
- 抽取欄位：timestamp、original、statuscode、mimetype、digest、length。
- 清理 URL（去除參數、斜線）。
- 輸出為 JSON，依 ninjs3.0 標準釋出。
- CDX-only（僅在 CDX 出現，多數為 Save Page Now / IA 抓取）/WARC-only（僅在 WARC 出現，可能因政策隱藏，CDX 不顯示）統計差異數量與比例。

## API 及套件說明

以下是相關說明。

### API

[Wayback CDX api](https://github.com/internetarchive/wayback/tree/master/wayback-cdx-server)
用來取得 Wayback Machine 上最新的網頁快取連結。

### 套件

網頁爬蟲與擷取 requests → BeautifulSoup 解析 HTML → pandas 整理 → CSV/JSON 存檔
大量 URL ThreadPoolExecutor → 針對失敗加 time.sleep 重試
網址處理 用 urlparse 取得 domain 當成文章 id 或輸出欄位
分日輸出 datetime.strftime 日期目錄 → os.makedirs 建資料夾 → 完成之後用 zipfile 打包

### ninjs3.0 範例

Referance
https://github.com/iptc/newsinjson/blob/main/examples/3.0/20231008-anti-doping-measures-at-the-tour-down-under-will-be-the-toughest-ever-addb-ninjs-nitf.json
https://github.com/iptc/newsinjson/blob/main/examples/3.0/businesswire-newsml-20130731006140.json
https://www.iptc.org/std/ninjs/userguide/#_multimedia_example

```
{
"uri": "URL from warc",
"standard": {
		"name": "ninjs",
		"version": "3.0",
		"schema": "https://www.iptc.org/std/ninjs/ninjs-schema_3.0.json"
	},
"firstcreated": "2024-02-07",
"versioncreated": "2024-02-07",
"contentcreated": "2024-02-07",
"type": "text",
"language": "zh-Hant-TW"
"headlines": [
		{
			"role": "main",
			"value": "This is Headline"
		}
    	],
"subjects": [
		{
			"name": "Crime, law and justice"
		}
        ],
"bodies": [
		{
			"role": "main",
            "contentType": "text/plain",
			"value": "body content "
		}
    	],
"associations": [
        {
              "name": "mainpic",
              "uri":"URL from warc photo",
              "type":"picture",
              "headlines" : [
                {
                      "value": "picture's title"
                }
              ],
              "renditions" : [
                {
                  "href" : "./ID/img/X.jpg (relative path)",
                  "contentType": "image/jpg",

                }
              ]
        },
        {
              "name": "firstpic",
              "uri":"URL from warc photo",
              "type":"picture",
              "headlines" : [
                {
                      "value": "picture's title"
                }
              ],
              "renditions" : [
                {
                  "href" : "./ID/img/X.jpg (relative path)",
                  "contentType": "image/jpg",

                }
              ]
        },
        {
              "name": "secondpic....",
              "uri":"URL from warc photo",
              "type":"picture",
              "headlines" : [
                {
                      "value": "picture's title"
                }
              ],
              "renditions" : [
                {
                  "href" : "./PID/img/X.jpg (relative path)",
                  "contentType": "image/jpg",

                }
              ]
        }

],
"by": "by whom",
"located": "place name",
"altids": [
		    {
			"role": "internal",
			"value": "TNT3L7GJIB7M3WEDXNIS3MYPGI"
		    }
    	]
}
```
