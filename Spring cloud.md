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



#### 4.环境隔离 （namespace）

```yaml
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: GZ
        namespace: fad344b8-6b8d-4d83-a5db-49c9c2e3045c
```

不同namespace下的服务**互相不可见**

#### 5.Nacos和eureka的区别

##### 1.存活实例检测方法

对于临时实例Nacos和eureka一样发送心跳包

对于非临时实例Nacos主动检测，并且不会踢去

当存在实例状态变更，Nacos主动推送给消费者

##### 2.配置非临时实例

![image-20220331211045678](https://gitee.com/ingachin/mdimage/raw/master/image-20220331211045678.png)

#### 6.Nacos实现配置管理  

##### 1.Nacos配置管理  



![image-20220403100321980](https://gitee.com/ingachin/mdimage/raw/master/image-20220403100321980.png)

##### 2.微服务配置拉取  

###### 1.引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```



###### 2.配置`bootstrap.yml`文件  

```yaml
spring:
  application:
    name: userserver
  profiles:
    active: dev
  cloud:
    nacos:
      server-addr: localhost:8848
      config:
        file-extension: yaml
```



###### 3.读取配置文件

```java
@Value("${pattern.datetimeformate}")
private String dateformate;
```

##### 3.实现配置热更新

###### 方式1

在类上添加`@RefreshScope`注解

##### 4.多环境共享

![image-20220403103011391](https://gitee.com/ingachin/mdimage/raw/master/image-20220403103011391.png)

##### 5.配置环境优先级

![image-20220403103436711](https://gitee.com/ingachin/mdimage/raw/master/image-20220403103436711.png)

#### 7.Nacos集群搭建

搭建集群的基本步骤：

- 搭建数据库，初始化数据库表结构
- 下载nacos安装包
- 配置nacos
- 启动nacos集群
- nginx反向代理  

###### 1.配置数据库

```sql
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `src_user` text,
  `src_ip` varchar(50) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';


CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE `users` (
	`username` varchar(50) NOT NULL PRIMARY KEY,
	`password` varchar(500) NOT NULL,
	`enabled` boolean NOT NULL
);

CREATE TABLE `roles` (
	`username` varchar(50) NOT NULL,
	`role` varchar(50) NOT NULL,
	UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE
);

CREATE TABLE `permissions` (
    `role` varchar(50) NOT NULL,
    `resource` varchar(255) NOT NULL,
    `action` varchar(8) NOT NULL,
    UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```

##### 2.配置Nacos  

###### 1.配置Nacos

将这个包解压到任意非中文目录下，如图：

目录说明：

- bin：启动脚本
- conf：配置文件



进入nacos的conf目录，修改配置文件cluster.conf.example，重命名为cluster.conf：

然后添加内容：

```
127.0.0.1:8845
127.0.0.1.8846
127.0.0.1.8847
```



然后修改application.properties文件，添加数据库配置

```properties
spring.datasource.platform=mysql

db.num=1

db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=123
```



###### 2.启动

将nacos文件夹复制三份，分别命名为：nacos1、nacos2、nacos3

然后分别修改三个文件夹中的application.properties，

nacos1:

```properties
server.port=8845
```

nacos2:

```properties
server.port=8846
```

nacos3:

```properties
server.port=8847
```



然后分别启动三个nacos节点：

```
startup.cmd
```

###### 3.nigix配置

修改conf/nginx.conf文件，配置如下：

```nginx
upstream nacos-cluster {
    server 127.0.0.1:8845;
	server 127.0.0.1:8846;
	server 127.0.0.1:8847;
}

server {
    listen       80;
    server_name  localhost;

    location /nacos {
        proxy_pass http://nacos-cluster;
    }
}
```

而后在浏览器访问：http://localhost/nacos即可。

## 4.Fegin

### 1.Fegin配置

###### 1.导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

###### 2.配置注解

`@EnableFeignClients`

###### 3.编写Feign客户端

```java
@FeignClient("userserver")
public interface UserClient {
    @GetMapping("/user/{id}")
    public User getById(@PathVariable Long id);
}
```

### 2.自定义Fegin配置

![image-20220403203627885](https://gitee.com/ingachin/mdimage/raw/master/image-20220403203627885.png)

![image-20220403204157467](https://gitee.com/ingachin/mdimage/raw/master/image-20220403204157467.png)

![image-20220403204421531](https://gitee.com/ingachin/mdimage/raw/master/image-20220403204421531.png)

### 3.Fegin性能优化

![image-20220403204757819](https://gitee.com/ingachin/mdimage/raw/master/image-20220403204757819.png)

#### 1.引入依赖

```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

#### 2.添加配置

```yaml
feign:
  httpclient:
    enabled: true
    max-connections: 200
    max-connections-per-route: 50
```

### 4.Fegin最佳实践

##### 1.方式1

**不推荐**

![image-20220403205646895](https://gitee.com/ingachin/mdimage/raw/master/image-20220403205646895.png)

##### 2.方式2

![image-20220403205953562](https://gitee.com/ingachin/mdimage/raw/master/image-20220403205953562.png)

![image-20220403210911043](https://gitee.com/ingachin/mdimage/raw/master/image-20220403210911043.png)

## 5.Gateway

### 1.引入依赖

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
</dependencies>
```

### 2.编写配置

```yaml
server:
  port: 10010
spring:
  application:
    name: gateway
  cloud:
    nacos:
      server-addr: localhost:8848
    gateway:
      routes:
        - id: userserver
          uri: lb://userserver
          predicates:
            - Path=/user/**
```

### 3.过滤器配置

#### 1.默认过滤器  



```yaml
spring:
	...
    gateway:
      routes:
	...
          filters:
            - AddRequestHeader=True, 添加到请求头的内容
```

#### 2.default过滤器  



```yaml
spring:
  application:
    name: gateway
	...
    gateway:
	...
      default-filters:
        - - AddRequestHeader=True, 添加到请求头的内容
```

#### 3.global过滤器  (自定义过滤器)  

##### 1.编写filter实现`GlobalFilter`接口

```java
@Order//过滤器优先级
@Component
public class MyGlobalFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String authorization = request.getHeaders().getFirst("Authorization");
        if ("admin".equals(authorization)){//放行
            Mono<Void> filter = chain.filter(exchange);
            return filter;
        }
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);//拦截
        return exchange.getResponse().setComplete();
    }
}
```

![image-20220405161248099](https://gitee.com/ingachin/mdimage/raw/master/image-20220405161248099.png)



#### 4.顺序指定

![image-20220405161808250](https://gitee.com/ingachin/mdimage/raw/master/image-20220405161808250.png)

![image-20220405161905816](https://gitee.com/ingachin/mdimage/raw/master/image-20220405161905816.png)

### 4.跨域问题处理

![image-20220405162149149](https://gitee.com/ingachin/mdimage/raw/master/image-20220405162149149.png)

## 6.Docker

### 1.常用命令

![image-20220405165952256](https://gitee.com/ingachin/mdimage/raw/master/image-20220405165952256.png)

![image-20220405172025706](https://gitee.com/ingachin/mdimage/raw/master/image-20220405172025706.png)

### 2.创建容器

![image-20220405195440005](https://gitee.com/ingachin/mdimage/raw/master/image-20220405195440005.png)

![image-20220405200656812](https://gitee.com/ingachin/mdimage/raw/master/image-20220405200656812.png)

### 3.数据卷

![image-20220405201652680](https://gitee.com/ingachin/mdimage/raw/master/image-20220405201652680.png)

#### 挂在数据卷到容器

![image-20220405202000555](https://gitee.com/ingachin/mdimage/raw/master/image-20220405202000555.png)

### 4.将宿主机目录挂载到容器目录

![image-20220405203210119](https://gitee.com/ingachin/mdimage/raw/master/image-20220405203210119.png)

#### 5.数据卷和宿主机目录挂载区别

![image-20220405203845940](https://gitee.com/ingachin/mdimage/raw/master/image-20220405203845940.png)

### 5.Dockerfile自定义镜像

![image-20220405204510393](https://gitee.com/ingachin/mdimage/raw/master/image-20220405204510393.png)

![image-20220405210713311](https://gitee.com/ingachin/mdimage/raw/master/image-20220405210713311.png)

![image-20220405211719922](https://gitee.com/ingachin/mdimage/raw/master/image-20220405211719922.png)

### 6.Dockercompose  

![image-20220405212223466](https://gitee.com/ingachin/mdimage/raw/master/image-20220405212223466.png)

![image-20220405213617502](https://gitee.com/ingachin/mdimage/raw/master/image-20220405213617502.png)

## 7.MQ  

### 1.同步调用的利弊  （Fegin）

![image-20220406171429422](https://gitee.com/ingachin/mdimage/raw/master/image-20220406171429422.png)



### 2.异步通讯的优缺点



![image-20220406172212412](https://gitee.com/ingachin/mdimage/raw/master/image-20220406172212412.png)

### 3.不同MQ的区别

![image-20220406193017952](https://gitee.com/ingachin/mdimage/raw/master/image-20220406193017952.png)

### 4.RabiitMQ

#### 1.RabbitMQ配置

##### 1.RabbitMQ安装

```bash
docker pull rabbitmq:3-management
```

##### 2.运行RabbitMQ

```bash
docker run \
 -e RABBITMQ_DEFAULT_USER=itcast \
 -e RABBITMQ_DEFAULT_PASS=123321 \
 --name mq \
 --hostname mq1 \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3-management
```

#### 2.Spring AMQP使用

##### 1.导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

##### 2.配置application.yml文件

```yaml
spring:
  rabbitmq:
    host: 192.168.117.128
    port: 5672
    virtual-host: /
    username: itcast
    password: 123321
```

##### 3.使用RabbitTemplate完成实例

###### 1.Helloworld

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PublisherTest {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testSendMessage() throws IOException, TimeoutException {
        String queueName = "simple.queue";
        String message = "123123";
        rabbitTemplate.convertAndSend(queueName,message);
    }
}
```

**Cosumer**

```java
@Component
public class rabbitListener {
    @RabbitListener(queues = "simple.queue")
    public void listen(String s){
        System.out.println(s);
    }
}
```

###### 2.Wrokshop  

![image-20220409195043853](https://gitee.com/ingachin/mdimage/raw/master/image-20220409195043853.png)

配置perfetch使其达到负载均衡

```xml
spring:
...
    listener:
      simple:
        prefetch: 1
```



###### 3.发布订阅模型

一条消息发给多个消费者

![image-20220409195329752](https://gitee.com/ingachin/mdimage/raw/master/image-20220409195329752.png)



**Fanout**

Fanout Exchange 会将收到的消息路由到每一个跟其绑定的queue

```java
@Configuration
public class FanoutConfig {
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("fanout.exchange");
    }
    @Bean
    public Queue queue1(){
        return new Queue("simple.queue1");
    }
    @Bean
    public Binding fanoutBinding(Queue queue1,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue1).to(fanoutExchange);
    }
    @Bean
    public Queue queue2(){
        return new Queue("simple.queue2");
    }
    @Bean
    public Binding fanoutBinding2(Queue queue2,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue2).to(fanoutExchange);
    }

}
```

```java
@Test
public void testsentoexchange(){
    String exchangeName = "fanout.exchange";
    String message = "hello,every one";
    rabbitTemplate.convertAndSend(exchangeName,"",message);
}
```



**DirectExchange**

> Direct Exchange 会将接收到的消息根据路由规则路由到指定的Queue

![image-20220409202055807](https://gitee.com/ingachin/mdimage/raw/master/image-20220409202055807.png)

当不指定路由规则时 = FanoutExchange

基于`@RabbitListener`声明Exchange,Queue,RoutingKey

消费者代码

```java
@RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "direct.queue2"),
            exchange = @Exchange(name = "direct.exchange",type = ExchangeTypes.DIRECT),
            key = {"yello"}
    ))
    public void directQueue2(String msg){
        System.out.println("[yello]"+"========>"+msg);
    }
```

Publisher代码

```java
@Test
public void testDirectExchange(){
    String exchangeName = "direct.exchange";
    String message = "hello,every one";
    rabbitTemplate.convertAndSend(exchangeName,"red",message);
}
```



**TopicExchange**

> TopicExchange 和 DirectExchange 类似，区别在于routingKey必须是多个单词的列表，并且以.为分隔

![image-20220409203932179](https://gitee.com/ingachin/mdimage/raw/master/image-20220409203932179.png)

  

###### 4.Spring-AMQP消息转换器

Spring 的消息对象处理默认是由MessageConverter处理的，默认实现为SimpleMessageConverter，基于JDK的ObjectOutputStream完成序列化

推荐修改为JSON方式完成序列化

![image-20220409205638506](https://gitee.com/ingachin/mdimage/raw/master/image-20220409205638506.png)

  

## 8.elasticsearch

  ### 1.基本原理

![image-20220409210311770](https://gitee.com/ingachin/mdimage/raw/master/image-20220409210311770.png)

![image-20220409210343901](https://gitee.com/ingachin/mdimage/raw/master/image-20220409210343901.png)

 正排索引和倒排索引的区别

正向索引：按文档逐个查询，类似于关系数据表里逐条数据查询，并以like模糊匹配
优点：易于维护：新增的话直接跟在原来的后面，删除的话直接删除某一条即可
缺点：查询时间长，检索效率低下

![image-20220409211521019](https://gitee.com/ingachin/mdimage/raw/master/image-20220409211521019.png)

倒排索引：倒排表以字或词为关键字进行索引，表中关键字所对应的记录表项记录了出现这个字或词的所有文档，一个表项就是一个字表段，它记录该文档的ID和字符在该文档中出现的位置情况。
 优点：检索的快速响应是一个最为关键的性能
 缺点：倒排表的建立和维护都较为复杂（索引建立由于在后台进行，尽管效率相对低一些，但不会影响整个搜索引擎的效率）  

![image-20220409211503409](https://gitee.com/ingachin/mdimage/raw/master/image-20220409211503409.png)

### 2.Mysql和es概念区别

![image-20220409211945496](https://gitee.com/ingachin/mdimage/raw/master/image-20220409211945496.png)

![image-20220409212008448](https://gitee.com/ingachin/mdimage/raw/master/image-20220409212008448.png)

![image-20220409212039518](https://gitee.com/ingachin/mdimage/raw/master/image-20220409212039518.png)

![image-20220409212438687](https://gitee.com/ingachin/mdimage/raw/master/image-20220409212438687.png)

### 3.es部署

#### 1.创建网络    

因为我们还需要部署kibana容器，因此需要让es和kibana容器互联。这里先创建一个网络：

```bash
docker network create es-net
```

 

#### 2.下载镜像

```bash
docker pull elasticsearch
```

  

#### 3.部署容器  

运行docker命令，部署单点es：

```sh
docker run -d \
	--name es \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/share/elasticsearch/data \
    -v es-plugins:/usr/share/elasticsearch/plugins \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:7.12.1
```

命令解释：

- `-e "cluster.name=es-docker-cluster"`：设置集群名称
- `-e "http.host=0.0.0.0"`：监听的地址，可以外网访问
- `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"`：内存大小
- `-e "discovery.type=single-node"`：非集群模式
- `-v es-data:/usr/share/elasticsearch/data`：挂载逻辑卷，绑定es的数据目录
- `-v es-logs:/usr/share/elasticsearch/logs`：挂载逻辑卷，绑定es的日志目录
- `-v es-plugins:/usr/share/elasticsearch/plugins`：挂载逻辑卷，绑定es的插件目录
- `--privileged`：授予逻辑卷访问权
- `--network es-net` ：加入一个名为es-net的网络中
- `-p 9200:9200`：端口映射配置

### 4.部署kibana

kibana可以给我们提供一个elasticsearch的可视化界面，便于我们学习。

#### 2.1.部署

运行docker命令，部署kibana

```sh
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es:9200 \
--network=es-net \
-p 5601:5601  \
kibana:7.12.1
```

- `--network es-net` ：加入一个名为es-net的网络中，与elasticsearch在同一个网络中
- `-e ELASTICSEARCH_HOSTS=http://es:9200"`：设置elasticsearch的地址，因为kibana已经与elasticsearch在一个网络，因此可以用容器名直接访问elasticsearch
- `-p 5601:5601`：端口映射配置

kibana启动一般比较慢，需要多等待一会，可以通过命令：

```sh
docker logs -f kibana
```

查看运行日志，当查看到下面的日志，说明成功：

### 5.IK分词器  

####  在线安装ik插件

```shell
# 进入容器内部
docker exec -it elasticsearch /bin/bash

# 在线下载并安装
./bin/elasticsearch-plugin  install https://ghproxy.com/https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip

#退出
exit
#重启容器
docker restart elasticsearch
```

IK分词器包含两种模式：

* `ik_smart`：最少切分
* `ik_max_word`：最细切分    

####  扩展词词典

随着互联网的发展，“造词运动”也越发的频繁。出现了很多新的词语，在原有的词汇列表中并不存在。比如：“奥力给”，“传智播客” 等。

所以我们的词汇也需要不断的更新，IK分词器提供了扩展词汇的功能。

1）打开IK分词器config目录：

2）在IKAnalyzer.cfg.xml配置文件内容添加：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 *** 添加扩展词典-->
        <entry key="ext_dict">ext.dic</entry>
</properties>
```

3）新建一个 ext.dic，可以参考config目录下复制一个配置文件进行修改

```properties
传智播客
奥力给
```

4）重启elasticsearch 

```sh
docker restart es

# 查看 日志
docker logs -f elasticsearch
```

日志中已经成功加载ext.dic配置文件

5）测试效果：

```json
GET /_analyze
{
  "analyzer": "ik_max_word",
  "text": "传智播客Java就业超过90%,奥力给！"
}
```

> 注意当前文件的编码必须是 UTF-8 格式，严禁使用Windows记事本编辑

####  停用词词典

在互联网项目中，在网络间传输的速度很快，所以很多语言是不允许在网络上传递的，如：关于宗教、政治等敏感词语，那么我们在搜索时也应该忽略当前词汇。

IK分词器也提供了强大的停用词功能，让我们在索引时就直接忽略当前的停用词汇表中的内容。

1）IKAnalyzer.cfg.xml配置文件内容添加：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典-->
        <entry key="ext_dict">ext.dic</entry>
         <!--用户可以在这里配置自己的扩展停止词字典  *** 添加停用词词典-->
        <entry key="ext_stopwords">stopword.dic</entry>
</properties>
```

3）在 stopword.dic 添加停用词

```properties
习大大
```

4）重启elasticsearch 

```sh
# 重启服务
docker restart elasticsearch
docker restart kibana

# 查看 日志
docker logs -f elasticsearch
```

日志中已经成功加载stopword.dic配置文件

5）测试效果：

```json
GET /_analyze
{
  "analyzer": "ik_max_word",
  "text": "传智播客Java就业率超过95%,习大大都点赞,奥力给！"
}
```

> 注意当前文件的编码必须是 UTF-8 格式，严禁使用Windows记事本编辑

### 6.mapping属性  

![image-20220410093934984](https://gitee.com/ingachin/mdimage/raw/master/image-20220410093934984.png)

```DCL
PUT /maisann
{
  "mappings": {
    "properties": {
      "info":{
        "type": "text"
        , "analyzer": "ik_smart"
      },
      "email":{
        "type": "keyword",
        "index": false
      },
      "name":{
        "type": "object",
        "properties": {
          "firstName":{
            "type": "keyword"
          },
          "lastName":{
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

![image-20220410095019640](https://gitee.com/ingachin/mdimage/raw/master/image-20220410095019640.png)
