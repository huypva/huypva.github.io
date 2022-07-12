---
layout: post
title: Aggregate Fundtions
date: 2022-07-12 09:00:20 +0700
description: Soft description
img: sql/crud.png
tags: [SQL]
categories: [SQL]
---

Mô tả các thao tác tổng hợp trên tập giá trị, thường được sử dụng với **GROUP BY** để group các giá trị trong một tập dữ liệu

Tạo table BIN và insert data sau:
| bin      | card_network | card_type |
|----------|--------------|-----------|
| 469174   | Visa         | 2         |
| 45587232 | Visa         | 1         |
| 47738927 | Visa         | 2         |
| 53035843 | MasterCard   | 1         |

- **AVG(expr)** - Lấy giá trị trung bình các row trong tập dữ liệu
{% highlight sql %}
mysql> SELECT AVG(bin) FROM BIN;
+---------------+
| AVG(bin)      |
+---------------+
| 36707794.0000 |
+---------------+
1 row in set (0.01 sec)
{% endhighlight %}

- **COUNT**(expr) - Đếm số lượng row trong tập dữ liệu
{% highlight sql %}
mysql> SELECT COUNT(*) FROM BIN;
+----------+
| COUNT(*) |
+----------+
|        4 |
+----------+
1 row in set (0.01 sec)
{% endhighlight %}

- **MIN**(expr), **MAX**(expr) - Lấy giá trị nhỏ nhất, lớn nhất của tập dữ liệu

{% highlight sql %}
mysql> SELECT MIN(bin) FROM BIN;
+----------+
| MIN(bin) |
+----------+
|   469174 |
+----------+
1 row in set (0.01 sec)
{% endhighlight %}

- **SUM**(expr) - Tổng giá trị tất cả các dòng trong tập dữ liệu

{% highlight sql %}
mysql> SELECT SUM(bin) FROM BIN;
+-----------+
| SUM(bin)  |
+-----------+
| 146831176 |
+-----------+
1 row in set (0.00 sec)
{% endhighlight %}