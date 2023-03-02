# 安装Git

## 1、查看当前git版本：git --version

[查看最新版git（官网）](https://www.kernel.org/pub/software/scm/git/)

[查看最新版git（GitHub）](https://github.com/git/git/releases)

## 2、移除旧版本

```bash
yum remove git
```

## 3、官网下载

```bash
wget https://Github.com/Git/Git/archive/v2.11.0.tar.gz
tar -zxvf git-2.11.0.tar.gz
```

>  或者下载rz到服务器,安装yum install lrzsz后rz即可

## 4、编译安装

> 避免报错，安装相关依赖

```bash
yum install libcurl-devel zlib-devel zlib
yum install  autoconf automake libtool
```

> 编译安装

```bash
cd git-2.11.0
make configure
./configure --prefix=/usr/local/git --with-iconv --with-curl --with-expat=/usr/local/lib
#或者./configure --prefix=/usr/local/git --with-iconv =/usr/local/lib
make && make install
```

> 设置环境变量

```bash
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
source /etc/bashrc
git --version
```

## 5、Git配置