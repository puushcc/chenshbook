# 安装Jenkins

## 1、Jenkins版本和jdk对应关系

```shell
2.361.1 (September 2022) and newer
Java 11 or Java 17
2.346.1 (June 2022) and newer
Java 8, Java 11, or Java 17
2.164.1 (March 2019) and newer
Java 8 or Java 11
2.60.1 (June 2017) and newer
Java 8
1.625.1 (October 2015) and newer
Java 7
```

## 2、**在线安装（联网）**

```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
#注意： 这种安装方式默认安装最新版本，需要支持JDK11版本以上
```

## 3、**离线安装**

下载地址：[https://jenkins.io/zh/download/](https://links.jianshu.com/go?to=https://jenkins.io/zh/download/)或者

https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/

> Ubuntu使用dpkg安装

[Ubuntu](https://so.csdn.net/so/search?q=Ubuntu&spm=1001.2101.3001.7020)的软件包格式是deb，如果要安装rpm的包，则要先用alien把rpm转换成deb

```bash
sudo apt-get install alien 

#alien默认没有安装，所以首先要安装它

sudo alien jenkins-2.346.1-1.1.noarch.rpm 

#将rpm转换位deb，完成后会生成一个同名的xxxx.deb

sudo dpkg -i jenkins_2.346.1-2.1_all.deb #安装
```

> 配置

**1）检查/usr/bin/java下是否链接正确**

```bash
#软连接的地址必须与配置Jenkins的Java地址一致，否则会导致启动失败
ln -s /usr/local/java/jdk1.8.0_131/bin/java /usr/bin/java
```

**2）修改配置文件/etc/init.d/jenkins**

vim /etc/init.d/jenkins添加java路径

```bash
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-11./bin/java
/usr/lib/jvm/jre-11.8/bin/java
/usr/lib/jvm/java-11-openjdk-amd64usr/bin/java
#添加java路径
/usr/local/java/jdk1.8.131/bin/java
```

**3）修改文件/etc/sysconfig/jenkins**

vim /etc/sysconfig/jenkins

JENKINS_USER="root"，如果不是root，而是jenkins或者其它，要改成root

```bash
#Unix user account that runs the Jenkins daemon
#Becareful when you change this, as you need to update
#permissions of $JENKINS HOME and /var/log/jenkins.
# and if you have already run Jenkins, potentially other
# directories such as /var/cache/jenkins
JENKINS USER="root'
```

默认8080端口被占用，则需要修改端口号

```bash
#Port Jenkins is listening on.
#Set to -1 to disable
JENKINS_PORT="8080"
```

**4）修改文件/usr/lib/systemd/system/jenkins.service**

vim /usr/lib/systemd/system/jenkins.service

```bash
#如果不是root，而是jenkins或者其它，要改成root
User=root
Group=root
#JENKINS_HOME配置JAVA路径
Environment="JENKINS_HOME=/usr/local/java/jdk1.8.0_131"
#JENKINS_PORT配置端口号与/etc/sysconfig/jenkins一致
Environment="JENKINS_PORT=8089"
```

> 启动

```bash
systemctl daemon-reload
systemctl start jenkins
systemctl stop jenkins.service
systemctl status jenkins.service
systemctl enable jenkins.service
```

> 卸载Jenkins

```bash
dpkg -l |grep jenkins
dpkg -P jenkins
dpkg -l |grep jenkins
```

> Jenkins 访问地址

IP+端口：http://10.29.4.223:8089/