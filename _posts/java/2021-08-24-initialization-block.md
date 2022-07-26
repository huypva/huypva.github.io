---
layout: post
title: Java - Initialization block
date: 2021-08-24 10:00:20 +0700
description: Giải thích Initialization block trong Java
img: spring_boot/initialization-spring-boot.png
tags: [Java]
categories: [Java]
source:  https://github.com/huypva/initialization-block-example
---

- *Initialization block* là khối code không thuộc method nào, được thực thi trước contructor của class. Mỗi lần khởi tạo đối tượng sẽ được gọi 1 lần.

```java
public class InitializationBlockExample {

  public InitializationBlockExample() {
    System.out.println("Contructor 1");
  }
 
  {
    System.out.println("Initialization block 1");
  }

  public static void main(String[] args) {
    InitializationBlockExample ibe = new InitializationBlockExample();
  }
}
``` 

Output:

```text
Initialization block 1
Contructor 1
Initialization block 1
Contructor 1
```

- Nếu có 2 block thì thứ tự thực thi theo thứ tự xuất hiện (thứ tự code từ trên xuống).

```java
public class InitializationBlockExample {

  public InitializationBlockExample() {
    System.out.println("Contructor 1");
  }    

  {
    System.out.println("Initialization block 1");
  } 

  {
    System.out.println("Initialization block 2");
  } 

  public static void main(String[] args) {
    InitializationBlockExample ibe = new InitializationBlockExample();
  }
}
``` 

Output:

```text
Initialization block 1
Initialization block 2
Contructor 1
```

- *Static initialization block* tương tự như sử dụng static, được thực thi trước initialization block. Và chỉ được thực thi 1 lần duy nhất.

```java
public class InitializationBlockExample {
  static {
      System.out.println("Static initialization block 1");
  }

  public InitializationBlockExample() {
    System.out.println("Contructor 1");
  }    

  {
    System.out.println("Initialization block 1");
  } 

  {
    System.out.println("Initialization block 2");
  } 

  public static void main(String[] args) {
    InitializationBlockExample ibe = new InitializationBlockExample();
  }
}
```

Output:

```text
Static initialization block 1
Initialization block 1
Contructor 1
Initialization block 1
Contructor 1
```` 

> Thứ tự: Static initialization block -> Super initialization block -> Contructor