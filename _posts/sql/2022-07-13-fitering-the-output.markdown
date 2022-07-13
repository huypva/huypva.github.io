---
layout: post
title: Filtering The Output
date: 2022-07-12 09:00:20 +0700
description: Soft description
img: sql/aggregate_function.png
tags: [SQL]
categories: [SQL]
---

Sử dụng các phép toán cơ bản để filter dữ liệu

Tạo table BIN và insert data sau

| bin      | card_network | card_type |
|----------|--------------|-----------|
| 469174   | Visa         | 2         |
| 45587232 | Visa         | 1         |
| 47738927 | Visa         | 2         |
| 53035843 | MasterCard   | 1         |

- Toán sử so sánh: **=**, **>**, **<**, **>=**, **>=**, ...
{% highlight sql %}
mysql> SELECT * FROM BIN WHERE card_network='Visa';
+----------+--------------+-----------+
| bin      | card_network | card_type |
+----------+--------------+-----------+
|   469174 | Visa         |         2 |
| 45587232 | Visa         |         1 |
| 47738927 | Visa         |         2 |
+----------+--------------+-----------+
3 rows in set (0.02 sec)
{% endhighlight %}

- **IN()**, **NOT IN()** - Lấy theo tập dữ liệu
{% highlight sql %}
mysql> SELECT * FROM BIN WHERE bin IN (45587232, 47738927);
+----------+--------------+-----------+
| bin      | card_network | card_type |
+----------+--------------+-----------+
| 45587232 | Visa         |         1 |
| 47738927 | Visa         |         2 |
+----------+--------------+-----------+
2 rows in set (0.01 sec)
{% endhighlight %}

- **IS NULL**, **IS NOT NULL** - Chỉ lấy các giá trị NULL, NOT NULL
{% highlight sql %}
mysql> SELECT * FROM BIN WHERE card_network IS NULL;
Empty set (0.02 sec)
{% endhighlight %}

- **LIKE()** - Lấy theo pattern

  **%** lấy bất cứ ký tự nào thỏa mãn
{% highlight sql %}
mysql> SELECT * FROM BIN WHERE bin like '45%';
+----------+--------------+-----------+
| bin      | card_network | card_type |
+----------+--------------+-----------+
| 45587232 | Visa         |         1 |
+----------+--------------+-----------+
1 row in set (0.79 sec)
{% endhighlight %}

  **_** lấy đúng 1 ký dự thỏa mãn
{% highlight sql %}
mysql> SELECT * FROM BIN WHERE bin like '45_';
Empty set (0.01 sec)
{% endhighlight %}

- **AND**, **OR** - Các phép kết hợp
{% highlight sql %}
mysql> SELECT * FROM BIN WHERE card_network='Visa' AND card_type!=1;
+----------+--------------+-----------+
| bin      | card_network | card_type |
+----------+--------------+-----------+
|   469174 | Visa         |         2 |
| 47738927 | Visa         |         2 |
+----------+--------------+-----------+
2 rows in set (0.00 sec)
{% endhighlight %}