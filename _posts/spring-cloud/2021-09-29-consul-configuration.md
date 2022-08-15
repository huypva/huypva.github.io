---
layout: post
title: Spring Cloud + Configuration with Consul
date: 2021-09-27 10:00:20 +0700
description: Hướng dẫn sử dụng Consul configuration trong Spring Boot application
img: spring_cloud/spring_cloud.png
tags: [Spring Cloud, Consul]
categories: [Spring Cloud]
source: https://github.com/huypva/spring-boot-consul-example
---

> Hướng dẫn sử dụng Consul configuration trong Spring Boot application

## Tạo ứng dụng

- Trong ứng dụng Spring Boot, config được đặt trong file application.yml (hoặc .properties).

Tạo một project với file configuration application.yml như sau

{% highlight yaml %}
server:
  port : 8081
spring:
  application:
    name: "spring-boot-consul"
my-config:
  id: 1
  value: my value
{% endhighlight %}

- Tạo class lấy config từ file 

{% highlight java %}
@Getter
@Setter
@Configuration
@ConfigurationProperties(prefix = "my-config")
public class MyConf {

  private int id;
  private String value;

}
{% endhighlight %}

- Tạo class autowired bean **MyConf** và sử dụng

{% highlight java %}
    @Autowired
    MyConf myConf;

	@Value("${spring.application.name}")
	String applicationName;

	@EventListener(ApplicationReadyEvent.class)
	protected void readyProcess() {
		System.out.println("Id: " + myConf.getId());
		System.out.println("Value: " + myConf.getValue());

		System.out.println("Application name: " + applicationName);
	}
{% endhighlight %}

- Kết quả sau khi chạy chương trình

{% highlight shellscipt %}
$ ./mvnw spring-boot:run
2021-09-29 10:18:22.194  INFO 2892 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8081 (http) with context path ''
2021-09-29 10:18:22.201  INFO 2892 --- [           main] i.c.springbootconsul.Application         : Started Application in 3.853 seconds (JVM running for 4.93)
Id: 1
Value: my value
Application name: spring-boot-consul
{% endhighlight %}

## Sử dụng configuration với Consul

- Thêm dependency [spring-cloud-starter-consul-config](https://cloud.spring.io/spring-cloud-consul/reference/html/){:target="_blank"}

{% highlight xml %}
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-consul-config</artifactId>
        <version>2.2.7.RELEASE</version>
    </dependency>
</dependencies>
{% endhighlight %}

- Tạo file bootstrap.yml

{% highlight yaml %}
spring:
  profiles:
    active: local
  cloud:
    consul:
      host: localhost
      port: 8500
      config:
        enabled: true
        format: YAML
        prefix: prefix #default: config
        name: name
        default-context: context #default: application
        profile-separator: '::'
        data-key: key
{% endhighlight %}

  - *Lưu ý*: Project sẽ đọc tất cả các nội dụng trong các path sau:
    - [prefix]/[name]/[key]
    - [prefix]/[name][profile-separator][spring.profiles.active]/[key]
    - [prefix]/[default-context]/[key]
    - [prefix]/[default-context][profile-separator][spring.profiles.active]/[key]

- Sử dung ui của Consul tại địa chỉ http://localhost:8500/ui/dc1/kv để thêm nội dung giống như application.yml tại path /prefix/name/key/ 

Ở đây, ví dụ có thay đổi một số giá trị **value** trong config trên Consul để dễ kiểm tra

<div align="center">
    <img src="/assets/img/spring_cloud/consul_configuration.jpeg"/>
</div>

- Kết quả sau khi chạy chương trình

{% highlight text %}
$ ./mvnw spring-boot:run
...
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.6.RELEASE)

2021-09-29 10:34:46.467  INFO 3429 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: [BootstrapPropertySource {name='bootstrapProperties-prefix/name::local/'}, BootstrapPropertySource {name='bootstrapProperties-prefix/name/'}, BootstrapPropertySource {name='bootstrapProperties-prefix/context::local/'}, BootstrapPropertySource {name='bootstrapProperties-prefix/context/'}]
2021-09-29 10:34:46.472  INFO 3429 --- [           main] i.c.springbootconsul.Application         : The following profiles are active: local
2021-09-29 10:34:46.817  INFO 3429 --- [           main] o.s.cloud.context.scope.GenericScope     : BeanFactory id=0adf9a47-44e9-34ef-9488-1afeb9ef43b8
2021-09-29 10:34:46.999  INFO 3429 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8081 (http)
2021-09-29 10:34:47.006  INFO 3429 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-09-29 10:34:47.006  INFO 3429 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.33]
2021-09-29 10:34:47.094  INFO 3429 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-09-29 10:34:47.095  INFO 3429 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 609 ms
2021-09-29 10:34:47.214  INFO 3429 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2021-09-29 10:34:47.326  INFO 3429 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'configWatchTaskScheduler'
2021-09-29 10:34:47.410  INFO 3429 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8081 (http) with context path ''
2021-09-29 10:34:47.415  INFO 3429 --- [           main] i.c.springbootconsul.Application         : Started Application in 2.029 seconds (JVM running for 2.308)
Id: 1
Value: my value from consul
Application name: spring-boot-consul
{% endhighlight %}

*Lưu ý*: Sau khi sử dụng config trên Consul, nên xóa file config ***application.yml*** để tránh nhầm lẫn