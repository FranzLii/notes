# Spring cloud

## 1.eureka 

**微服务注册中心**

### 1.eureka 作用  

Eureka是[Netflix](https://baike.baidu.com/item/Netflix/662557)开发的服务发现框架，本身是一个基于[REST](https://baike.baidu.com/item/REST/6330506)的服务，主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。    

![image-20220331172927087](https://gitee.com/ingachin/mdimage/raw/master/image-20220331172927087.png)

### 2.eureka配置  

#### 1.服务端配置  

---

##### 1.导入依赖  

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

##### 2.配置客户端

```yaml
server:
  port: 8999
spring:
  application:
    name: eurekaserver

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8999/eureka
```

  

#### 2.服务端配置

---

##### 1.导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

##### 2.配置文件

```yaml
spring:
  application:
    name: orderserver

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8999/eureka
```







