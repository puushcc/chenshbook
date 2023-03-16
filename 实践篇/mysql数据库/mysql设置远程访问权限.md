# MySQL开启远程访问权限

默认情况下，mysql只允许本地登录，即只能在安装MySQL环境所在的主机下访问。
但是在日常开发和使用中，我们经常需要访问远端服务器的数据库，此时就需要开启服务器端MySQL的远程连接权限。

## 1、连接MySQL环境

通过mysql命令连接MySQL

```bash
mysql -uroot -p
```

## 2、查看MySQL当前远程访问权限配置

```sql
use mysql
select  User,authentication_string,Host from user;
#通过命令可以看到当前只有一个用户root,并且Host为localhost,即只能本地访问权限
```

## 3、开启远程访问权限

> 方式一：改表法

顾名思义,该方法就是直接修改更改"mysql"数据库里的"user"表里的"host"项，从"localhost"改为"%"

```bash
update user set host='%' where user='root';
```

> 方式二：授权法

```sql
--赋予任何主机访问权限：
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
--允许指定主机(IP地址)访问权限：
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'root' WITH GRANT OPTION;
```

通过GRANT命令赋权后,需要通过`FLUSH PRIVILEGES`刷新权限表使修改生效：

```bash
FLUSH PRIVILEGES;
```

## 4、再次查看MySQL远程访问权限配置

```sql
select  User,authentication_string,Host from user;
#此时,root已经多了一条记录,且Host记录值为%,代表已经开启了root的远程访问权限,我们后续就可以通过root用户远程访问该MySQL了
```

## 注意:

**出于安全性考虑，尤其是正式环境下**

> 1.**不推荐**直接给root开启远程访问权限。

本案例仅以root用户作为例子做开启远程访问权限的配置,此为演示环境!

> 2.建议做**权限细分**和**限制**

正式环境中，推荐通过创建Mysql用户并给对应的用户赋权的形式来开放远程服务权限，并指定IP地址，赋权时根据用户需求，在GRANT命令中只开放slect、update等权限，做到权限粒度最小化。