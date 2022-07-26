---
layout: post
title: Strategy Pattern
date: 2021-11-08 10:00:20 +0700
description: Pattern tạo ra một instance duy nhất của một class
img: design_pattern/strategy_icon.png
tags: [Design Pattern, Java]
categories: [Design Pattern]
source: https://github.com/huypva/java-strategy-pattern-example
---

## Lý thuyết

- Mục đích *Strategy Pattern* là tách rời xử lý chức năng ra khỏi đối tượng, mỗi thuật toán xử lý trên một class khác nhau, và lựa chọn giải thuật khi thực thi chương trình

- Class diagram như sau:

<div align="center">
    <img src="../assets/images/design_pattern/strategy_pattern.png"/>
</div>

  - Strategy: định nghĩa chức năng xử lý
  - ConcreteStrategy: class xử lý cho từng chức năng cụ thể
  - Context: chứa đối tượng Strategy, nhận yêu cầu từ Client và thực thi chức năng

## Demo

- Code example [ở đây](https://github.com/huypva/java-strategy-pattern-example)

## Reference

- <https://refactoring.guru/design-patterns/strategy>
- <https://gpcoder.com/4796-huong-dan-java-design-pattern-strategy/>
