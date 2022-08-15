---
layout: post
title: Database Isolation Level
date: 2022-08-15 10:00:20 +0700
description: Giải thích database isolation level
img: theory/isolation_level.png
img-hidden: yes
tags: [Theory]
categories: [Theory]
---

> Giải thích database isolation level

## Database isolation

> Database isolation refers to the ability of a database to allows a transaction to execute as if there are no other concurrently running transactions.
   
Database isolation là khả năng cho phép một transaction được thực thi độc lập với các giao dịch đồng thời khác đang chạy

Tùy theo mức độ mà database isolation được chia làm 4 cấp độ (levels) - mức độ giảm dần như bên dưới  

<div align="center">
    <img src="/assets/img/theory/isolation_level.png"/>
</div>

Giải thích một số khái niệm 
- **S Lock** (Shared Lock): Nếu transaction A đã lock data, thì transaction B chỉ có thể read data thôi, không được chỉnh sửa
- **X Lock** (Exclusive Lock): Nếu X Lock vào data nào đó, thì sẽ không cho phép 1 transaction khác đọc hay chỉnh sửa nó
- **MVCC(MVCC Multi-Version Consistency Control) first**: read data at the beginning of the transaction 
- **MVCC last**: read lasted committed data

## Read phenomena
> Three different read phenomena (hiện tượng) when Transaction 1 reads data that Transaction 2 might have changed

- **Dirty read**
> A dirty read occurs when a transaction is allowed to read data from a row that has been modified by another running transaction and not yet committed

Là hiện tượng mà một giao dịch đọc data mà sau đó data này đã bị chỉnh sửa bởi một giao dịch khác  
{% highlight sql %}
                                | Transaction 1             | Transaction 2              |
                                |---------------------------|----------------------------|
Transaction 1 changes a row,    | UPDATE users SET age = 21 |                            |
but does not commit the changes | WHERE id = 1;             |                            |
                                | /* No commit here */      |                            |
                                |---------------------------|--------------------------- |
Transaction 2 then reads        |                           | SELECT age FROM users      |
the uncommitted data            |                           | WHERE id = 1;              |  
                                |                           | /* will read 21 */         |
                                |---------------------------|----------------------------|
Transaction 1 rolls back or     | ROLLBACK;                 |                            | 
updates different changes to    |                           |                            |
the database                    |                           |                            |
                                |---------------------------|----------------------------|
Data got from Transaction 2     |                           |/* lock-based DIRTY READ */ |
is dirty                        |                           |                            |
                                |---------------------------|----------------------------|                                 
{% endhighlight %}

- **Non-repeatable read**
> During the course of a transaction, a row is retrieved twice and the values within the row differ between reads  

Trong quá trính thực hiện transaction, 1 row được đọc 2 lần và ra 2 kết quả khác nhau
{% highlight sql %}
                           | Transaction 1          | Transaction 2                     |
                           |------------------------|-----------------------------------|
Transaction 1 reads data   | SELECT * FROM users    |                                   |
from a row                 |  WHERE id = 1;         |                                   |
                           |------------------------|-----------------------------------|
Transaction 2 commits      |                        | UPDATE users SET age = 21         |
successfully a changed dat |                        | WHERE id = 1;                     |
                           |                        | COMMIT;                           |
                           |                        | /* in multiversion concurrency */ |
                           |                        | /* control, or lock-based READ */ |
                           |                        | /* COMMITTED */                   |
                           |------------------------|-----------------------------------|
Transaction 1 reads data   | SELECT * FROM users    |                                   |
again and got a different  | WHERE id = 1;          |                                   |
value in that row          | COMMIT;                |                                   |
                           | /* lock-based */       |                                   |
                           | /* REPEATABLE READ */  |                                   |
                           |------------------------|-----------------------------------|
{% endhighlight %}

- **Phantom read**: new rows are added or removed by another transaction to the records being read
{% highlight sql %}
                           | Transaction 1          | Transaction 2                    |
                           |------------------------|----------------------------------|
Transaction 1 executed     | SELECT * FROM users    |                                  |
the same query twice       |  WHERE age BETWEEN 10  |                                  |   
                           |  AND 30;               |                                  |
                           |------------------------|----------------------------------|
                           |                        | INSERT INTO users(id, name, age) |
                           |                        | VALUES (3, 'Bob', 27);           |
                           |                        | COMMIT;                          |
                           |------------------------|----------------------------------|
However, a different set   | SELECT * FROM users    |                                  |
of rows may be returned    |  WHERE age BETWEEN 10  |                                  |   
the second time            |  AND 30;               |                                  |
                           |------------------------|----------------------------------|
{% endhighlight %}
     
## Isolation level
- Serializable: mức cao nhất của isolation levels, yêu cầu cả read và write locks. Khi SELECT với điều kiện WHERE là ranged thì cũng yêu cầu range-locks để tránh phantom read
- Repeatable Reads: có read và write locks nhưng không cần đến range locks, nên phantom reads có thể xảy ra
- Read committed: chỉ bao gồm write locks, nên read committed chỉ đảm bảo dirty reads là không xảy ra
- Read Uncommitted: mức thấp nhất, cả 3 dirty reads, non-repeatable reads, phantom reads đều có thể xảy ra
- Ngoài ra, còn một level nữa là **Snapshot**: tương đương với Serializable, nhưng khác về cách thức hoạt động. 
    - Khi transaction đang select các bản ghi, nó không khóa các bản ghi này lại mà tạo một bản sao (snapshot) và select trên đó,
     mục đích để giảm blocking giữa các transction, nhưng sẽ tăng bộ nhớ để lưu trữ snapshot

## Reference

- <https://fauna.com/blog/introduction-to-transaction-isolation-levels>
- <https://blog.bytebytego.com/p/what-are-database-isolation-levels>
- <https://en.wikipedia.org/wiki/Isolation_(database_systems)#Read_phenomena>