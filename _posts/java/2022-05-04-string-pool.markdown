---
layout: post
title: String Pool
date: 2022-05-04 13:32:20 +0300
description: Soft description. # Add post description (optional)
img: java/string_pool.jpg # Add image post (optional)
fig-caption: Figcaption # Add figcaption (optional)
tags: [String, Java]
categories: [Java]
---
Giải thích hoạt động của kiểu dữ liệu `String` trong Java.


- Tạo `String` bằng `String str1 = "abc"`
    - JMV sẽ kiểm tra và tạo mới một object nếu chưa có trong String pool.

```java
String str1 = "abc";
String str2 = "abc";
System.out.println("Compare str1==str2: " + (str1==str2)); 

``` 

```shell
$ mvn clean package
$ java -jar string-pool/target/string-pool-0.0.1-SNAPSHOT.jar
Compare str1==str2: true
```

> return true, str1 và str2 tham khảo đến 1 object trong String constant pool.

- Tạo `String` bằng `String str3 = new String("abc")`
    - Tạo một object trong String pool
    - Tạo thêm một object trong vùng nhớ Heap, và tham khảo đến object trong String pool

```java
String str3 = new String("abc");
String str4 = new String("abc");
System.out.println("Compare str3==str4: " + (str3==str4));
``` 

```shell
$ mvn clean package
$ java -jar string-pool/target/string-pool-0.0.1-SNAPSHOT.jar
Compare str3==str4: false
```

> return false, bởi vì str3 tham khảo đến object ngoài pool(non-pool).

- So sánh giá trị 2 biến `String` bằng toán tử `equal`

```java
System.out.println("Compare value str1.equals(str3): " + (str1.equals(str3)));
```

```shell
$ mvn clean package
$ java -jar string-pool/target/string-pool-0.0.1-SNAPSHOT.jar
Compare value str1.equals(str3): true
```` 