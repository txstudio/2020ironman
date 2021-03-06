建議上 SQL Server 最好安裝於獨立環境
不要與應用程式共用同機器以免互相影響效能

例：

將 ASP.NET Web Application 與 SQL Server 放在同一台機器中
就可能造成 IIS 與 SQL Server 互相搶資源的情況產生	
	
就安裝獨立的 SQL Server 而言
有很多項目是可以進行優化

- 從資料庫檔案位置的規劃
- 硬碟格式化的配置單位大小
- 暫存資料庫優化
- 備份檔案的存放位置
- ... 等 ...	

下面列出幾個安裝獨立 SQL Server 時您可以考慮的設定項目

### 建立額外的 Windows 存取帳號

於 Windows 作業系統安裝 SQL Server 資料庫時
會需要輸入服務執行的 Windows 帳號

建議上使用的 Windows 帳號權限越小越好
因額外建立提供給服務使用的 Windows 帳號

SQL Server 是可以執行 Windows 指令的（需設定）
若使用系統管理員帳號作為執行 SQL Server 服務帳號時
代表可以透過 SQL Server 對作業系統做任意操作

### 設定 SQL Server 可使用的記憶體上限

一個很忙碌的資料庫是會很吃作業系統的記憶體
當記憶體使用量過高時往往會影響到底層作業系統的操作

在資料庫的屬性中可以設定 SQL Server 最多可以使用多少記憶體

![https://ithelp.ithome.com.tw/upload/images/20190918/20119758JyHpXfDNXx.png](https://ithelp.ithome.com.tw/upload/images/20190918/20119758JyHpXfDNXx.png)

假設全部記憶體為 4GB
就可以設定 SQL Server 最多使用 3GB
讓作業系統或是其他應用程式進行使用