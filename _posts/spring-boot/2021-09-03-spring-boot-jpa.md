---
layout: post
title: Spring Boot + JPA
date: 2021-08-30 10:00:20 +0700
description: Hướng dẫn sử dụng jpa giao tiếp database trong Spring Boot
img: spring_boot/jpa.png
tags: [Spring Boot, MySQL]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-jpa-example
---

Hướng dẫn sử dụng jpa giao tiếp database trong Spring Boot

- Thư viện sử dụng  
  - [spring-data-jpa](https://spring.io/projects/spring-data-jpa){:target="_blank"}

- Định nghĩa dependency trong pom.xml

{% highlight xml %}
<dependencies>
    <!--spring mvc, rest-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--spring data jpa-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!--mysql connector java-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
</dependencies>
{% endhighlight %}

- Cấu hình datasource & jpa trong application.yml

{% highlight yaml %}
spring:
    datasource:
        url: "jdbc:mysql://localhost:3306/spring_boot_jpa_example"
        username: user
        password: password
        driver-class-name: com.mysql.cj.jdbc.Driver
    jpa:
        database-platform: org.hibernate.dialect.MySQL5Dialect
        show-sql: true
        hibernate:
          ddl-auto: none
{% endhighlight %} 

- Tạo class entity tương ứng với table trong database

{% highlight java %}
@Entity
@Table(name = "user")
public class UserDto {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "user_id")
  private int userId;

  @Column(name = "user_name")
  private String userName;
}
{% endhighlight %} 

- Tạo Repository thao tác với table

{% highlight java %}
public interface UserRepository extends JpaRepository<UserDto, Integer> {

  Optional<UserDto> findUserByUserName(String username);
}
{% endhighlight %}

- Query database thông qua repository

{% highlight java %}
  @Autowired
  private final UserRepository userRepository;
  
  @Override
  public UserDto getUserByName(String username) {
      return userRepository.findUserByUserName(username)
          .orElseThrow(() -> new UserNotFoundException(String.format("Not found user by name %s", username)));
  }
{% endhighlight %}