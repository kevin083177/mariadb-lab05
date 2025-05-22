## 圖書館資料庫設計
> 書籍：ISBN、書名、作者 (可能多位)、出版社名稱、出版社地址、出版年份、書籍類別。
> 
> 會員：會員卡號、會員姓名、會員地址、會員電話、會員Email。
> 
> 借閱：會員卡號、會員姓名、ISBN、書名、借閱日期、應還日期、實際歸還日期。
### 1. 問題識別
* 1-1 重複資料：
    * 書籍資訊（ISBN、書名、作者、...）若直接存於借閱表中，會隨每筆借閱重複。
    * 會員資訊（會員卡號、姓名、地址、...）若直接存於借閱表中，會隨每筆借閱重複。
* 1-2 插入異常：
    * 當還沒有任何借閱記錄時，若想新增會員或書籍，需在借閱表預先插入空借閱資料。
    * 若要錄入新的書籍，出版者資訊也需一併存在借閱表中。
* 1-3 更新異常：
    * 如果出版社地址變更，需在所有相關借閱記錄、書籍紀錄中更新多筆。
    * 成員電話或Email變更需更新多筆借閱資料。
* 1-4 刪除異常：
    * 若刪除最後一筆借閱紀錄，會連帶失去該會員或書籍的資訊。
### 2. 函數相依性 (FD)
> 以「→」表示「函數相依」，並省略重複屬性
* 書籍
    * ISBN → 書名, 出版年份, 書籍類別, 出版社名稱
    * 出版社名稱 → 出版社地址

* 會員
    * 會員 ID → 會員姓名, 會員地址, 會員電話, 會員 Email
    * 會員 Email → 會員姓名
* 借閱 
    * (會員 ID, ISBN, 借閱日期) → 應還日期, 實際歸還日期

* 衍生依賴
    * 會員 ID → 會員姓名  （簡化查詢時可直接透過 `借閱` 取得會員姓名）
    * ISBN → 書名  （簡化報表時可直接透過 `借閱` 取得書名）
### 3. 正規化設計
* 第一正規化 (1NF)
    > 所有屬性皆為不可分割的原子值。
    > 將多作者改用關聯表儲存。
    * Book (書籍)   **範例資料** 
      |ISBN|Title|PublisherName|PublisherAddress|PublishYear|Category|
        |:--|:--|:--|:--|:--|:--|
        |9781234567890|資料庫設計|ABC出版社|台北市信義路100號|2020|資訊類|
    * Author (作者)
        |AuthorID|Name|
        |:--|:--|
        |A01|王小明|
    * Book_Author (多作者)
        |ISBN|AuthorID|
        |:--|:--|
        |9781234567890|A01|
    * Publisher (出版社)
        |PublisherName|PublisherAddress|
        |:--|:--|
        |ABC出版社|台北市信義路100號|
    * Member (會員)
        |MemberID|Name|Address|Phone|Email|
        |:--|:--|:--|:--|:--|
        |M001|張三|台北市大安區|0912-345678|zhang3@example.com|
    * Loan (借閱)
        |MemberID|ISBN|LoanDate|DueDate|ReturnDate|
        |:--|:--|:--|:--|:--|
        |M001|9781234567890|2025-05-01|2025-05-15|2025-05-10|
* 第二正規化 (2NF)
    > 移除部分相依：所有非鍵屬性必須完全相依於整個主鍵。

    * 範例資料
        |MemberID|ISBN|LoanDate|DueDate|MemberName|BookTitle|
        |:--|:--|:--|:--|:--|:--|
        |M001|9781234567890|2025-05-01|2025-05-15|張三|資料庫設計|

    * Loan (PK：MemberID, ISBN, LoanDate)
        |MemberID|ISBN|LoanDate|DueDate|
        |:--|:--|:--|:--|
        |M001|9781234567890|2025-05-01|2025-05-15|

    * Member (PK：MemberID)
        |MemberID|Name|
        |:--|:--|
        |M001|張三|

    * Book (PK：ISBN)
        |ISBN|Title|
        |:--|:--|
        |9781234567890|資料庫設計|

* 第三正規化 (3NF)
    > 所有非鍵屬性不得傳遞相依於主鍵

    * 範例資料
      |ISBN|Title|PublisherName|PublisherAddress|PublishYear|Category|
        |:--|:--|:--|:--|:--|:--|
        |9781234567890|資料庫設計|ABC出版社|台北市信義路100號|2020|資訊類|
        |9780987654321|演算法導論|XYZ出版社|高雄市前鎮區中山路50號|2018|計算機科學|

    * Publisher (PK：PublisherName)
        |PublisherName|PublisherAddress|
        |:--|:--|
        |ABC出版社|台北市信義路100號|
        |XYZ出版社|高雄市前鎮區中山路50號|

    * Book (PK：ISBN)
        |ISBN|Title|PublishYear|Category|PublisherName|
        |:--|:--|:--|:--|:--|
        |9781234567890|資料庫設計|2020|資訊類|ABC出版社|
        |9780987654321|演算法導論|2018|計算機科學|XYZ出版社|
    * Member、Author、Book_Author、Loan 在 2NF 後已無傳遞相依，維持不變

### 4. ER Diagram
![ERD-01](https://github.com/kevin083177/mariadb-lab05/blob/main/Lab-05_1/ERD-01.png)