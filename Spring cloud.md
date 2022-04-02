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

  

## 2.ribbon

**实现负载均衡**

### 1.使用

在Restemplete上配置`@LoadBalanced`即可

### 2.原理

![image-20220331194720699](https://gitee.com/ingachin/mdimage/raw/master/image-20220331194720699.png)

### 3.自定义负载均衡策略  

**针对所有服务**

```java
@Bean
public RandomRule randomRule(){
    return new RandomRule();
}
```

**针对特定服务**(只针对userserver服务)

```yaml
userserver:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRuler
```

### 4.设置饥饿加载（默认懒加载）

```yaml
ribbon:
  eager-load:
    enabled: true
    clients: userserver
```

  

## 3.Nacos

**Alibaba提供微服务注册中心**

#### 1.引入nacos依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

#### 2.修改配置文件

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
```

#### 3.配置集群

```yaml
  cloud:
      discovery:
        cluster-name: GZ
```

##### 配置优先使用同集群服务器

```yaml
userserver:
  ribbon:
    NFLoadBalancerRulerClassName: com.alibaba.cloud.nacos.ribbon.NacosRule
```

先优先选择同集群服务后随机

##### 修改负载权重

![](https://gitee.com/ingachin/mdimage/raw/master/image-20220331204630639.png)



### 4.环境隔离 （namespace）

```yaml
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: GZ
        namespace: fad344b8-6b8d-4d83-a5db-49c9c2e3045c
```

不同namespace下的服务**互相不可见**

### 5.Nacos和eureka的区别

#### 1.存活实例检测方法

对于临时实例Nacos和eureka一样发送心跳包

对于非临时实例Nacos主动检测，并且不会踢去

当存在实例状态变更，Nacos主动推送给消费者

#### 2.配置非临时实例

![image-20220331211045678](https://gitee.com/ingachin/mdimage/raw/master/image-20220331211045678.png)

