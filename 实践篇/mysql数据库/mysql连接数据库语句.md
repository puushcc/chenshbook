> 连接mysql数据库后直接使用test数据库

```mysq
mysql -u root -D test -p123123
```

> -h指定mysql主机

```bash
mysql -u root -p -h 192.168.1.103 -P 3306
```

> 执行-e选项后面跟随的sql语句，并且返回语句执行的结果

```mysql
mysql -u root -p123123 -e 'use mysql; select user,host,password from user;'
```

