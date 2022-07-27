---
layout: post
title: Spring Boot + Redis with Redisson client
date: 2021-09-07 10:00:20 +0700
description: Hướng dẫn tích hợp Redis bằng Redisson client trong Spring Boot application
img: spring_boot/spring_boot_icon.png
tags: [Spring Boot, Redis]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-redisson-example
---

> Hướng dẫn tích hợp Redis bằng Redisson client trong Spring Boot application

- Sử dụng dependency

```xml
<dependencies>
  <dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.16.1</version>
  </dependency>
</dependencies>
``` 

Lưu ý: chọn version [redisson-spring-boot-starter](https://github.com/redisson/redisson/tree/master/redisson-spring-boot-starter) tương ứng version Spring Boot

- Tạo file redisson.yml chứa cấu hình kết nối redis server trong thư mục resouces   

```yml
singleServerConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  address: "redis://127.0.0.1:6379"
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 10
  connectionPoolSize: 64
  database: 0
  dnsMonitoringInterval: 5000
threads: 0
nettyThreads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
``` 

- Thêm cấu hình trong file application.yml

```yml
spring:
  redis:
    redisson:
      file: classpath:redisson.yml
```

- Thao tác với redis thông qua bean `RedissonClient`

```java
  @Autowired
  RedissonClient redissonClient;
    
  public void setExample(String key, String value) {
    RBucket<String> bucket = redissonClient.getBucket(key);
    bucket.set(value);
  }

  public String getExample(String key) {
    RBucket<String> bucket = redissonClient.getBucket(key);
    return bucket.get();
  }
```