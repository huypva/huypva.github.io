---
layout: post
title: Consistency
date: 2021-11-01 10:00:20 +0700
description: Giải thích Strong Consistency vs Eventual Consistency
img: theory/consistency.png
tags: [Theory]
categories: [Theory]
---

> Giải thích Strong Consistency vs Eventual Consistency

## Consistency

- Consistency(tính nhất quán): dữ liệu của một hay nhiều resource nhất quán với nhau, cùng trạng thái

- Strong Consistency(tính nhất quán mạnh): một cập nhập, thay đổi cần phải được thực hiện trên toàn bộ, sau khi thực hiện thì tất cả các dữ liệu cùng trạng thái trước khi xử lý tiếp theo

<div align="center">
    <img src="assets/img/theory/strong_consistency.png"/>
</div>

- Eventual Consitency(tính nhất quán có độ trễ): tại thời điểm cập nhập, có thể có 1 hoặc vài dữ liệu đã cập nhập, một số khác thì chưa. Nhưng sau một khoảng thời gian thì tất cả vẫn có trạng thái giống nhau

<div align="center">
    <img src="assets/img/theory/eventual_consistency.png"/>
</div>

## Reference

- <https://www.cohesity.com/blogs/strict-vs-eventual-consistency/>