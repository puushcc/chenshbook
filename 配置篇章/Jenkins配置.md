# Jenkins配置

## 1、用户管理

> 添加用户

dashboard  👉  系统管理  👉  管理用户 👉  新建用户

> 登录问题

未设置密码时在服务器上获取密码

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

## 2、配置JDK

> 安装JDK

[Linux下安装JAVA](https://puushcc.github.io/chenshbook/#/linux/java)

> Jenkins配置JDK

Dashboard👉  系统管理👉  全局工具配置👉  JDK👉  新增JDK

```bash
别名：JDK1.8
JAVA_HOME：/usr/local/jdk1.8.0_131
```

## 3、配置Git

[Linux下安装Git](https://puushcc.github.io/chenshbook/#/linux/安装Git)

Dashboard👉  系统管理👉  全局工具配置👉  Git👉  ADD Git

```bash
Name：gitName
Path to Git executable: /usr/bin/git
```

## 4、配置nodeJS

[Linux下安装NodeJs](https://puushcc.github.io/chenshbook/#/linux/安装nodeJs)

Dashboard👉  系统管理👉  全局工具配置👉  NodeJS👉  NodeJS安装

```bash
别名:  nodejs
安装目录: /usr/local/node
```

## 5、配置maven

[Linux下安装Maven](https://puushcc.github.io/chenshbook/#/linux/安装maven)

Dashboard👉  系统管理👉  全局工具配置👉  Maven配置👉  默认 settings 提供/默认全局 settings 提供

```bash
文件路径：/usr/local/maven-3.8.7
```

## 6、配置Publish over SSH

Dashboard👉  系统管理👉  插件管理 👉 安装Publish Over SSH

Dashboard👉  系统管理👉  Configure System 👉 Publish Over SSH

```bash
Name: 自定义名称
Hostname:  IP地址
Username:  用户名
Remote Directory: 传输路径
Port: 22
Timeout (ms):  30000
```

