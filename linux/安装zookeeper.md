# Zookeeper安装

部署要求：部署系统上已经安装了JDK

## 单机

> 下载

[官网](https://downloads.apache.org/zookeeper/)下载zk安装包，比如[apache](https://so.csdn.net/so/search?q=apache&spm=1001.2101.3001.7020)-zookeeper-3.6.1-bin.tar.gz

> 解压到本地指定目录

```bash
[root@vm1 ~]# tar -zxvf apache-zookeeper-3.6.1-bin.tar.gz -C /usr/local/myapp/zookeeper
[root@vm1 ~]# mv /usr/local/myapp/zookeeper/apache-zookeeper-3.6.1-bin/ /usr/local/myapp/zookeeper/zookeeper-3.6.1
```

> 环境变量配置

```bash
[root@vm1 ~]# vim /etc/profile

export ZOOKEEPER_HOME=/usr/local/myapp/zookeeper/zookeeper-3.6.1
export PATH=P A T H : PATH:PATH:ZOOKEEPER_HOME/bin
```

执行 source /etc/profile使配置生效

> 修改zk配置文件

```bash
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg
#数据目录
dataDir=/usr/local/myapp/zookeeper/zookeeper-3.6.1/dataDir
#单独指定事务日志目录
dataDir=/usr/local/myapp/zookeeper/zookeeper-3.6.1/dataLogDir
#zk对外服务端口
clientPort=2181
#基本时间单位，和时间相关的配置都是该值的倍数；即zk服务器单次心跳间隔时间，单位毫秒
tickTime=2000
#投票选举leader时zk服务器连上leader的时间限制，initLimit*tickTime为总的超时时间
initLimit=10
#zk正常工作时leader和follower之间最多心跳数限制，syncLimit*tickTime为总超时容忍时间，超过此时间，follower将从zk集群中被剔除
syncLimit=5
```

注意：上述配置中的数据和日志文件路径先要在系统中创建

```bash
[root@vm1 conf]# mkdir -p /usr/local/myapp/zookeeper/zookeeper-3.6.1/dataDir
[root@vm1 conf]# mkdir -p /usr/local/myapp/zookeeper/zookeeper-3.6.1/dataLogDir
```

> 启动zookeeper

```bash
zkServer.sh start
```

## 集群

机器规划：3台服务器，1台leader，2台follower，每台机器都在/etc/hosts中配置好域名ip映射

| 机器主机名 | 机器IP          | 机器角色        |
| ---------- | --------------- | --------------- |
| vm1        | 192.168.174.129 | leader(master)  |
| vm2        | 192.168.174.131 | follower(slave) |
| vm3        | 192.168.174.130 | follower(slave) |

下列所有的安装都可以先在第一台机器进行，然后再拷贝到集群其它机器中修改。

Zookeeper集群原则上需要2n+1个实例才能保证集群有效性，所以集群规模至少是3台。vm1上的安装参照单机安装

>  在vm2、vm3上建好目录，并从vm1复制

```bash
[root@vm2 ~]# mkdir -p /usr/local/myapp/zookeeper
[root@vm3 ~]# mkdir -p /usr/local/myapp/zookeeper
[root@vm1 ~]# scp -r /usr/local/myapp/zookeeper/zookeeper-3.6.1/ root@vm2:/usr/local/myapp/zookeeper/
[root@vm1 ~]# scp -r /usr/local/myapp/zookeeper/zookeeper-3.6.1/ root@vm3:/usr/local/myapp/zookeeper/
```

> 在单机安装的基础上增加下列配置：

```bash
#每台机器的/conf/zoo.cfg文件中增加节点信息
#server.A=B:C:D，A表示服务器编号；B是IP；C是该服务器与leader通信端口；D是leader挂掉后重新选举所用通信端口
server.1=192.168.174.129:2888:3888
server.2=192.168.174.131:2888:3888
server.3=192.168.174.130:2888:3888
```

```bash
#在每台ZK节点的dataDir目录下新建myid文件
#myid配置文件内容是当前服务器的编号，即server.后面的数字，分别在每台机器执行一次echo赋值
echo 1 > /usr/local/myapp/zookeeper/zookeeper-3.6.1/dataDir/myid #第1台上执行
echo 2 > /usr/local/myapp/zookeeper/zookeeper-3.6.1/dataDir/myid #第2台上执行
echo 3 > /usr/local/myapp/zookeeper/zookeeper-3.6.1/dataDir/myid #第3台上执行
```

```bash
#分别启动每个服务器上的zk服务，注意先要关闭各个机器的防火墙
systemctl stop firewalld
zkServer.sh start
```

```bash
#分别查看每个节点的状态
[root@vm1 ~]zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/myapp/zookeeper/zookeeper-3.6.1/bin/…/conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader

[root@vm3 ~]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/myapp/zookeeper/zookeeper-3.6.1/bin/…/conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

执行 zkServer.sh stop 命令停掉leader可以发现又会重新选举出leader