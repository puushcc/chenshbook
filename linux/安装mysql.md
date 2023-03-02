# 安装mysql

## Ubuntu下安装mysql

> 下载二进制包

官网：https://www.mysql.com/

选择版本：mysql-8.0.28-linux-glibc2.17-x86_64-minimal

> 基础准备

```bash
1.解压 
tar -xf mysql-8.0.28-linux-glibc2.17-x86_64-minimal.tar
2.选择版本，再次解压
tar -xf mysql-8.0.28-linux-glibc2.17-x86_64-minimal.tar.xz
3.我这里为了方便把名字改了
mv  mysql-8.0.28-linux-glibc2.17-x86_64-minimal mysql
4. 创建用户组
groupdel mysql  #删除组
groupadd mysql  #添加组
5.创建用户
userdel mysql  #删除用户
useradd -r -g  mysql -s /bin/false mysql  #添加用户 -g mysql 是指定用户组 -s /bin/false 是指定用户所使用的shell 最后mysql是用户名称
6. 指定mysql文件夹为mysql用户所有
chown mysql ./mysql -R
7.在data目录下创建mysql相关文件夹文件夹
mkdir /data/mysql
mkdir /data/mysql/logs
mkdir /data/mysql/conf
mkdir /data/mysql/data
8.安装相关依赖
apt-get install libaio1 libaio-dev
sudo apt-get install libncurses5-dev
```

> 创建my.cnf 文件

```bash
[mysql]
#设置mysql客户端默认字符集
default-character-set=utf8mb4
[mysqld]
skip-name-resolve
user=mysql
ngram_token_size=2
server-id=1
default_password_lifetime=0
port=3306
#设置安装目录
basedir=/opt/mysql
#数据存放目录
datadir=/data/mysql/data
log-error=/data/mysql/logs/err.log
#允许最大连接数
max_connections=1000
#服务端默认使用的字符集
character-set-server=utf8mb4
#创捷新表时默认的储存引擎
default-storage-engine=INNODB
#忘记密码时使用
#skip-grant-tables
#不区分大小写
lower_case_table_names=1
#认证方式
default_authentication_plugin=mysql_native_password
max_allowed_packet=500M
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
wait_timeout=28800
interactive_timeout=28800
max_connect_errors=100
max_user_connections=0
#日志文件大小
max_binlog_size=100M
```

> 安装启动

```bash
指定配置文件安装
./mysqld --defaults-file=/data/mysql/conf/my.cnf --initialize 
指定配置文件启动
./mysqld --defaults-file=/data/mysql/conf/my.cnf & 
```

> 配置systemctl

```bash
vim /usr/lib/systemd/system/mysql.service
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql

# Have mysqld write its state to the systemd notify socket
Type=notify

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Start main service
ExecStart=/opt/mysql/bin/mysqld --defaults-file=/data/mysql/conf/my.cnf $MYSQLD_OPTS 

# Use this to switch malloc implementation
EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 10000

Restart=on-failure

RestartPreventExitStatus=1

# Set environment variable MYSQLD_PARENT_PID. This is required for restart.
Environment=MYSQLD_PARENT_PID=1

PrivateTmp=false
```

```
systemctl start mysql  #启动
systemctl stop mysql  #停止
systemctl restart mysql #重启
systemctl enable mysql  #开机自启
```

> 相关错误

```bash
1.查看错误日志如果出现 ：
mysql.sock.lock
解决办法删除/tmp目录下的mysql相关文件

2.mysql: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory 
解决办法：安装依赖 sudo apt-get install libncurses5-dev

3.mysqld: File ‘./binlog.xxxxxxxx’ not found (OS errno 2 - No such file or directory)
解决办法删除“mysql/data”目录下的binlog.index文件

4.mysqld: [Warning] World-writable config file ‘xxxxx/my.cnf’ is ignored.
解决办法把配置文件权限设置成 chmod 644 xxxxx/my.cnf
```

## windows下安装mysql