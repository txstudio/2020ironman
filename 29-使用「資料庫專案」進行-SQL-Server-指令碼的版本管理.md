在 Visual Studio 中可使用「資料庫專案」資料庫物件進行管理
若在專案類型中沒有此項目可重新進入安裝畫面勾選安裝「資料庫專案」

> 目前僅支援 SQL Server Transact-SQL 指令碼

下面舉例幾個使用「資料庫專案」的好處

## 支援版本管理

將資料庫專案放到版本管理伺服器，就可進行版本管理

若是先前已經存在的資料庫

可將既有資料庫的物件指令碼匯入資料庫專案後就可繼續支援版本管理

## 透過編譯可找出異動後會出現錯誤的資料庫物件

變更主索引鍵資料型態但未更新外來鍵的型態時
重新編譯專案會出現錯誤訊息

![https://ithelp.ithome.com.tw/upload/images/20191013/20119758iiarhV0oV9.png](https://ithelp.ithome.com.tw/upload/images/20191013/20119758iiarhV0oV9.png)

若變更資料表主索引鍵資料型態時
在部屬時會自動產生將關聯卸載並重新加入的指令碼

> 不影響的既有的資料

## 如何部屬至 SQL Server

可透過「產生指令碼」檢視資料庫專案產生的 Transact-SQL 語法

![https://ithelp.ithome.com.tw/upload/images/20191013/20119758KmxsVM8iWU.png](https://ithelp.ithome.com.tw/upload/images/20191013/20119758KmxsVM8iWU.png)

若有修改主索引鍵欄位型態時

會自動產生物件異動的指令碼

僅需要執行指令碼就可以完成變更

**請注意：若有變更欄位名稱的話要注意產生的異動指令碼**

### 由資料庫專案產生的 Transact-SQL 部屬指令碼

可以發現 **CONSTRAINT** 會自動進行卸載與重建，並會將原本資料表透過 **sp_rename** 方式重新匯入

```
PRINT N'正在卸除 [Application].[User] 上的未命名條件約束...';


GO
ALTER TABLE [Application].[User] DROP CONSTRAINT [DF__User__whenCreate__3A81B327];


GO
PRINT N'正在卸除 [Application].[FK_UserInRole_RoleId]...';


GO
ALTER TABLE [Application].[UserInRole] DROP CONSTRAINT [FK_UserInRole_RoleId];


GO
PRINT N'正在卸除 [Application].[FK_UserInRole_UserId]...';


GO
ALTER TABLE [Application].[UserInRole] DROP CONSTRAINT [FK_UserInRole_UserId];


GO
PRINT N'開始重建資料表 [Application].[Role]...';


GO
BEGIN TRANSACTION;

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SET XACT_ABORT ON;

CREATE TABLE [Application].[tmp_ms_xx_Role] (
    [Id]   SMALLINT     IDENTITY (1, 1) NOT NULL,
    [Name] VARCHAR (25) NULL,
    CONSTRAINT [tmp_ms_xx_constraint_PK_Role1] PRIMARY KEY CLUSTERED ([Id] ASC)
);

IF EXISTS (SELECT TOP 1 1 
           FROM   [Application].[Role])
    BEGIN
        SET IDENTITY_INSERT [Application].[tmp_ms_xx_Role] ON;
        INSERT INTO [Application].[tmp_ms_xx_Role] ([Id], [Name])
        SELECT   [Id],
                 [Name]
        FROM     [Application].[Role]
        ORDER BY [Id] ASC;
        SET IDENTITY_INSERT [Application].[tmp_ms_xx_Role] OFF;
    END

DROP TABLE [Application].[Role];

EXECUTE sp_rename N'[Application].[tmp_ms_xx_Role]', N'Role';

EXECUTE sp_rename N'[Application].[tmp_ms_xx_constraint_PK_Role1]', N'PK_Role', N'OBJECT';

COMMIT TRANSACTION;

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;


GO
PRINT N'開始重建資料表 [Application].[User]...';


GO
BEGIN TRANSACTION;

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SET XACT_ABORT ON;

CREATE TABLE [Application].[tmp_ms_xx_User] (
    [Id]         SMALLINT       IDENTITY (1, 1) NOT NULL,
    [Account]    VARCHAR (50)   NULL,
    [Email]      NVARCHAR (150) NULL,
    [whenCreate] SMALLDATETIME  DEFAULT (GETDATE()) NULL,
    CONSTRAINT [tmp_ms_xx_constraint_PK_User1] PRIMARY KEY CLUSTERED ([Id] ASC)
);

IF EXISTS (SELECT TOP 1 1 
           FROM   [Application].[User])
    BEGIN
        SET IDENTITY_INSERT [Application].[tmp_ms_xx_User] ON;
        INSERT INTO [Application].[tmp_ms_xx_User] ([Id], [Account], [Email], [whenCreate])
        SELECT   [Id],
                 [Account],
                 [Email],
                 [whenCreate]
        FROM     [Application].[User]
        ORDER BY [Id] ASC;
        SET IDENTITY_INSERT [Application].[tmp_ms_xx_User] OFF;
    END

DROP TABLE [Application].[User];

EXECUTE sp_rename N'[Application].[tmp_ms_xx_User]', N'User';

EXECUTE sp_rename N'[Application].[tmp_ms_xx_constraint_PK_User1]', N'PK_User', N'OBJECT';

COMMIT TRANSACTION;

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;


GO
PRINT N'開始重建資料表 [Application].[UserInRole]...';


GO
BEGIN TRANSACTION;

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SET XACT_ABORT ON;

CREATE TABLE [Application].[tmp_ms_xx_UserInRole] (
    [UserId] SMALLINT NOT NULL,
    [RoleId] SMALLINT NOT NULL,
    CONSTRAINT [tmp_ms_xx_constraint_PK_UserInRole1] PRIMARY KEY CLUSTERED ([RoleId] ASC, [UserId] ASC)
);

IF EXISTS (SELECT TOP 1 1 
           FROM   [Application].[UserInRole])
    BEGIN
        INSERT INTO [Application].[tmp_ms_xx_UserInRole] ([RoleId], [UserId])
        SELECT   [RoleId],
                 [UserId]
        FROM     [Application].[UserInRole]
        ORDER BY [RoleId] ASC, [UserId] ASC;
    END

DROP TABLE [Application].[UserInRole];

EXECUTE sp_rename N'[Application].[tmp_ms_xx_UserInRole]', N'UserInRole';

EXECUTE sp_rename N'[Application].[tmp_ms_xx_constraint_PK_UserInRole1]', N'PK_UserInRole', N'OBJECT';

COMMIT TRANSACTION;

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;


GO
PRINT N'正在建立 [Application].[FK_UserInRole_RoleId]...';


GO
ALTER TABLE [Application].[UserInRole] WITH NOCHECK
    ADD CONSTRAINT [FK_UserInRole_RoleId] FOREIGN KEY ([RoleId]) REFERENCES [Application].[Role] ([Id]);


GO
PRINT N'正在建立 [Application].[FK_UserInRole_UserId]...';


GO
ALTER TABLE [Application].[UserInRole] WITH NOCHECK
    ADD CONSTRAINT [FK_UserInRole_UserId] FOREIGN KEY ([UserId]) REFERENCES [Application].[User] ([Id]);


GO
PRINT N'正在針對新建立的條件約束檢查現有資料';


GO
USE [$(DatabaseName)];


GO
ALTER TABLE [Application].[UserInRole] WITH CHECK CHECK CONSTRAINT [FK_UserInRole_RoleId];

ALTER TABLE [Application].[UserInRole] WITH CHECK CHECK CONSTRAINT [FK_UserInRole_UserId];


GO
PRINT N'更新完成。';


GO
```
