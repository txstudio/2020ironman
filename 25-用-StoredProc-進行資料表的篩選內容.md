查詢是一個應用程式常見功能
然而依使用者輸入參數進行不同條件篩選

對應用程式開發人員而言
或許會使用組合字串的方式對查詢語法進行加工

> 在 EntityFramework 就是如此達成

對資料庫開發人員而言
動態產生的 SQL 字串就是件麻煩事情

因為不同的查詢語法代表
雖顯示的欄位相同
但是使用的執行計畫會不相同

對於選擇性參數
可以考慮使用下列方式設計查詢語法

```
WHERE [Column] = ISNULL(@Column, [Column])
```

若使用模糊搜尋可考慮使用

```
WHERE (@UserName IS NULL OR [UserName] LIKE '%'+@UserName+'%')
```

組合起來的查詢片段可能如下

```
WHERE [Column] = ISNULL(@Column, [Column])
	AND [Column1] = ISNULL(@Column1, [Column1])
	AND (@UserName IS NULL OR [UserName] LIKE '%'+@UserName+'%')
```

在 Azure Database 中上述做法的執行計畫會是相同的

> 使用 [Column] LIKE '%ABC%' 的查詢就算 [Column] 有建立索引也不會使用索引搜尋

也可嘗試使用 CHARINDEX(@Column,[Column]) > 0
來替換 LIKE 片段

> 某些情境下會變成 Index Seed 降低查詢成本

若要提供分頁功能
就可以使用 OFFSET ... FETCH 來取代 ROW_NUMBER 

> 可不可以使用就要看資料庫的版本

https://docs.microsoft.com/en-us/sql/t-sql/queries/select-order-by-clause-transact-sql?view=sql-server-2017#using-offset-and-fetch-to-limit-the-rows-returned