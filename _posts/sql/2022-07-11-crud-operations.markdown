---
layout: post
title: CRUD Operations
date: 2022-07-11 10:00:20 +0700
description: Soft description
img: sql/crud.png
tags: [SQL]
categories: [SQL]
---

Sử dụng các lệnh CRUD trong sql

- **C**reate

Tạo một table
{% highlight sql %}
CREATE TABLE CardType (
  `card_type` tinyint(8) PRIMARY KEY,
  `description` varchar(64)
);

CREATE TABLE BIN (
  `bin` int(11) PRIMARY KEY,
  `card_network` varchar(64),
  `card_type` tinyint(8),
  INDEX card_network (`card_network`),
  FOREIGN KEY (card_type) REFERENCES CardType(card_type)
);
{% endhighlight %}

Thêm dữ liệu vào bảng
{% highlight sql %}
INSERT INTO CardType (`card_type`, `description`) VALUES (1, 'Debit card'), (2, 'Credit card');

INSERT INTO BIN(`bin`, `card_network`, `card_type`) VALUES
(469174, 'Visa', 2), (53035843, 'MasterCard', 1), (47738927, 'Visa', 2), (45587232, 'Visa', 1);
{% endhighlight %}

- **R**ead

Sử dụng command SELECT để lấy dữ liệu từ một bảng
{% highlight sql %}
SELECT * FROM BIN;
{% endhighlight %}

| bin      | card_network | card_type |
|----------|--------------|-----------|
| 469174   | Visa         | 2         |
| 45587232 | Visa         | 1         |
| 47738927 | Visa         | 2         |
| 53035843 | MasterCard   | 1         |

- **U**pdate
Cập nhập giá trị trong bảng
{% highlight sql %}
mysql> UPDATE BIN SET card_type = 1 WHERE bin=469174;
Query OK, 1 row affected (0.16 sec)
Rows matched: 1  Changed: 1  Warnings: 0
{% endhighlight %}

Kết quả sau UPDATE

| bin      | card_network | card_type |
|----------|--------------|-----------|
| 469174   | Visa         | 1         |
| 45587232 | Visa         | 1         |
| 47738927 | Visa         | 2         |
| 53035843 | MasterCard   | 1         |


- **D**elete
    Xóa dòng trong bảng
{% highlight sql %}
mysql> DELETE FROM BIN WHERE bin = 47738927;
Query OK, 1 row affected (0.02 sec)
{% endhighlight %}

Kết quả sau DELETE

| bin      | card_network | card_type |
|----------|--------------|-----------|
| 469174   | Visa         | 1         |
| 45587232 | Visa         | 1         |
| 53035843 | MasterCard   | 1         |

