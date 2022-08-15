---
layout: post
title: Database isolation level
date: 2022-08-15 10:00:20 +0700
description: Giải thích database isolation level
img: theory/consistency.png
tags: [Theory]
categories: [Theory]
---

> Giải thích database isolation level

## Database isolation

>> Database isolation refers to the ability of a database to allows a transaction to execute as if there are no other concurrently running transactions.
   
Database isolation là khả năng cho phép một transaction được thực thi độc lập với các giao dịch đồng thời khác đang chạy

Tùy theo mức độ mà database isolation được chia làm 4 cấp độ (levels) - mức độ tăng dần như bên dưới  
+----------------------+---------------------------------------------------------------+     +--------------+----------+
|   Isolation level    | Dirty read | Lost Update | Non-repeatable Read | Phantom Read |     |     Read     |   Write  |
+----------------------+------------+-------------+---------------------+--------------+     +--------------+----------+
| **Read Uncommitted** |  Possible  |   Possible  |      Possible       |   Possible   | --> |    No Lock    |  X Lock  | 
| **Read Committed**   | Impossible |   Possible  |      Possible       |   Possible   | --> | MVCC (first) |  X Lock  | 
| **Repeatable Read**  | Impossible | Impossible  |     Impossible      |   Possible   | --> |  MVCC (last) |  X Lock  | 
| **Serializable**     | Impossible | Impossible  |     Impossible      |  Impossible  | --> |    S Lock   |  X Lock  | 
+----------------------+------------+-------------+---------------------+--------------+     +--------------+----------+

Giải thích một số khái niệm 
- **S Lock** (Shared Lock): Nếu transaction A đã lock data, thì transaction B chỉ có thể read data thôi, không được chỉnh sửa
- **X Lock** (Exclusive Lock): Nếu X Lock vào data nào đó, thì sẽ không cho phép 1 transaction khác đọc hay chỉnh sửa nó

- **Dirty read**: Là hiện tượng mà một giao dịch đọc data mà sau đó data này đã bị chỉnh sửa bởi một giao dịch khác  
| Transaction 1             | Transaction 2                       |
|---------------------------|-------------------------------------|
| UPDATE users SET age = 21 |                                     |
| WHERE id = 1;             |                                     |
| /* No commit here */      |                                     |
|---------------------------|-------------------------------------|
|                           | SELECT age FROM users WHERE id = 1; | 
|                           | /* will read 21 */                  |
|---------------------------|-------------------------------------|
| ROLLBACK;                 |                                     |
|---------------------------|-------------------------------------|
|                           |/* lock-based DIRTY READ */          |
|---------------------------|-------------------------------------|

- **Non-repeatable read**: xảy ra khi transaction A tiến hành phép read trên dữ liệu, sau đó transaction B thực hiện phép write làm dữ liệu thay đổi, lần kế tiếp A lại tiến hành phép read với chính dữ liệu. Như vậy, 2 lần đọc của A thấy dữ liệu không nhất quán (consistency) trên cùng một bản ghi.

- **Phantom read**: là rủi ro xảy ra với lệnh read có điều kiện (chẳng hạn mệnh đề where trong sql). Transaction A đọc được một số X dữ liệu thỏa mãn điều kiện 1, transaction B tiến hành phép write sinh ra một lượng Y dữ liệu thỏa mãn điều kiện , A tính toán lại với điều kiện 1 thấy bổ sung thêm một lượng Y dữ và tổng dữ liệu giữa 2 lần trở lên không đồng nhất.
     
- Read Uncommitted: 

- Serializable: mức cao nhất của isolation levels, yêu cầu cả read và write locks. Khi SELECT với điều kiện WHERE là ranged thì cũng yêu cầu range-locks để tránh phantom read


## Reference

- <https://fauna.com/blog/introduction-to-transaction-isolation-levels>
- <https://blog.bytebytego.com/p/what-are-database-isolation-levels>
- <https://en.wikipedia.org/wiki/Isolation_(database_systems)#Read_phenomena>