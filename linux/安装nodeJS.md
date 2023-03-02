# 安装NodeJS

## 1、下载

[官网](https://nodejs.org/en/download/)

[中文官网](https://nodejs.org/zh-cn/download/)

>  node的安装方式有两种

- 通过Source Code源码安装（手动安装，麻烦、不利于维护）
- 通过编译后的二进制文件安装（简单，建立软连接即可）

## 2、解压、改目录名字、查看版本

```bash
tar -xvf node-v14.15.1-linux-x64.tar.xz
mv /usr/local/node-v14.15.1-linux-x64 /usr/local/node
./node -v
./npm -v
```

## 3、配环境变量

```bash
vim /etc/profile
export PATH=$PATH:/usr/local/node/bin
source /etc/profile
```

## 4、配软连接

```bash
ln -s /usr/local/node/bin/node /usr/local/bin/
ln -s /usr/local/node/bin/npm /usr/local/bin/
```

## 5、npm配置淘宝镜像源

```bash
npm config set registry https://registry.npm.taobao.org
```

```bash
npm get registry
https://registry.npm.taobao.org
```
