# nginx

## 1.nginx安装

```bash
$ sudo yum -y install gcc gcc-c++ # nginx 编译时依赖 gcc 环境
$ sudo yum -y install pcre pcre-devel # 让 nginx 支持重写功能
# zlib 库提供了很多压缩和解压缩的方式，nginx 使用 zlib 对 http 包内容进行 gzip 压缩
$ sudo yum -y install zlib zlib-devel 
# 安全套接字层密码库，用于通信加密
$ sudo yum -y install openssl openssl-devel
```

### nginx 源码包安装

将准备好的  `nginx-1.11.5.tar.gz` 包，拷贝至 `/usr/local/nginx` 目录下（一般习惯在此目录下进行安装）进行解压缩。

源码包下载地址：[nginx.org/en/download…](https://link.juejin.cn?target=https%3A%2F%2Fnginx.org%2Fen%2Fdownload.html)

```bash
$ sudo tar -zxvf  nginx-1.11.5.tar.gz # 解压缩
```

在完成解压缩后，进入 `nginx-1.11.5` 目录进行源码编译安装。

```bash
$  cd nginx-1.11.5
$ ./configure --prefix=/usr/local/nginx # 检查平台安装环境
  # --prefix=/usr/local/nginx  是 nginx 编译安装的目录（推荐），安装完后会在此目录下生成相关文件
```

进行源码编译并安装 nginx

```bash
$ make # 编译
$ make install # 安装
```

## 2.nginx常用命令

```bash
nginx -v

nginx -t #测试配置文件

nginx -s stop #停止nginx服务

nginx -s reload #重新加载nginx配置文件

# linux/mac启动
$ service nginx

# 手动指定配置
$ nginx -c /usr/local/nginx/conf/nginx.conf

# -p指定nginx运行目录(日志存储位置)
$ nginx -c /path/nginx.conf -p /path/

# 重启
$ nginx -s reload

# 关闭
$ nginx -s stop

# 查看端口
$ netstat -an | grep 端口  # linux/mac系统
> netstat -an | findstr 端口  # windows系统

# 测试web服务
$ curl -i 主机:端口
# 或
$ telnet 主机 端口

# 查看进程
$ ps -ef | grep nginx

# 查看错误日志
$ tail -30 /var/log/nginx/error.log
```

