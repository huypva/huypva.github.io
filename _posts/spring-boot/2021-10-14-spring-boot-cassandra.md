---
layout: post
title: Spring Boot + Cassandra
date: 2021-10-14 10:00:20 +0700
description: Hướng dẫn tích hợp Cassandra trong Spring Boot application
img: spring_boot/cassandra.png
tags: [Spring Boot, Cassandra]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-cassandra-example
---

> Hướng dẫn tích hợp Cassandra trong Spring Boot application  

- Thêm dependency trong pom.xml

{% highlight xml %}
<dependencies>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-boot-starter-data-cassandra</artifactId>
    </dependency>
</dependencies>
{% endhighlight %}

- Thêm cấu hình Cassandra trong file application.yml

{% highlight yaml %}
spring:
  data:
    cassandra:
      contact-points: 127.0.0.1
      port: 9042
      keyspace-name: spring_boot_cassandra_example
{% endhighlight %}

- Tạo class entity mapping table trong cassandra

{% highlight java %}
@Data
@Table("User")
public class UserEntity {

  @PrimaryKey("user_id")
  private long userId;

  @Column("user_name")
  private String userName;

}
{% endhighlight %}

- Tạo Repository thao tác với table

{% highlight java %}
@Repository
public interface UserRepository extends CassandraRepository<UserEntity, Long> {

}
{% endhighlight %}

- Sử dụng Repository để thực hiện query đến database

{% highlight java %}
  @Autowired
  UserRepository userRepository;
    
  @Override
  public Greeting greet(long userId) {
    UserEntity userEntity = userRepository.findById(userId).get();
    return new Greeting(userEntity.getUserId(), String.format("Hello %s!", userEntity.getUserName()));
  }
{% endhighlight %}