---
layout: post
title: Spring Boot - Rate Limiting with Resilience4j
date: 2021-12-15 10:00:20 +0700
description: Hướng dẫn áp dụng Rate Limiting bằng Resilience4j
img: spring_boot/rate_limiting.png
tags: [Spring Boot, Ratelimit]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-rate-limit-example
---

> Hướng dẫn áp dụng Ratelimit bằng Resilience4j

- Thêm dependency trong pom.xml  

{% highlight xml %}
  <!--  resilience4j ratelimit  -->
  <properties>
    <spring-cloud.version>2020.0.4</spring-cloud.version>
  </properties>
  <dependencies>
    <!--  spring cloud circuit breaker  -->
    <dependency>
      <groupId>io.github.resilience4j</groupId>
      <artifactId>resilience4j-spring-boot2</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
  </dependencies>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
{% endhighlight %}

- Thêm config cho ratelimiter  

{% highlight yaml %}
resilience4j.ratelimiter:
  instances:
    serviceA_greet:
      limitForPeriod: 5
      limitRefreshPeriod: 1s
      timeoutDuration: 0
{% endhighlight %}

>> Ý nghĩa: Số lượng ***limitForPeriod*** request trong một khoảng thời gian ***limitRefreshPeriod***.
>> timeoutDuration: thời gian đợi sau khi đạt ngưỡng

- Thêm annotation ***@RateLimiter*** trên method cần ratelimit  

{% highlight java %}
@RestController
public class Controller {

  @GetMapping("/greet/{id}")
  @RateLimiter(name = "serviceA_greet", fallbackMethod = "greetFallBack")
  public ResponseEntity greet(@PathVariable(name = "id") int id) {
    log.info("ServiceA.greet");
    Greeting greeting = new Greeting(id, "Hello ServiceA");

    return ResponseEntity.ok()
        .body(GsonUtils.toJson(greeting));
  }

  public ResponseEntity greetFallBack(int id, Throwable t) {
    log.error("Rate limit applied");
    return ResponseEntity
        .status(HttpStatus.TOO_MANY_REQUESTS)
        .build();
  }

}
{% endhighlight %}

## Reference

- https://resilience4j.readme.io/docs/ratelimiter
- https://reflectoring.io/rate-limiting-with-springboot-resilience4j/