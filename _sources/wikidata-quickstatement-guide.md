---
title: Ｗikidata麻瓜也可以上手ㄉ批次編輯教學
tags: [wikidata, 教材]

---

# Ｗikidata麻瓜也可以上手ㄉ批次編輯教學
這份筆記依照[維基數據編輯手冊](https://www.wikidata.org/wiki/Help:QuickStatements)，以及[Hsu的文章](https://hackmd.io/X9vwu6UpS8q8teWPgUb_Eg?view)編寫而成。
> 版本紀錄V3


## 批次編輯概要
透過[wikidata Quickstatements](https://quickstatements.toolforge.org/#/batch)進行批次編輯共有兩種新增方式：
1. 逗點分隔值檔案(CSV.file)
2. V1 TAB鍵分隔值
```
我一開始用第一種，但不知為何總有問題...
所以這邊建議用第二種方法來進行，新增較容易，下面也以這方法說明
```
> 檔案範例 [會訊示範檔](https://docs.google.com/spreadsheets/d/1iFHj1s8SPAx55lm3wCJwbsMicI-TJ7iM/edit?usp=sharing&ouid=104625020094623012034&rtpof=true&sd=true)

```
❗️❗️這份檔案只提供輸入資料參考，請千萬不要拿去再建置一遍，會出事
```
進行批次編輯時，如果對常用的property還不是很熟，一般建議在試算表中建置一個詞語對照表以供參考

### 創建一個Item的意思
```
CREATE //創建//
下方每欄的意思分別為--
待創建的唯一識別碼（qid）    Property    value
LAST	Lzh-tw	"從革新的情操開始：林懷民老師談新港藝術高中"
LAST	Dzh-tw	"收錄於新港文教基金會《會訊》第158期的文章"
LAST	P31	Q191067
LAST	P407	Q262828
LAST	P1433	Q120987867
LAST	P478	"158" 
LAST	P495	Q865
LAST	P2093	"范巽綠"
LAST	P577	+2006-02-01T00:00:00Z/11
```
文字和數字（如Lzh-tw, Dzh-tw, P478, P2093）收錄的都是字串(String)，所以要加上雙引號
詳情請參考-[wikidata的資料類型](https://hackmd.io/X9vwu6UpS8q8teWPgUb_Eg?view#Wikidata-%E8%B3%87%E6%96%99%E9%A1%9E%E5%9E%8B)

### 使用小工具
如果你是新手，請勿直接在Wikidata進行批次編輯！！！
[Quickstatement](https://quickstatements.toolforge.org/#/batch)
1. 請先建立任一專案的維基帳號，進入[Quickstatement](https://quickstatements.toolforge.org/#/batch)頁面。
:::danger
2. 進行批次編輯前，在''Create new command batch for...''請先選擇==Wikidata-test==欄位。
![截圖 2024-02-05 下午12.35.17](https://hackmd.io/_uploads/SJbLU1056.png)
這個欄位是Wikidata的沙盒，提供測試使用。
:::

4. 在試算表完成資料填入後，先**複製**表中內容，並貼在空白框框內
5. 因為我們是以V1版本填入資料，所以進到工具內請選擇***import V1 Commands*** 輸入資料
6. 預覽完畢後，按下Run，沒有紅色error跑出來，就是成功啦～
> 詳細過程對照[Hsu的文章](https://hackmd.io/X9vwu6UpS8q8teWPgUb_Eg?view)即可
