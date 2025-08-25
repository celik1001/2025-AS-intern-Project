# 歡迎！

> 這是張維芹在 2025 年 中央研究院資訊科學研究所 Depositar 研究資料寄存所 的專題作品。

## 專題題目

「Web Archive 編目的研究與實作：以 Wayback CDX API 建立《蘋果新聞網》索引」
Research and Implementation of Web Archive Indexing: Building an Apple Daily Catalogue with the CDX API

## 摘要

本專題旨在探討如何運用 **Internet Archive 所提供的 CDX API**，為《蘋果新聞網》（Apple Daily Taiwan）建立一套文章存檔的編目流程。隨著《蘋果日報》於 2021 年停刊，其網站內容逐漸消失，僅能透過 Internet Archive 及 Archive Team 等社群力量加以保存。雖然使用者仍能透過 **Wayback Machine** 存取單頁快取，但對於研究者而言，新聞檢索與系統化分析並不容易。先前已有暑期實習生利用 Archive Team 保存的 WARC 檔嘗試建立索引，但經過兩輪整理後，仍有超過三十萬篇新聞未能收錄，顯示現有方法存在明顯缺口。

本研究將以 **CDX API** 為主要工具，蒐集《蘋果新聞網》的快取清單，並依日期與版別（如生活、國際、娛樂等）加以分類，初步聚焦於 **2022 年 1 月至 8 月的本土新聞版**，建立一份可檢索的文章索引。在此基礎上，進一步比對 **CDX API 所揭示的快取** 與 **ArchiveBot WARC 檔**，檢視兩者在存檔完整性上的差異，並探討造成落差的可能因素，例如跨日抓取、robots.txt 政策限制、種子範圍設定與使用者透過 Save Page Now 所提交的快取等。

透過本專題，期望能提出一套可重複應用的新聞存檔編目流程，不僅改善《蘋果新聞網》存檔的檢索可用性，也為數位新聞保存與學術研究提供可行的技術參考。

This project aims to explore the use of the **CDX API provided by the Internet Archive** to develop a systematic indexing workflow for _Apple Daily Taiwan_. After the newspaper ceased publication in 2021, its website content gradually disappeared, with preservation efforts largely relying on the Internet Archive and community initiatives such as Archive Team. Although users can still access snapshots via the **Wayback Machine**, systematic retrieval and analysis remain difficult for researchers. Previous attempts by summer interns to build an index from Archive Team’s WARC files revealed significant gaps, leaving more than 300,000 articles uncaptured across two rounds of processing.

In this study, we employ the **CDX API** to collect snapshot lists of Apple Daily, classifying them by date and section (e.g., Life, International, Entertainment), with a specific focus on **the Local News section between January and August 2022**. We then compare the coverage between **snapshots retrievable via the CDX API** and those stored in **ArchiveBot WARC files**, in order to identify discrepancies in archival completeness and to investigate possible causes such as cross-day captures, robots.txt restrictions, seed scope limitations, and Save Page Now user submissions.

By doing so, this project not only produces a searchable index for the Apple Daily archives but also develops a replicable workflow for indexing news archives more broadly, contributing to the practice of digital preservation and supporting scholarly research in the digital humanities.

```{tableofcontents}

```
