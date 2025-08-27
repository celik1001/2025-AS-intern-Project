# WARC 與 Wayback Machibe CDX api 的差異分析

```{Note}
透過兩支程式碼爬取下來的連結，並使用 python 進行數量比較
```

````{dropdown} example metadata
```python
import pandas as pd

# 讀取 CSV
df_a = pd.read_csv('202201_local.csv') #WARC
df_b = pd.read_csv('202201local_cdx.csv') #CDX

# 假設欄位名字叫 'id' 和 'url'
ids_a = set(df_a['id'])
ids_b = set(df_b['id'])

only_in_a = ids_a - ids_b
only_in_b = ids_b - ids_a
in_both   = ids_a & ids_b

# 輸出 only_in_a（id + url）
df_only_a = df_a[df_a['id'].isin(only_in_a)][['id', 'uri']]
df_only_a.to_csv('only_in_a.csv', index=False)

# 輸出 only_in_b（id + url）
df_only_b = df_b[df_b['id'].isin(only_in_b)][['id', 'uri']]
df_only_b.to_csv('only_in_b.csv', index=False)

# 輸出 in_both（id + url，這邊用 a.csv 的 url）
df_both = df_a[df_a['id'].isin(in_both)][['id', 'uri']]
df_both.to_csv('in_both.csv', index=False)
print ('完成')

print("只在 A 有的數量:", len(only_in_a))
print("只在 B 有的數量:", len(only_in_b))
print("兩邊都有的數量:", len(in_both))
print("A 總數:", len(ids_a))
print("B 總數:", len(ids_b))
```
````

## 印出結果

The history saving thread hit an unexpected error (OperationalError('attempt to write a readonly database')).History will not be written to the database.
完成
只在 A 有的數量: 1 # 這是列
只在 B 有的數量: 1642
兩邊都有的數量: 261
A 總數: 262
B 總數: 1903

## 分析

經過測試後發現，WARC 與 Wayback Machine CDX api 在 2022 年 1 月本土版`local/202201{DD}`所保存的頁面共相差 1642 則，推測可能的原因是：

1. 收錄時間點的差異
   Archive Team 存檔是在蘋果日報關站時一次抓；但 Internet Archive（IA）長期有零散的快取。在某些日子、特定分類，兩邊的覆蓋率不會完全重疊，所以最後數量可能不一樣。

2. 實際差距的意義
   但是以 90 萬筆來說，差了 1,642 筆算是非常小的數字（~0.1–0.2%），也就是說，一部分 WARC 有，但 IA 沒有；另一部分 IA 有但 WARC 沒有。
   也許針對一整年進行爬取，會更了解實際情形。
