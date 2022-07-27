---
layout: post
title: KISS - YAGNI - DRY
date: 2021-11-09 10:00:20 +0700
description: Giải thích các nguyên lý KISS - YAGNI - DRY
img: principle/principle.png
tags: [Principle]
categories: [Principle]
---

## KISS Principle

> KISS - Keep It Simple, Stupid
  
- Hãy giữ mọi thứ đơn giản, dễ nhìn. Một thiết kế đơn giản, một hệ thống đơn giản, một service đơn giản hay từng class, method đơn giản sẽ giúp cho dự án của bạn đơn giản, dễ maintain hơn.

- Chọn giải pháp đơn giản hơn

## YAGNI Principle 

> YAGNI - You Aren’t Gonna Need It
  
- Bạn chỉ nên tập trung vào chức năng, yêu cầu hiện tại thay vì giải quyết cho một vấn đề chưa xảy ra, không chắc xảy ra trong tương lai.

- 90% of features for the future are not used

Nguồn: [extremeprogramming.org](http://www.extremeprogramming.org/rules/early.html)

## DRY Principle

> DRY - Don’t Repeat Yourself

- Nguyên tắc này có nghĩa là bạn đừng viết lại đoạn code của bạn.

- Nếu một đoạn mã được sử dụng nhiều lần, thì nên đóng gói lại thành một mothod và sử dụng nó bằng cách gọi method đó
