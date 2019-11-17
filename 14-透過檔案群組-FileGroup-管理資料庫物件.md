檔案群組是用來儲存資料庫物件的檔案

在設計上會建議資料表的索引儲存在與資料內容不同的檔案群組
並可將儲存索引的檔案群組放置讀取速度較快的儲存位置
（例：固態硬碟）

## 經驗談

### 不需要嘗試將資料表**資料內容**與**叢集主索引鍵**的檔案群組分開

叢集主索引鍵的檔案群組就是該資料表資料內容檔案群組位置

例

```
CREATE TABLE [dbo].[FileGroupDemoTable]
(
	[Id]		INT IDENTITY(1,1),
	
	...
	
	CONSTRAINT [PK_FileGroupDemoTable]
		PRIMARY KEY CLUSTERED ([Id]) ON [INDEX]

) ON [PRIMARY]
```

使用上述指令碼建立之後
檢查檔案群組儲存空間使用量時
發現下列狀況

```
檔案群組	使用空間(MB)
PRIMARY		0 MB
INDEX		30 MB
```

資料內容都儲存到 PRIMARY KEY CLUSTERED 指定的 [INDEX] 檔案群組位置

將非叢集索引建立到不同檔案群組才可分離 I/O 

要將資料內容改儲存到 PRIMARY 檔案群組的話
重新設定 CLUSTERED INDEX 的檔案群組位置即可

資料表的資料是依據「叢集索引 (clustered-index)」儲存的檔案群組而定
PRIMARY KEY NONCLUSTERED 不會影響到資料內容的儲存位置

### TEXTIMAGE_NO

透過指定資料表的 **TEXTIMAGE_ON** 可指定大數值資料行儲存至額外的檔案群組

當查詢指定大數值資料行欄位時，才會去讀取 TEXTIMAGE_ON 指定的檔案群組