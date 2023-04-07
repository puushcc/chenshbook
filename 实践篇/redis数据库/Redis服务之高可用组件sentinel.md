# Redis服务之高可用组件sentinel

## 1、环境说明

| 角色       | ip地址       | 端口  |
| ---------- | ------------ | ----- |
| master     | 192.168.0.41 | 6379  |
| slave01    | 192.168.0.42 | 6379  |
| slave02    | 192.168.0.43 | 6379  |
| sentinel01 | 192.168.0.41 | 26379 |
| sentinel02 | 192.168.0.42 | 26379 |
| sentinel03 | 192.168.0.43 | 26379 |

## 2、redis主从复制集群搭建

> 在192.168.0.41/42/43上安装redis

安装过程省略

> 配置192.168.0.41/42/43上的redis监听在非本机127.0.0.1上并配置42/43上的redis从属于192.168.0.41

　master

```bash
bind 0.0.0.0
```

　slave01

```bash
bind 0.0.0.0
SLAVEOF 192.168.0.41 6379
```

　slave02

```bash
bind 0.0.0.0
SLAVEOF 192.168.0.41 6379
```

　在master上查看是否有两个从节点连接到master

```bash
info replication
```

验证：在master上写数据，看看是否能够及时同步到两个slave上？

## 3、配置sentinel，让其监控master

```bash
#三个sentinel的配置都是一样的，这里需要明确指定监控主从同步集群的master的ip地址和端口，以及有效法定票数，有效法定票数指的是至少有多少个sentinel主观认为master down了，然后才触发选举新master操作；通常在这种流言协议中，一般都是大于集群半数，如果是3台sentinel，至少要2台主观认为master宕机，才开始触发选举新master；如果是5台，那至少要3台；如果master配置的有认证密码，我们还需要在sentinel中指定认证密码；

sentinel monitor mymaster 192.168.0.41 6379 2
```

> 查看sentinel状态

```bash
info  sentinel
```

