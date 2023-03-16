# 安装Kafka

前提条件：已经安装JDK和zookeeper

## 单机

> [官网](http://kafka.apache.org/downloads)下载安装包

kafka_2.12-2.6.0.tgz

> 解压到本地指定目录

```bash
tar -zxvf kafka_2.12-2.6.0.tgz -C /usr/local/myapp/kafka
```

>  环境变量配置

配置环境变量的目的之一是在任何目录下都可以执行kafka bin目录下的命令

```bash
export KAFKA_HOME=/usr/local/myapp/kafka/kafka_2.12-2.6.0
export PATH=P A T H : PATH:PATH:KAFKA_HOME/bin
```

> 修改kafka配置文件

配置文件在config/server.properties中，主要修改如下参数

```bash
#broker的全局唯一id，一般从0开始编号，不能重复
broker.id=0
#kafka对外提供服务监听地址，设置运行kafka的机器IP地址，客户端用此地址连接到kafka
listeners=PLAINTEXT://192.168.174.129:9092
#日志目录，多个目可以用逗号隔开(先在系统中创建好目录)
log.dirs=/usr/local/myapp/kafka/kafka_2.12-2.6.0/log/kafka
#默认分区数配置，一般在集群配置中要设多个分区提高性能
num.partitions=1
#创建topic时的默认副本数，一般在集群配置中要设多个副本，提高可用性
default.replication.factor=1
#zk服务器地址配置，一般单机模式zk和kafka运行在同一台机器，配置kafka所在机器ip即可
zookeeper.connect=localhost:2181
#启用删除topic的功能，默认为true
delete.topic.enable=true
#用来处理磁盘I/O的线程数，默认为8
num.io.threads=8
#用来处理网络请求的线程数，默认为3
num.network.threads=3
#kafka log日志保留的时间，默认7天(168h)
log.retention.hours=168
```

> 启动kafka

启动kafka前必须先运行zookeeper，因为kafka会连接到zookeeper

```bash
[root@vm1 ~]# zkServer.sh start
[root@vm1 ~]# rm -rf /usr/local/myapp/kafka/kafka_2.12-2.6.0/log/kafka/*
[root@vm1 ~]# kafka-server-start.sh /usr/local/myapp/kafka/kafka_2.12-2.6.0/config/server.properties &
```

> kafka启动和停止脚本

把kafka和zookeeper的命令封装成shell脚本

vim kafka_start.sh

```bash
#!/bin/bash
#启动zookeeper
zkServer.sh start & sleep 10 #等10秒后执行
#启动kafka
kafka-server-start.sh /usr/local/myapp/kafka/kafka_2.12-2.6.0/config/server.properties &
```

vim kafka_stop.sh

```bash
#!/bin/bash
#停止kafka
kafka-server-stop.sh /usr/local/myapp/kafka/kafka_2.12-2.6.0/config/server.properties & sleep 10 #等10秒执行
#停止zookeeper
zkServer.sh stop
```

## 集群

按照单机在第一台机器vm1上解压、配置kafka

> 配置config/server.properties

```bash
#集群中每个broker的id唯一，一般从0开始

broker.id=0

#kafka对外提供服务监听地址，设置运行kafka的机器IP地址，客户端用此地址连接到kafka

listeners=PLAINTEXT://192.168.174.129:9092

#日志目录，多个目录可以用逗号隔开，先创建好目录

log.dirs=/usr/local/myapp/kafka/kafka_2.12-2.6.0/log/kafka

#分区数配置，集群模式下一般设多个分区提高性能

num.partitions=3

#创建topic时的默认副本数，集群模式下一般配置多个副本，提高可用性

default.replication.factor=3

#zk服务器地址配置，集群模式下配置zk的地址列表，逗号分隔

zookeeper.connect=192.168.174.129:2181,192.168.174.130:2181,192.168.174.131:2181
```

> 复制到其它机器

```bash
scp -r /usr/local/myapp/kafka/ root@vm2:/usr/local/myapp/
scp -r /usr/local/myapp/kafka/ root@vm3:/usr/local/myapp/
```

分别修改其它机器的config/server.properties配置

```bash
#集群中每个broker的id唯一，一般从0开始

broker.id=1

#kafka对外提供服务的监听地址设置为本机ip

listeners=PLAINTEXT://192.168.174.131:9092
```

```bash
#集群中每个broker的id唯一，一般从0开始

broker.id=2

#kafka对外提供服务的监听地址设置为本机ip

listeners=PLAINTEXT://192.168.174.130:9092
```

> 启动kafka集群
>
> 先启动zk，再启动kafka

```bash
[root@vm1 ~]# zkServer.sh start

[root@vm2 ~]# zkServer.sh start

[root@vm3 ~]# zkServer.sh start

[root@vm1 ~]# kafka-server-start.sh /usr/local/myapp/kafka/kafka_2.12-2.6.0/config/server.properties &

[root@vm1 ~]# kafka-server-start.sh /usr/local/myapp/kafka/kafka_2.12-2.6.0/config/server.properties &

[root@vm1 ~]# kafka-server-start.sh /usr/local/myapp/kafka/kafka_2.12-2.6.0/config/server.properties &
```

> 验证kafka集群

随机找一台kafka的机器创建topic，在另外的kafka服务器查看集群topic，如果有则集群配置正常

```bash
[root@vm1 ~]# kafka-topics.sh --create --zookeeper 192.168.174.129:2181 --replication-factor 2 -partitions 2 --topic kafkatest
```

```bash
[root@vm1 ~]# kafka-topics.sh --describe --zookeeper 192.168.174.129:2181
Topic: kafkatest PartitionCount: 2 ReplicationFactor: 2 Configs:
Topic: kafkatest Partition: 0 Leader: 0 Replicas: 0,1 Isr: 0,1
Topic: kafkatest Partition: 1 Leader: 1 Replicas: 1,2 Isr: 1,2


[root@vm2 ~]# kafka-topics.sh --describe --zookeeper 192.168.174.131:2181
Topic: kafkatest PartitionCount: 2 ReplicationFactor: 2 Configs:
Topic: kafkatest Partition: 0 Leader: 0 Replicas: 0,1 Isr: 0,1
Topic: kafkatest Partition: 1 Leader: 1 Replicas: 1,2 Isr: 1,2


[root@vm3 ~]# kafka-topics.sh --describe --zookeeper 192.168.174.130:2181
Topic: kafkatest PartitionCount: 2 ReplicationFactor: 2 Configs:
Topic: kafkatest Partition: 0 Leader: 0 Replicas: 0,1 Isr: 0,1
Topic: kafkatest Partition: 1 Leader: 1 Replicas: 1,2 Isr: 1,2
```

也可以在每一台机器上使用list命令查看kafka集群topic

```bash
[root@vm1 ~]# kafka-topics.sh --list --bootstrap-server 192.168.174.129:9092
kafkatest
kafkatest2
kafkatest3

[root@vm3 ~]# kafka-topics.sh --list --bootstrap-server 192.168.174.130:9092
kafkatest
kafkatest2
kafkatest3

[root@vm3 ~]# kafka-topics.sh --list --bootstrap-server 192.168.174.130:9092
kafkatest
kafkatest2
kafkatest3
```

