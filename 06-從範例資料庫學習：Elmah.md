Elmah 是很多開發人員在 ASP.NET 進行錯誤訊息紀錄的套件

除預設儲存在 XML 檔案之外，說明文件有提供將錯誤訊息儲存到 SQL Server 的方法

以下為儲存錯誤訊息資料表物件建立語法

```
CREATE TABLE [dbo].[ELMAH_Error]
(
	[ErrorId]     UNIQUEIDENTIFIER NOT NULL,
	[Application] NVARCHAR(60)  COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
	[Host]        NVARCHAR(50)  COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
	[Type]        NVARCHAR(100) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
	[Source]      NVARCHAR(60)  COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
	[Message]     NVARCHAR(500) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
	[User]        NVARCHAR(50)  COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
	[StatusCode]  INT NOT NULL,
	[TimeUtc]     DATETIME NOT NULL,
	[Sequence]    INT IDENTITY (1, 1) NOT NULL,
	[AllXml]      NTEXT COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL 
) 
ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]

GO

ALTER TABLE [dbo].[ELMAH_Error] WITH NOCHECK ADD 
	CONSTRAINT [PK_ELMAH_Error] PRIMARY KEY NONCLUSTERED ([ErrorId]) ON [PRIMARY] 
GO

ALTER TABLE [dbo].[ELMAH_Error] ADD 
	CONSTRAINT [DF_ELMAH_Error_ErrorId] DEFAULT (NEWID()) FOR [ErrorId]
GO
```

### TEXTIMAGE_ON

在 CREATE TABLE 語法最後有 TEXTIMAGE_ON 指令
此指令可指定 AllXml 的 NTEXT 資料儲存到不同的檔案群組

未指定該欄位時
就不會去讀取 TEXTIMAGE_ON 的指定的檔案群組內容
相較之下可減少 I/O 成本

> 在建議上 NTEXT 應改使用 NVARCHAR(MAX) 屬性，但還是適用 TEXTIMAGE_ON 屬性

### ErrorId

另外在建立主索引鍵是使用 GUID 欄位進行設定
在建議上主索引鍵資料型態不應使用具有隨機屬性的 GUID 類型

但在宣告上加入了 NONCLUSTERED
此主索引鍵就變成非叢集索引的主索引鍵

可以考慮將 Sequence 欄位設定成叢集索引
因該欄位使用遞增序號具有遞增性質，故可設定成叢集索引

資料表就會變成

- 非叢集的主索引鍵
- 叢集的索引鍵

### 參考資料

建立主索引鍵 (Primary-Key)

https://docs.microsoft.com/en-us/sql/relational-databases/tables/create-primary-keys?view=sql-server-2017

Elmah 建立之 SQL Server 資料庫物件指令碼來源

https://elmah.github.io/downloads/