# 安装redis

## 1、二进制编译安装redis
从英文官方网站https://redis.io/download或者中文官方网站http://redis.cn/下载Redis安装包

基本环境安装

```bash
yum install gcc-c++ -y  
```
解压安装包

```shell
tar -zxvf redis-6.0.6.tar.gz
```
编译安装

```shell
make
make install PREFIX=/usr/local/redis 
#提示"Hint: It's a good idea to run 'make test' ;)"，代表安装成功
```

查看默认安装目录：

- redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何
- redis-check-aof：修复有问题的AOF文件，rdb和aof后面讲
- redis-check-dump：修复有问题的dump.rdb文件
- redis-sentinel：Redis集群使用
- redis-server：Redis服务器启动命令
- redis-cli：客户端，操作入口

启动服务

```shell
#进入/usr/local/redis，该目录存在一个bin文件存放这可执行文件
cd /usr/local/redis  
#在/usr/local/redis中新建一个conf目录用于存放redis的配置文件
mkdir conf   
#将/usr/local/redis-6.0.6中的redis.conf复制到/usr/local/redis/conf中
cp /usr/local/redis-6.0.6/redis.conf /usr/local/redis/conf/ 
#编辑redis.conf文件，把 daemonize no改为 
vim redis.conf  
daemonize yes
#输入命令进行后台启动
/usr/local/redis/bin/redis-server /usr/local/redis/conf/redis.conf 
```
设置开机启动

```bash
vim /usr/lib/systemd/system/redis.service  
```

> #添加以下内容：

```shell
[Unit] 
Description=redis-server 
After=network.target 
[Service] 
Type=forking 
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/conf/redis.conf 
PrivateTmp=true 
[Install] 
WantedBy=multi-user.target
```
```bash
systemctl daemon-reload 
systemctl start redis.service 
systemctl enable redis.service
```

创建redis命令软连接

```shell
ln -s /usr/local/redis/bin/redis-cli /usr/bin/redis
#这样在/usr/bin/目录下就创建了一个redis文件
ln -s /data/redis-4.0.12/src/redis-cli /usr/bin/redis-cli
```
连接redis

```shell
redis -h localhost -a 密码
ping
```
## 2、Linux安装redis
