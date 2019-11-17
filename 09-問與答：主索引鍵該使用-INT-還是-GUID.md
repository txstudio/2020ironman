回答這個問題前
先從欄位的儲存空間開始

### UNIQUEIDENTIFIER

儲存空間是 16 位元

https://docs.microsoft.com/en-us/sql/t-sql/data-types/uniqueidentifier-transact-sql?view=sql-server-2017

### BIGINT

儲存空間是 8 位元

https://docs.microsoft.com/en-us/sql/t-sql/data-types/int-bigint-smallint-and-tinyint-transact-sql?view=sql-server-2017

使用 GUID 欄位所需要的儲存空間是使用 BIGINT 的兩倍，INT 的四倍

### 再談索引鍵特性

預設 SQL Server 的主索引鍵為叢集索引
叢集索引的特性就是預先排序

使用遞增性質欄位 (INT) 做叢集主索引鍵時
每次新增資料就會於最末端進行新增

若使用隨機性質 (GUID) 的欄位時
每次新增資料就會重新掃描後再進行新增
（相較之下會影響新增資料的效能）

### 使用 GUID 的極端例子

將所有資料表的主索引鍵都建立成 UNIQUEIDENTIFIER 欄位
在 JOIN 資料表的時候比起使用數值欄位相較之下就很吃效能

將所有員工資料表與所屬縣市資料表進行 JOIN 
若縣市資料表主索引鍵使用 GUID 的話

JOIN 的成本就會是員工資料表總數 x GUID (16) 的空間

使用 TINYINT 作為縣市資料表主索引鍵的話

JOIN 的成本就會是員工資料表總數 x TINYINT (1) 的空間

在上述情境「主索引鍵」使用「數值」欄位就會比較節省成本

從 AdventureWork 範例資料庫可以發現
主索引鍵以數值欄位為主

就 SQL Server 叢集索引特性來說
主索引鍵選擇數值欄位會是比較好的作法

若真的很在意使用數字作為主索引鍵會有用完的問題
就可以考慮使用 NEWSEQUENTIALID() 產生具有遞增性質的 GUID 型態

https://docs.microsoft.com/en-us/sql/t-sql/functions/newsequentialid-transact-sql?view=sql-server-2017