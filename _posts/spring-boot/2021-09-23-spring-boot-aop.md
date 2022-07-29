---
layout: post
title: Spring Boot + AOP
date: 2021-09-23 10:00:20 +0700
description: Hướng dẫn tạo annotation xử lý AOP trong Spring Boot application
img: spring_boot/aop.png
tags: [Spring Boot, AOP]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-aop-example
---

> Hướng dẫn sử dụng Spring AOP để xử lý cho annotation trong Spring Boot application

- Thư viện sử dụng
  - [spring-boot-starter-aop](https://docs.spring.io/spring-framework/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html){:target="_blank"}

- Thêm dependency trong file pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
</dependencies>    
```

- Tạo annotation

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TimeLogger {

  String value() default "";
}
```

- Tạo class xử lý annotation

```java
@Aspect
@Component
public class TimeLoggerAspectJ {

  @Pointcut("@annotation(timeLogger)")
  public void annotatedTimeLogger(TimeLogger timeLogger) {
  }

  @Around("annotatedTimeLogger(timeLogger)")
  public Object around(ProceedingJoinPoint joinPoint, TimeLogger timeLogger) throws Throwable {
    String value = timeLogger.value();
    long startTime = System.currentTimeMillis();

    try {
      return joinPoint.proceed();
    } finally {

      long processTime = System.currentTimeMillis() - startTime;
      log.info(String.format("Process [%s] in [%s]", value, processTime));
    }

  }
}
```

- Thêm annotaion `EnableAspectJAutoProxy` để kích hoạt Aop

```java
@EnableAspectJAutoProxy
@SpringBootApplication
public class AopApplication {

	public static void main(String[] args) {
		SpringApplication.run(AopApplication.class, args);
	}

}
```