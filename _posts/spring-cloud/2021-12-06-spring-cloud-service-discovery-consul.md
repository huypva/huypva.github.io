---
layout: post
title: Spring Cloud + Service Discovery with Consul
date: 2021-12-06 10:00:20 +0700
description: Hướng dẫn sử dụng Service Discovery với Consul
img: spring_boot/spring_boot_icon.png
tags: [Spring Cloud, Service Discovery]
categories: [Spring Cloud]
source: https://github.com/huypva/spring-boot-service-discovery-consul-example
---

> Hướng dẫn sử dụng Service Discovery với Consul

## Thư viện sử dụng

- Spring Boot: version 2.4.11
- Spring Cloud: version 2020.0.4
    - org.springframework.cloud:spring-cloud-starter-openfeign
    - org.springframework.cloud:spring-cloud-starter-consul-discovery

## Kiến trúc demo

<div align="center">
    <img src="./assets/img/spring_cloud/consul_example.png"/>
</div>

## Cài đặt Consul service bằng docker-compose

- Tạo file docker-compose.yml

```yaml
version: "3.4"
services:
  consul:
    image: bitnami/consul:latest
    container_name: consul
    volumes:
      - ./data/consul:/bitnami
    ports:
      - '8300:8300'
      - '8301:8301'
      - '8301:8301/udp'
      - '8500:8500'
      - '8600:8600'
      - '8600:8600/udp'
```

- Start Consul bằng command

```shell script
$ docker-compose up -d
```

- Truy cập Consul admin trên browser tại địa chỉ `http://localhost:8500/`

<div align="center">
    <img src="assets/img/spring_cloud/start.png"/>
</div>

## Tạo services

### Tạo client service-A

- Thêm dependencies trong pom.xml

```xml
    <parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.11</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

    <properties>
        <spring-cloud.version>2020.0.4</spring-cloud.version>
    </properties>

    <dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-consul-discovery</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
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
```

- Thêm config Consul trong application.yml

```yaml
spring:
  cloud:
    consul:
      host: localhost
      port: 8500
      enabled: true
      discovery:
        register: true
        instanceId: "${spring.application.name}-${server.port}-${spring.cloud.client.ip-address}"
        prefer-ip-address: true
        health-check-critical-timeout: "1m"
      config:
        enabled: false
```

- Enable Feign and discovery client

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class ServiceAApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServiceAApplication.class, args);
	}

}
```

- Tao Feign client

Thêm config path 

```yaml
service-B:
  path:
    greet: /greetb/{id}
```

Code client

```java
@FeignClient(value = "service-B")
public interface ServiceBClient {

  @RequestMapping(method = RequestMethod.GET, value = "${service-B.path.greet}")
  MessageB greet(@PathVariable(name = "id") int id);

}
```

### Tạo server service-B

- Thêm dependencies trong pom.xml

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.11</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <spring-cloud.version>2020.0.4</spring-cloud.version>
    </properties>

    <dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-consul-discovery</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
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
```

- Thêm config Consul trong application.yml

```yaml
spring:
  cloud:
    consul:
      host: localhost
      port: 8500
      enabled: true
      discovery:
        register: true
        instanceId: "${spring.application.name}-${server.port}-${spring.cloud.client.ip-address}"
        prefer-ip-address: true
        health-check-critical-timeout: "1m"
      config:
        enabled: false
```

- Enable Feign and discovery client

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class ServiceAApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServiceAApplication.class, args);
	}

}
```

