---
layout: post
title: CRUD Operations
date: 2022-07-11 10:00:20 +0700
description: Soft description
img: sql/crud.png
tags: [SQL]
categories: [SQL]
---
Các lệnh CRUD trong sql

- *C*read

Tạo table `CardType`, `BIN`
{% highlight mysql %}
CREATE TABLE CardType (
`card_type` tinyint(8) PRIMARY KEY,
`description` varchar(64)
);
CREATE TABLE BIN (
bin int(11) PRIMARY KEY,
card_network varchar(64),
card_type tinyint(8),
INDEX card_network (`card_network`),
FOREIGN KEY (card_type) REFERENCES CardType(card_type)
);
{% endhighlight %}

Insert data
{% highlight mysql %}
INSERT INTO CardType (`card_type`, `description`) VALUES
(1, 'Debit card'), (2, 'Credit card');
INSERT INTO BIN(`bin`, `card_network`, `card_type`) VALUES
(469174, 'Visa', 2), (53035843, 'MasterCard', 1), (47738927, 'Visa', 2), (45587232, 'Visa', 1);
{% endhighlight %}