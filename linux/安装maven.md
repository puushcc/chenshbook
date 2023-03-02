# 安装Maven

## 1、下载

maven 官网地址：https://maven.apache.org/download.cgi

## 2、解压

```bash
mkdir /usr/local/maven
tar -zxvf apache-maven-3.8.5-bin.tar.gz -C /usr/local/maven
```

## 3、maven配置

```bash
cd /usr/local/maven/apache-maven-3.8.5/
cd conf/
mkdir -p /m2/repository
```

修改资源库位置，添加阿里云国内镜像

```absh
vim settings.xml
```

```bash
<localRepository>/m2/repository</localRepository>

   <mirror>  
   	  <id>alimaven</id>  
   	  <name>aliyun maven</name>  
	  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
   	  <mirrorOf>central</mirrorOf>          
    </mirror> 
```

## 4、添加环境变量

```bash
vim /etc/profile
```

```bash

MAVEN_HOME=/usr/local/maven/apache-maven-3.8.5
PATH=$MAVEN_HOME/bin:$PATH
export MAVEN_HOME PATH
```

```bash
source /etc/profile
```

测试

```bash
mvn -version

```

