# 結語

經過測試後發現，由 Archive Team 整理的 WARC file 文章數量相較於 Internet Archive Wayback Machine CDX api 爬蟲相對少，可能是因為兩者爬蟲時間點的差異，或許這也說明了網站的數位保存是一個日常就要做的功課。

[^1]: Sherratt, Tim. GLAM-Workbench/Web-Archives. Version v1.2.0. 2023. https://doi.org/10.5281/zenodo.7898218.

不過，這種方法比較適合具備結構簡單的網頁，如果結構較複雜，這種方法就不太可行了。
另外，也因為請求量比較大，測試過程中會比較常出現`HTTPSConnectionPool(host='web.archive.org', port=443): Read timed out. (read timeout=20)`的情形，可以拉長 timeout，或是加上重試機制。

透過 Wayback Machine CDX api 結合爬蟲，使用者能更快、更容易取得不同年份的文章索引查詢；此外，使用這個 api 作人文研究也有一定的社群資源學習，適合程式入門的人文研究者。[^1]
