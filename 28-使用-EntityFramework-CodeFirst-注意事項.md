EntityFramework 在 .NET Core 環境中只有 Code First 與 Database First
在此描述先前使用 Code First 應注意項目與遇到有趣的事情

## 注意字串型態文字長度
	
建立 Entity 類別時若沒有字串型態沒有指定長度時
Entity Framework 會依預設字串欄位長度自動設定

預設字串長度的資料型態為 **NVARCHAR(MAX)**
若使用在關聯欄位的話會使用固定長度的 NVARCHAR(N)

NVARCHAR(MAX) 屬於「大數值資料行」
該欄位就無法用來建立索引
且相較於固定長度的 NVARCHAR 會使用掉更多儲存空間

所以使用 Code First 設定屬性型態為字串時記得給予適當的長度

> 使用「Data Annotations」或「Fluent API」

## 自動建立資料庫

> 復活的 Azure SQL Database

這是一個很有趣的經驗
情境為將既有資料庫 (Azure Database) 搬移至另外一台資料庫伺服器 (Azure SQL Server)
已經將資料庫搬移完成並移除原本資料庫伺服器上的資料庫

但之後發現原本被刪除的資料庫又出現在該資料庫伺服器上
原來是 ASP.NET Core 專案的連線字串忘記修改

造成每次執行時 EntityFramework 會當作第一次建置環境
自動建立一個新的資料庫出來