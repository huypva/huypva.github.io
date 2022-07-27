---
layout: post
title: Auto-Configuration
date: 2021-08-28 10:00:20 +0700
description: Tạo auto configuration trong Spring Boot
img: spring_boot/spring_boot_icon.png
tags: [Spring Boot]
categories: [Spring Boot]
source: https://github.com/huypva/auto-configuration-example
---

> Tạo lớp configuration và tự động load từ file trong Spring Boot

- Thêm configuration trong file application.yml

```yml
my-config:
    id: 1
    value: my value
``` 

- Tạo class tương ứng với configuration

```java
public class MyConf {

  public int id;
  public String value;

}
``` 

- Sử dụng annotation `@Configuration`, `@ConfigurationProperties` và các hàm setter để *Spring Boot* biết và tự động tạo bean tương ứng 

```java
@Setter
@Configuration
@ConfigurationProperties(prefix = "my-config")
public class MyConf {
...
```

- Sử dụng bean *MyConf* thông qua annotation *Autowired* 

```java
  @Autowired
  MyConf myConf;
  ...
  System.out.println("Id: " + myConf.getId());
  System.out.println("Value: " + myConf.getValue());
```

Output:

```
Id: 1
Value: my value
```