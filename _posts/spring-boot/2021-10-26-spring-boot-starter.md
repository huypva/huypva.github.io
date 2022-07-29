---
layout: post
title: Spring Boot - Starter
date: 2021-10-26 10:00:20 +0700
description: Hướng dẫn viết một library Spring Boot Starter
img: spring_boot/spring_boot_icon.png
tags: [Spring Boot]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-starter-example
---

> Hướng dẫn viết một library Spring Boot Starter

## A Starter project

- Project bao gồm 3 class chính:
    - Class `SampleProperties`: chưa các variable được load từ file config application.yml
    - Class `SampleService`: sample service cần sử dụng
    - Class `SampleAutoConfiguration`: thực hiện việc auto-configuration

- Class `SampleProperties`

```java
@Getter
@Setter
@ConfigurationProperties("sample-code")
public class SampleProperties {

  public boolean active;
  public String value;

}
```

Giá trị được lấy từ config sau thuộc tính `sample-code`

- Class `SampleService` 

```java
public class SampleServiceImpl implements SampleService {

  private SampleProperties sampleProperties;

  public SampleServiceImpl(SampleProperties sampleProperties) {
      this.sampleProperties = sampleProperties;
  }

  @Override
  public void doSample() {
    System.out.println("Do sample: " + sampleProperties.value);
  }
}
```

- Class `SampleAutoConfiguration` thực hiện các việc
    - Chỉ chạy auto khi thuộc tính `sample-code.active=true`
    - Load config theo class `SampleProperties`
    - Tự động tạo bean của class `SampleService`

```java
@Configuration
@ConditionalOnProperty(name = "active", prefix = "sample-code", havingValue = "true")
@EnableConfigurationProperties(SampleProperties.class)
public class SampleAutoConfiguration {

  @ConditionalOnMissingBean
  @Bean
  SampleService sampleService(SampleProperties sampleProperties) {
    return new SampleServiceImpl(sampleProperties);
  }

}
```

- Thêm file spring.factories bên dưới thư mục src/main/resouces/META-INF

```text
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
io.codebyexample.springbootstarter.autoconfiguration.SampleAutoConfiguration

```

## Write a sample

- Thêm config trong file application.yml  

```yaml
sample-code:
  active: true
  value: Sample Value
```

- Lấy bean `SampleService` để sử dụng

```java
@SpringBootApplication
public class Application {

	@Autowired
	SampleService sampleService;

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);

	}

	@EventListener(ApplicationReadyEvent.class)
	public void ready() {
		sampleService.doSample();
	}

}
```