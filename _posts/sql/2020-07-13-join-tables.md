---
layout: post
title: SQL - Join Tables
date: 2020-07-13 10:00:20 +0700
description: Soft description
img: sql/join.png
tags: [SQL]
categories: [SQL]
---

Join 2 table trong SQL

Cho 2 table sau
{% highlight sql %}
Table BIN                                             Table CardType
+----------+--------------+-----------+               +-----------+--------------------+
| bin      | card_network | card_type |               | type | description        |
+----------+--------------+-----------+               +-----------+--------------------+
|   469174 | Visa         |         2 |               |         1 | Debit card         |
| 45587232 | Visa         |         1 |               |         2 | Credit card        |
| 47738927 | Visa         |         2 |               |         3 | Domestic bank card |
| 53035843 | MasterCard   |         1 |               +-----------+--------------------+                            
|   970415 | VietinBank   |         4 |
+----------+--------------+-----------+
{% endhighlight %}

- **INNER JOIN** hay **JOIN** - Lấy tập dữ liệu nằm trong cả 2 table

Tạo một table
{% highlight sql %}
mysql> SELECT * FROM BIN AS a JOIN CardType AS b ON a.card_type=b.type;
+----------+--------------+-----------+------+-------------+
| bin      | card_network | card_type | type | description |
+----------+--------------+-----------+------+-------------+
| 45587232 | Visa         |         1 |    1 | Debit card  |
| 53035843 | MasterCard   |         1 |    1 | Debit card  |
|   469174 | Visa         |         2 |    2 | Credit card |
| 47738927 | Visa         |         2 |    2 | Credit card |
+----------+--------------+-----------+------+-------------+
4 rows in set (0.41 sec)
{% endhighlight %}

- **LEFT JOIN** - Lấy dữ liệu theo table bên trái, giá trị table bên phải nếu không có thì có giá trị NULL
{% highlight sql %}
mysql> SELECT * FROM BIN AS a LEFT JOIN CardType AS b ON a.card_type=b.type;
+----------+--------------+-----------+------+-------------+
| bin      | card_network | card_type | type | description |
+----------+--------------+-----------+------+-------------+
|   469174 | Visa         |         2 |    2 | Credit card |
|   970415 | VietinBank   |         4 | NULL | NULL        |
| 45587232 | Visa         |         1 |    1 | Debit card  |
| 47738927 | Visa         |         2 |    2 | Credit card |
| 53035843 | MasterCard   |         1 |    1 | Debit card  |
+----------+--------------+-----------+------+-------------+
5 rows in set (0.02 sec)
{% endhighlight %}

- **RIGHT JOIN** - Ngược lại với LEFT JOIN, lấy dữ theo table bên phải, giá trị table bên trái nếu không có thì có giá trị NULL
{% highlight sql %}
mysql> SELECT * FROM BIN AS a RIGHT JOIN CardType AS b ON a.card_type=b.type;
+----------+--------------+-----------+------+--------------------+
| bin      | card_network | card_type | type | description        |
+----------+--------------+-----------+------+--------------------+
| 45587232 | Visa         |         1 |    1 | Debit card         |
| 53035843 | MasterCard   |         1 |    1 | Debit card         |
|   469174 | Visa         |         2 |    2 | Credit card        |
| 47738927 | Visa         |         2 |    2 | Credit card        |
|     NULL | NULL         |      NULL |    3 | Domestic bank card |
+----------+--------------+-----------+------+--------------------+
5 rows in set (0.01 sec)
{% endhighlight %}

- **FULL JOIN** - Lấy tất cả giá trị 2 table, nếu không tìm thấy giá trị phù hợp ở table còn lại thì giá trị đó NULL
 
Trong MySql có thể sử dụng UNION kết hợp 2 kết quả LEFT JOIN và RIGHT JOIN 
{% highlight sql %}
mysql> (SELECT * FROM BIN LEFT JOIN CardType ON card_type=type) UNION (SELECT * FROM BIN RIGHT JOIN CardType ON card_type=type);
+----------+--------------+-----------+------+--------------------+
| bin      | card_network | card_type | type | description        |
+----------+--------------+-----------+------+--------------------+
|   469174 | Visa         |         2 |    2 | Credit card        |
|   970415 | VietinBank   |         4 | NULL | NULL               |
| 45587232 | Visa         |         1 |    1 | Debit card         |
| 47738927 | Visa         |         2 |    2 | Credit card        |
| 53035843 | MasterCard   |         1 |    1 | Debit card         |
|     NULL | NULL         |      NULL |    3 | Domestic bank card |
+----------+--------------+-----------+------+--------------------+
6 rows in set (0.05 sec)
{% endhighlight %}

- **CROSS JOIN** - Giống như JOIN, chỉ là không có điều kiện join (ON)

{% highlight sql %}
SELECT * FROM BIN,CardType;
+----------+--------------+-----------+------+--------------------+
| bin      | card_network | card_type | type | description        |
+----------+--------------+-----------+------+--------------------+
|   469174 | Visa         |         2 |    1 | Debit card         |
|   970415 | VietinBank   |         4 |    1 | Debit card         |
| 45587232 | Visa         |         1 |    1 | Debit card         |
| 47738927 | Visa         |         2 |    1 | Debit card         |
| 53035843 | MasterCard   |         1 |    1 | Debit card         |
|   469174 | Visa         |         2 |    2 | Credit card        |
|   970415 | VietinBank   |         4 |    2 | Credit card        |
| 45587232 | Visa         |         1 |    2 | Credit card        |
| 47738927 | Visa         |         2 |    2 | Credit card        |
| 53035843 | MasterCard   |         1 |    2 | Credit card        |
|   469174 | Visa         |         2 |    3 | Domestic bank card |
|   970415 | VietinBank   |         4 |    3 | Domestic bank card |
| 45587232 | Visa         |         1 |    3 | Domestic bank card |
| 47738927 | Visa         |         2 |    3 | Domestic bank card |
| 53035843 | MasterCard   |         1 |    3 | Domestic bank card |
+----------+--------------+-----------+------+--------------------+
15 rows in set (0.02 sec)
{% endhighlight %}