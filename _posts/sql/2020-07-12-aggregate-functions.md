---
layout: post
title: Aggregate Functions
date: 2020-07-12 09:00:20 +0700
description: Soft description
img: sql/aggregate-functions.png
tags: [SQL]
categories: [SQL]
---

Mô tả các thao tác tổng hợp trên tập giá trị, thường được sử dụng với **GROUP BY** để group các giá trị trong một tập dữ liệu

Tạo table BIN và insert data sau

| bin      | card_network | card_type |
|----------|--------------|-----------|
| 469174   | Visa         | 2         |
| 45587232 | Visa         | 1         |
| 47738927 | Visa         | 2         |
| 53035843 | MasterCard   | 1         |

- **AVG()** - Lấy giá trị trung bình các row trong tập dữ liệu
{% highlight sql %}
mysql> SELECT AVG(bin) FROM BIN;
+---------------+
| AVG(bin)      |
+---------------+
| 36707794.0000 |
+---------------+
1 row in set (0.01 sec)
{% endhighlight %}

- **COUNT()** - Đếm số lượng row trong tập dữ liệu
{% highlight sql %}
mysql> SELECT card_network, COUNT(*) FROM BIN;
+----------+
| COUNT(*) |
+----------+
|        4 |
+----------+
1 row in set (0.01 sec)
{% endhighlight %}

- **MIN()**, **MAX()** - Lấy giá trị nhỏ nhất, lớn nhất của tập dữ liệu

{% highlight sql %}
mysql> SELECT MIN(bin) FROM BIN;
+----------+
| MIN(bin) |
+----------+
|   469174 |
+----------+
1 row in set (0.01 sec)
{% endhighlight %}

- **SUM()** - Tổng giá trị tất cả các dòng trong tập dữ liệu

{% highlight sql %}
mysql> SELECT SUM(bin) FROM BIN;
+-----------+
| SUM(bin)  |
+-----------+
| 146831176 |
+-----------+
1 row in set (0.00 sec)
{% endhighlight %}

Sử dụng chung với **GROUP BY**
{% highlight sql %}
mysql> SELECT card_type,SUM(bin) FROM BIN GROUP BY card_type;
+-----------+----------+
| card_type | SUM(bin) |
+-----------+----------+
|         1 | 98623075 |
|         2 | 48208101 |
+-----------+----------+
2 rows in set (0.00 sec)
{% endhighlight %}