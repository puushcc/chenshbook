# 安装JDK

## 1、卸载旧版本JDK

使用`rpm -qa | grep java`命令查看系统中是否存在有Java。

使用`rpm -e --nodeps`相关应用名称来进行卸载。(相关应用名称就是上一个命令中显示出来的名称复制到这里卸载即可)。

## 2、yum安装（不推荐）
查看JDK软件包列表

```bash
yum search java | grep -i --color jdk
```

选择版本安装

```bash
yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel
```

或者如下命令，安装jdk1.8.0的所有文件

```bash
yum install -y java-1.8.0-openjdk*
```

查看JDK是否安装成功

```bash
java -version
```


## 3、二进制安装
>下载并上传服务器并解压

```bash
tar -zxvf jdk1.8.0_212.tar.gz
mv jdk1.8.0_212 /usr/local
```


> 配置系统环境变量

```bash
vim /etc/profile
#配置项内容如下所示。
JAVA_HOME=/usr/local/jdk1.8.0_212
CLASS_PATH=.:$JAVA_HOME/lib
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASS_PATH PATH
```

>使系统环境变量生效

```bash
source /etc/profile
```


>查看版本信息

```bash
Java -version
```
