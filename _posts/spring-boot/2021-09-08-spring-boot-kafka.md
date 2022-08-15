---
layout: post
title: Spring Boot + Kafka
date: 2021-09-08 10:00:20 +0700
description: Hướng dẫn tích hợp Kafka trong Spring Boot application
img: spring_boot/kafka.png
tags: [Spring Boot, Kafka]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-kafka-example
---

> Hướng dẫn tích hợp Kafka trong Spring Boot application

### Configuration

- Thêm dependency trong pom.xml

{% highlight xml %}
<dependencies>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
</dependencies>
{% endhighlight %} 

- Thêm cấu hình kafa trong file application.yml

{% highlight yaml %}
spring:
  kafka:
    bootstrap-servers: "localhost:9093"
    listener:
      missing-topics-fatal: false
{% endhighlight %}

### Write a producer

- Sử dụng bean ***KafkaTemplate*** để gửi một message lên topic **UserMessage** của Kafka

{% highlight java %}
    @Autowired
    KafkaTemplate<String, String> userKafka;
  
    @Override
    public void sendUserMesage(UserMessage message) {
      userKafka.send("UserMessage", GsonUtils.toJson(message))
          .addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
  
            @Override
            public void onSuccess(SendResult<String, String> result) {
              log.info("Kafka sent message='{}' with offset={}", message,
                  result.getRecordMetadata().offset());
            }
  
            @Override
            public void onFailure(Throwable ex) {
              log.error("Kafka unable to send message='{}'", message, ex);
            }
          });
    }
{% endhighlight %}

### Write a consumer

{% highlight java %}
  @Autowired
  UserUseCase userUseCase;
  
  @KafkaListener(topics = "UserMessage", groupId = "example")
  public void consume(ConsumerRecord<String, String> record) {
    try {
      log.info(
          "Consumed - Partition: {} - Offset: {} - Value: {}",
          record.partition(),
          record.offset(),
          record.value());
  
      UserMessage userMessage = GsonUtils.fromJson(record.value(), UserMessage.class);
      userUseCase.goodbye(userMessage);
  
    } catch (Exception ex) {
      log.error("Exception - Reason:", ex);
    }
  }    
{% endhighlight %}
