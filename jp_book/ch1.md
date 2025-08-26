# 研究問題、方法及流程

## 實作目標

- 使用 CDX API 建立《蘋果新聞網》文章快照的 編目流程。
- 以 2022 年 1 月至 8 月本土版（local）為案例，建立一份具體的文章索引。
- 比對 CDX API 快取清單 與 ArchiveBot WARC，檢視存檔完整性差異。
- 探討差異成因（跨日抓取、robots.txt 限制、種子設定、Save Page Now 介入）。

## 研究方法與製作流程

- CDX API 抽取www.appledaily.com.tw/*，指定日期區間。
- 抽取欄位：timestamp、original、statuscode、mimetype、digest、length。
- 清理 URL（去除參數、斜線）。
- 輸出為 JSON，依 ninjs3.0 標準釋出。
- CDX-only（僅在 CDX 出現，多數為 Save Page Now / IA 抓取）/WARC-only（僅在 WARC 出現，可能因政策隱藏，CDX 不顯示）統計差異數量與比例。

## 預期成果

- 一份《蘋果新聞網》CDX 編目索引（含日期、版別、文章 ID）。
- 完整性分析報告：呈現 CDX 與 WARC 的 coverage 差異。
- 一套可重複應用的 新聞網站存檔編目流程。
