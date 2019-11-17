先前某個情境

Table-A

```
Id	A	B	C
1	A1	B1	123
2	A1	B2	551
3	A2	B3	13
```

Table-B

```
Id	A	B	D
1	A1	B1	11
2	A1	B2	76
3	A2	B3	9
```

要將 Table-A 的 C 欄位資料儲存到 Table-B 的 D 欄位
使用 CURSOR 將 Table-A 的 A|B 欄位逐筆比對更新到 Table-B 中的 D 欄位

為了保持一致性設定了 Transaction 
為了比對快速對 A|B 欄位加入了索引

問題解決了
但是卻產生了另一個問題

先來討論 CURSOR 的特性
CURSOR 僅使用「單一核心」並逐筆進行（不會鎖定整張資料表）

## 上述情境最佳解

應使用 UPDATE JOIN 來善用多核心進行處理
CURSOR 僅單一核心的特性會讓資料處理時間相較之下較久
這時再加上 Transaction 等於又將資料表鎖定起來

- 要減少資料鎖定情況可以考慮使用 CURSOR
- 使用 CURSOR 不應使用 Transaction 

### 先前使用 CURSOR 情境

將前一個月份的「存取紀錄資料表」搬移到封存的資料表
此時「存取紀錄資料表」還是處於上線狀態
新的紀錄還是可以寫到「存取紀錄資料表」中