# lsyncd和rsync实现文件实时同步

**实现：源服务器端同步数据到客户端**

## 客户端配置

>  接收端安装rsync

```bash
apt-get install -y rsync
或者yum install -y rsync
```

> 修改配置

```bash
vim /etc/rsyncd.conf
```

```bash
#Global Settings 
uid = root
gid = root
use chroot = no
max connections = 20
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
comment = hello world
#motd file = /etc/rsyncd.motd 

[repositories]
path = /var/opt/gitlab/git-data/repositories/  
auth users = repositories
read only = no
hosts allow = *
#hosts deny = 0.0.0.0/0.0.0.0
list = true
ignore errors
secrets file = /etc/repositories.password
```

>  配置密码文件

```bash
vim /etc/repositories.password
repositories:ciei
```

>  给文件授权

```bash
chmod 600 /etc/repositories.password 
```

> 启动rsync

```bash
systemctl start rsync.service
```

## 源服务器端配置

> 发送端安装rsync+lsyncd

```bash
#安装rsync

yum install -y rsync

systemctl start rsync

#安装lsyncd

yum install -y rsync
```

> 修改配置文件

```
vim /etc/lsyncd/lsyncd.conf.lua
```

```bash
settings {
    logfile = "/var/log/lsyncd/lsyncd.log",
    statusFile = "/var/log/lsyncd/lsyncd.status",
    inotifyMode = "CloseWrite",
    maxProcesses = 1000,
    maxDelays = 200
}

sync {
    default.rsync,
    source = "/var/opt/gitlab/git-data/repositories",
    target = "repositories@5.5.5.5::repositories",
    delay = 10,
    rsync = {
        binary = "/usr/bin/rsync",
        password_file = "/etc/repositories.client.pas",
        archive = true,
        compress = false,
        verbose = true,
        --delete = true
        }
}

}
```

> 配置密码

```bash
vim /etc/repositories.client.pas  
ciei
```

> 修改读写属性

```bash
chmod 600 /etc/repositories.client.pas
```

> lsyncd使用中遇到的问题

日志里显示：

```bash
Error: Terminating since out of inotify watches.Consider increasing /proc/sys/fs/inotify/max_user_watches
```

解决办法

```bash
vim /etc/sysctl.conf 添加如下内容：
fs.inotify.max_user_watches=99999999添加完执行 sysctl -p立即生效
```

## 测试

在源服务器端下创建文件，可在发生端查看实时同步的数据
