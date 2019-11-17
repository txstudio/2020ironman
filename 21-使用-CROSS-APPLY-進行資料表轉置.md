很多既有系統中存在著用資料行名稱來定義資料儲存內容的設計

> 非正規化的資料表設計

這時就可以使用 CROSS APPLY 以資料行名稱定義的設計
轉換成以資料列為主的設計

情境
食譜成本資料表

此情境中食譜的資料表有下列成本：份數、時間與電力
使用不同欄位來代表對應的單位與消耗量

原始資料如下

```
Id	Name			Type01	Unit01	Value01	Type02	Unit02	Value02	Type03	Unit03	Value03
1	芹菜蔬煎餅		份數	人		2.000	時間	分鐘	5.000	NULL	NULL	NULL
2	蛋包日式咖哩飯	份數	人		4.000	時間	分鐘	30.000	電力	度		2.500
3	龍蝦味噌湯		份數	人		8.000	時間	分鐘	15.000	電力	度		1.350
```

透過 CROSS APPLY 指令就可以將上述資料表內容進行行與列的轉置

```
--食譜資料表
SELECT a.[Id]
	,a.[Name]
	,b.[Type]
	,b.[Unit]
	,b.[Value]
FROM @RecipeCost a
	CROSS APPLY (
		VALUES ([Type01],[Unit01],[Value01])
			,([Type02],[Unit02],[Value02])
			,([Type03],[Unit03],[Value03])
	) b (
		[Type],[Unit],[Value]
	)
WHERE [Type] IS NOT NULL
```

轉置完成後的資料內容

```
-------------------------------------------
Id	Name			Type	Unit	Value
-------------------------------------------
1	芹菜蔬煎餅		份數	人		2.000
1	芹菜蔬煎餅		時間	分鐘	5.000
2	蛋包日式咖哩飯	份數	人		4.000
2	蛋包日式咖哩飯	時間	分鐘	30.000
2	蛋包日式咖哩飯	電力	度		2.500
3	龍蝦味噌湯		份數	人		8.000
3	龍蝦味噌湯		時間	分鐘	15.000
3	龍蝦味噌湯		電力	度		1.350
```

轉置後就可以將資料表進行調整
將 Type 與 Unit 抽離後建立關聯 ... 等

使用 UNPIVOT 也可以達成類似的效果

下列為完整的 Transact-SQL 片段
可複製到 SQL Server 上執行並確認效果

```
--食譜資料表
DECLARE @RecipeCost AS TABLE
(
	[Id]			SMALLINT,
	[Name]			NVARCHAR(50),

	[Type01]		NVARCHAR(10),
	[Unit01]		NVARCHAR(5),
	[Value01]		DECIMAL(13,3),	
	[Type02]		NVARCHAR(10),
	[Unit02]		NVARCHAR(5),
	[Value02]		DECIMAL(13,3),
	[Type03]		NVARCHAR(10),
	[Unit03]		NVARCHAR(5),
	[Value03]		DECIMAL(13,3),

	PRIMARY KEY ([Id])
)

--初始化資料內容
INSERT INTO @RecipeCost (
	[Id],[Name]
	,[Type01],[Unit01],[Value01]
	,[Type02],[Unit02],[Value02]
	,[Type03],[Unit03],[Value03]
) VALUES (
	1,N'芹菜蔬煎餅'
	,N'份數',N'人',2
	,N'時間',N'分鐘',5
	,NULL,NULL,NULL
),(
	2,N'蛋包日式咖哩飯'
	,N'份數',N'人',4
	,N'時間',N'分鐘',30
	,N'電力',N'度',2.5
),(
	3,N'龍蝦味噌湯'
	,N'份數',N'人',8
	,N'時間',N'分鐘',15
	,N'電力',N'度',1.35
)

SELECT a.[Id]
	,a.[Name]
	,b.[Type]
	,b.[Unit]
	,b.[Value]
FROM @RecipeCost a
	CROSS APPLY (
		VALUES ([Type01],[Unit01],[Value01])
			,([Type02],[Unit02],[Value02])
			,([Type03],[Unit03],[Value03])
	) b (
		[Type],[Unit],[Value]
	)
WHERE [Type] IS NOT NULL
```