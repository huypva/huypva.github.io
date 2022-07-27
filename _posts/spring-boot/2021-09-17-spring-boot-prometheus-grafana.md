---
layout: post
title: Spring Boot - Prometheus + Grafana
date: 2021-09-17 10:00:20 +0700
description: Hướng dẫn monitor Spring Boot application bằng Prometheus và Grafana
img: spring_boot/spring_boot_icon.png
tags: [Spring Boot, Monitor, Prometheus, Grafana]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-prometheus-grafana-example
---

> Hướng dẫn monitor Spring Boot application bằng Prometheus và Grafana

- Thư viện sử dụng
  - [spring-boot-actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
  - [Micrometer.io](https://micrometer.io/)
  - [Prometheus](https://prometheus.io/)
  - [Grafana](https://grafana.com/)

## Tạo một Spring Boot application 

- Thêm dependency trong file pom.xml

```xml
<dependencies>
    <!--  actuator  -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!--  micrometer  -->
    <dependency>
      <groupId>io.micrometer</groupId>
      <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
</dependencies>
```

- Thêm configuration cho micrometer trong file application.yml

```yml
management:
  endpoints:
    web:
      exposure:
        include: health, shutdown, prometheus
  metrics:
    tags:
      application: ${spring.application.name}
    export:
      prometheus:
        enabled: true
    distribution:
      percentiles-histogram:
        http: false
      sla:
        http:
          server:
            requests: 1ms,20ms,50ms,100ms,200ms,500ms,1s,2s,5s,10s,10s,50s
```

- Thêm các class cần thiết và start một Restfull service

- Kiểm tra các đường dẫn actuator quản lý

```shell
$ curl -X GET http://localhost:8081/actuator
{"_links":{"self":{"href":"http://localhost:8081/actuator","templated":false},"health":{"href":"http://localhost:8081/actuator/health","templated":false},"health-path":{"href":"http://localhost:8081/actuator/health/{*path}","templated":true},"prometheus":{"href":"http://localhost:8081/actuator/prometheus","templated":false}}}
```

- Xem các metrics cho prometheus

```shell
$ curl -X GET http://localhost:8081/actuator/prometheus
...
http_server_requests_seconds_max{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator/prometheus",} 0.2230911
http_server_requests_seconds_max{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator",} 0.2658599
http_server_requests_seconds_max{exception="None",method="GET",outcome="CLIENT_ERROR",status="404",uri="/**",} 0.033111
# HELP process_cpu_usage The "recent cpu usage" for the Java Virtual Machine process
# TYPE process_cpu_usage gauge
process_cpu_usage 0.009735744089012517
...
```

## Cài đặt prometheus & grafana

- File docker-compose.yml

```yaml
version: "3.4"

services:
  prometheus:
    image: "prom/prometheus"
    volumes:
      - ./infrastructure/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    container_name: prometheus
    networks:
      - prometheus-grafana-network
  grafana:
    image: "grafana/grafana"
    ports:
      - "3000:3000"
    container_name: grafana
    networks:
      - prometheus-grafana-network
    volumes:
      - grafana-storage:/var/lib/grafana
volumes:
  grafana-storage:
    external: true
networks:
  prometheus-grafana-network:
```

- Với file config prometheus.yml

```yaml
scrape_configs:
  - job_name: 'spring-boot-prometheus-grafana-metrics'
    metrics_path: '/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: [ 'spring-boot-prometheus-grafana:8081' ]
```

- Start docker-compose

```shell
$ docker-compose up -d
```

- Mở browser và truy cập tool qua link sau
    - Prometheus: http://localhost:9090/
    - Grafana: http://localhost:3000/ , tài khoản mặc định là admin/admin

## Tạo dashboard trên Grafana

### Dashboard monitor JVM metric

- Setup Prometheus datasource

![setup ds prometheus](../../assets/images/spring_boot/prometheus_grafana/setup_ds_prometheus.png)

- Tạo dashboard cho JVM metric bằng cách import id [4701](https://grafana.com/grafana/dashboards/4701)

![jvm_1](../../assets/images/spring_boot/prometheus_grafana/import_jvm_metric_1.png)
![jvm_2](../../assets/images/spring_boot/prometheus_grafana/import_jvm_metric_2.png)

- Dashboard hiển thị như bên dưới

![jmv metrics](../../assets/images/spring_boot/prometheus_grafana/grafana_jvm_metrics.png)


### Dashboard monitor api 
- Success Rate
    + Metric 1
    
```text
sum(increase(http_server_requests_seconds_bucket{application="$application", instance="$instance", status="200"}[1m]))
/
sum(increase(http_server_requests_seconds_bucket{application="$application", instance="$instance"}[1m]))
```

Metric 2
    
```text
sum(increase(http_server_requests_seconds_bucket{application="$application", instance="$instance", status!="200"}[1m]))
/
sum(increase(http_server_requests_seconds_bucket{application="$application", instance="$instance"}[1m]))
```


- Metric B

```text
sum(increase(http_server_requests_seconds_bucket{uri="/greeting", status!="200"}[1m]))
/
sum(increase(http_server_requests_seconds_bucket{uri="/greeting"}[1m]))
```

- Throughput

```text
sum(rate(http_server_requests_seconds_bucket{application="$application", instance="$instance"}[1m])) by (uri)
```

- Latency P99

```text
histogram_quantile(0.99, sum(rate(http_server_requests_seconds_bucket{application="$application", instance="$instance"}[1m])) by (le, uri))
```

![extra metrics](../../assets/images/spring_boot/prometheus_grafana/grafana_extra_metrics.png)
