# 安装nginx

## 1、卸载Nginx
1）执行命令 `rm -rf` *删除nignx安装的相关文件（这个命令是看你nginx位置而定的）

2）如果设置了Nginx开机自启动的话，可能还需要下面两步

```bash
systemctl stop nginx & systemctl disable nginx
rm -rf /usr/lib/systemd/nginx.service
```


3）可以再用yum指令清理

```bash
yum clean nginx
```

## 2、Nginx安装

官网http://nginx.org/下载对应的nginx包，推荐使用稳定版本

>安装gcc环境

```bash
yum install gcc-c++
```


>安装PCRE库，用于解析正则表达式

```bash
yum install -y pcre pcre-devel
```

>zlib压缩和解压缩依赖

```bash
yum install -y zlib zlib-devel
```

> SSL 安全的加密的套接字协议层，用于HTTP安全传输，也就是https

```bash
yum install -y openssl openssl-devel
```

> 解压

```bash
tar -zxvf nginx-1.16.1.tar.gz
```

> 创建nginx临时目录

```bash
mkdir /var/temp/nginx -p
```

> 创建makefile文件

```bash
./configure \   
--prefix=/usr/local/nginx \    
--pid-path=/var/run/nginx/nginx.pid \    
--lock-path=/var/lock/nginx.lock \    
--error-log-path=/var/log/nginx/error.log \    
--http-log-path=/var/log/nginx/access.log \    
--with-http_gzip_static_module \    
--http-client-body-temp-path=/var/temp/nginx/client \    
--http-proxy-temp-path=/var/temp/nginx/proxy \    
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \    
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \    
--http-scgi-temp-path=/var/temp/nginx/scgi
```

> make编译&安装

```bash
make
make install
```

> 创建软连接

```bash
ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/nginx
```

## 3、设置nginx开机启动

```bash
vim /usr/lib/systemd/system/nginx.service
```

```bash
[Unit]
Description=nginx 
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/serve/nginx/sbin/nginx
ExecReload=/serve/nginx/sbin/nginx -s reload
ExecStop=/serve/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```



