---
layout: post
title: Synchronous vs Asynchronous 
date: 2021-10-18 10:00:20 +0700
description: Giải thích synchronous và asynchronous
tags: [Theory]
categories: [Theory]
---

## Synchronous vs asynchronous processing

- Synchronous processing (xử lý đồng bộ) là một chương trình chạy tuần tự, một process sau phải chờ process trước thực thi xong mới bắt đầu thực hiện. 

- Còn với asynchronous thì process sau có thể thực hiện mà không cần chờ process trước đó, sau khi process trước xử lý xong sẽ thông báo kết quả sau cho chương trình

<div align="center">
    <img src="assets/img/theory/sync_async.png"/>
</div>


## Synchronous vs asynchronous communication    

- Synchronous: các service giao tiếp với nhau thông qua REST/gRPC API. Client sẽ call api và đợi response từ Server

<div align="center">
    <img src="assets/img/theory/api.png"/>
</div>

- Asynchronous: giao tiếp với nhau thông qua message broker như Kafka, RabbitMQ,... thường dùng trong giao tiếp service nội bộ 

<div align="center">
    <img src="assets/img/theory/message.png"/>
</div>

- Với một số api có business logic khá dài
    + Client gửi request lên Server
    + Server nhận request, tạo unique id, public một message vào một Internal Message Broker và trả id về cho Client
    + Server subscribe message từ Internal Message Broker, thực hiện business logic và lưu kết quả vào Cache
    + Client định kỳ call Server lấy kết quả của request ban đầu, Server sẽ lấy message từ Cache và response cho Client
    
<div align="center">
    <img src="assets/img/theory/long_api.png"/>
</div>

## Reference

- <https://www.koyeb.com/blog/introduction-to-synchronous-and-asynchronous-processing>