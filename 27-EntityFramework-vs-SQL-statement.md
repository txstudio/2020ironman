在 .NET 開發環境中
應使用 EntityFramework 還是 ADO.NET 存取資料庫一直是個爭論不休的議題

列出之前使用 EntityFramework 遇到的問題

- 建立過長的查詢字串：超過 8000 行
- 所有的文字欄位型態皆為 NVARCHAR(MAX)
- **特定**情境中，會將所有資料表內容全部讀取出來之後才會進行操作

> 例：將 Product 與 ProductInventory 全部讀取出來後再進行 join 取得需要的資料 (!?)

雖上述條件對資料庫而言會產生很巨大的成本
但不應上述條件就斷定 EntityFramework 不是一個很好的工具，應該捨棄不使用

就如同索引的概念：使用空間換取時間
並且應詳細了解該工具提供的功能與限制

因為：就算有好的工具但你不了還是會有問題

## Entity Framework Core 處理複雜查詢

可使用檢視表 (View) 來達成

> EF Core 版本使用 Scaffold-DbContext 工具依舊不會產生 View 的物件繫結

使用 ToView 方法搭配 Lambda 可以自動產生篩選語法

**範例**

```
DbContext.ViewForProductList.Where(x => x.Name == "Product").ToList()

/* 產生的 SQL 語法 */
SELECT *
FROM [dbo].[ViewForProductList]
WHERE [Name] == 'Product' --此片段是 Entity Framework Core 自動補上
```

#### 參考資料

https://docs.microsoft.com/zh-tw/ef/core/modeling/query-types

### 結論

使用 EntityFramework 還是要去確認產生的 SQL 語法還是比較好的