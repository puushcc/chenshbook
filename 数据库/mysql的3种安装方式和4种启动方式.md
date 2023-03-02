# MYSQL的3中安装方式和4种启动方式

## mysql 3中安装方式

|      | rpm安装                                                      | 源码编译安装                                                 | 二进制安装                                                   |
| :--: | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 优点 | 安装简单，适合对数据库要求不大高的场合，如并发不大。适合初学者使用 | 可以定制化编译灵活，性能最好台服务器上可以安装多个MySOL数据库 | 安装简单，方便、灵活，可以安装到任何目录下，并且一台服务器可以安装多个MySOL数据库，是很多专业DBA的选择 |
| 缺点 | 需要单独下载客户端和服务端，安装路径不能修改，一台服务器只能安装一个 | 安装过程比较复杂编译时间长                                   | 已经经过编译，性能不如源码编译，不能灵活定制编译参数         |

## mysql 4种启动方式

1. mysqld 启动

  ```bash
  #进入mysqld文件所在目录(/../libexec/mysqld)  
  ./mysqld --defaults-file=./my.cnf --user=mysql
  ```

2. mysqld_safe 启动

  ```bash
  #进入mysqld_safe所在目录(../bin/mysqld_safe)
  ./bin/mysqld_safe --defaults-file=./my.cnf --user=mysql
  #如果mysqld进程异常中断的话mysqld_safe会重启mysqld进程
  ```

3. mysql.servre

  ```bash
  #进入mysql.server所在目录（../share/mysql/mysql.server）
  ./mysql.server start
  #把mysql.server文件拷贝到系统启动目录下可以加入系统程序启动
  cp ./mysql.server /etc/rc.d/int.d/mysql
  chkconfig --add mysql
  #直接用server mysql start
  ```

4.	mysqld_multi

    管理多个数据库