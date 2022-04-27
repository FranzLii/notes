# mysql配置主从复制

## 1.主库配置    

```bash
docker run -d --restart=always --name mysql2 \
-v /itwxe/dockerData/mysql2/data:/var/lib/mysql \
-v /itwxe/dockerData/mysql2/conf:/etc/mysql \
-v /itwxe/dockerData/mysql2/log:/var/log/mysql \
-p 3301:3306 \
-e TZ=Asia/Shanghai \
-e MYSQL_ROOT_PASSWORD=root \
mysql:5.7.37 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_general_ci 
```

```sql
show variables like '%log_bin%';
```

### 1.修改配置文件

```
[mysqld]
skip-name-resolve
character_set_server=utf8
datadir=/var/lib/mysql
log-bin=mysql-bin
server-id=100
```

### 2.创建从库访问用户

```sql
GRANT REPLICATION SLAVE ON *.* to 'xiaoming'@'%'identified by 'ROOT@123456';
```

  ![image-20220424150806970](https://gitee.com/ingachin/mdimage/raw/master/image-20220424150806970.png)

![image-20220424160531131](https://gitee.com/ingachin/mdimage/raw/master/image-20220424160531131.png)

## 2.从库配置  

### 1.修改配置文件

```
[mysqld]
skip-name-resolve
character_set_server=utf8
datadir=/var/lib/mysql
server-id=101
```

![image-20220424151510041](https://gitee.com/ingachin/mdimage/raw/master/image-20220424151510041.png)

```sql
CHANGE MASTER TO MASTER_HOST='172.19.0.3',MASTER_PORT=3306,MASTER_USER='xiaoming',MASTER_PASSWORD='ROOT@123456', MASTER_LOG_FILE='mysql-bin.000005',MASTER_LOG_POS=441;
```

## 3.入门案例  

### 1.导入依赖

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>4.1.1</version>
</dependency>
```

### 2.配置读写分离规则

```yaml

spring:
  shardingsphere:
    datasource:
      names:
        master,slave
      # 主数据源
      master:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.117.128:3300/rw?characterEncoding=utf-8
        username: root
        password: root
      # 从数据源
      slave:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.117.128:3301/rw?characterEncoding=utf-8
        username: root
        password: root
    masterslave:
      # 读写分离配置
      load-balance-algorithm-type: round_robin #轮询
      # 最终的数据源名称
      name: dataSource
      # 主库数据源名称
      master-data-source-name: master
      # 从库数据源名称列表，多个逗号分隔
      slave-data-source-names: slave
    props:
      sql:
        show: true #开启SQL显示，默认false

```

### 3.在配置文件中配置允许bean定义覆盖配置项

```yaml
spring: 
  main:
    allow-bean-definition-overriding: true
```

