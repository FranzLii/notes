# MongoDB

## 1.MongoDB安装部署

### 1.WINDOWS

[mongoDB下载链接](https://www.mongodb.com/try/download/community)

启动方式

1. 命令行启动

```bash
./mongod.exe --dbpath=..\data\db
```
首先创建\data\db文件夹

其中`--dbpath=..\data\db`为数据存放路径

2. 配置文件方式启动服务

创建\conf\mongod.conf文件

```bash
mongod -f ../config/mongod.conf
或
mongod --config ../config/mongod.conf
```

更多配置文件参数

```yaml
systemLog:
  destination: file
 # The path of the log file to which mongod or mongos should send all diagnostic logging information
  path: "D:/02_Server/DBServer/mongodb-win32-x86_64-2008plus-ssl-4.0.1/log/mongod.log"
  logAppend: true
storage:
  journal:
    enabled: true
 #The directory where the mongod instance stores its data.Default Value is "/data/db".
  dbPath: "D:/02_Server/DBServer/mongodb-win32-x86_64-2008plus-ssl-4.0.1/data"
net:
#bindIp: 127.0.0.1
  port: 27017
setParameter:
  enableLocalhostAuthBypass: false

```

### 2.Linux下的部署





