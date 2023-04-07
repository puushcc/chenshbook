# Keepalived

## keepalived安装

> 下载keepalived 安装包 下载地址

```bash
http://www.keepalived.org/software/
```

> 解压文件

```bash
[root@zk03 apps]# tar -zvxf keepalived-2.0.10.tar.gz 
```

> 执行 configure 准备编译文件

```bash
yum install openssl-devel
./configure --prefix=/usr/apps/keepalived --sysconf=/etc
```

> 执行 make&& make install 命令进行编译安装,安装完成可以看到/usr/apps/keepalived 多了三个目录 sbin bin share

```bash
[root@zk03 keepalived]# ll /usr/apps/keepalived
总用量 912
drwxr-xr-x. 2 root root     21 1月  25 22:39 bin
drwxrwxr-x. 9 1000 1000   4096 1月  25 22:38 keepalived-2.0.10
-rw-r--r--. 1 root root 927631 2月  22 2019 keepalived-2.0.10.tar.gz
drwxr-xr-x. 2 root root     24 1月  25 22:39 sbin
drwxr-xr-x. 5 root root     40 1月  25 22:39 share
```

> cd 到/usr/apps/keepalived 然后执行ln命令创建软连接

```bash
ln -s sbin/keepalived  /sbin
```

> 复制需要的文件到 /etc

```bash
cp /usr/apps/keepalived/keepalived-2.0.10/keepalived/etc/init.d/keepalived /etc/init.d
```

> 增加系统服务

```bash
chkconfig --add keepalived
```

> chkconfig keepalived on

```bash
[root@zk03 keepalived]# chkconfig keepalived on
注意：正在将请求转发到“systemctl enable keepalived.service”。
Created symlink from /etc/systemd/system/multi-user.target.wants/keepalived.service to /usr/lib/systemd/system/keepalived.service.
```

> 启动 服务service keepalived start

```bash
service keepalived start
```

## keepalived配置

配置文件可以分为三块：

> **全局定义块：**对整个 Keepalive 配置生效的，不管是否使用 LVS；

```bash
global_defs {
     notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
```

- notification_email:设置报警邮件地址即报警邮件接收者，可设置多个，每行一个；如果要开启邮件报警功能，需要开启本机的postfix或者sendmail服务；
- notification_email_from:用于设置邮件的发送地址，即报警邮件发送者；
- smtp_server:用于设置邮件的SMTP Server地址；
- smtp_connect_timeout:设置连接SMTP Server的超时时间；
- router_id:表示运行keepalived服务器的一个标识，是发邮件时显示在邮件主题中的信息；

> **VRRP实际定义块：**是keepalived的核心；

- vrrp_sync_group VRRP同步组

同步组是相对于多个VRRP实例而言的，在多个VRRP实例的环境中，每个VRRP实例所对应的网络环境会有所不同，假设一个实例处于网段A，另一个实例处于网段B，而如果VRRP只配置了A网段的检测，那么当B网段主机出现故障时，VRRP会认为自己仍处于正常状态，不会进行主备节点切换，这样问题就出现了。而同步组会将所有VRRP实例都加入同步组中，这样任何一个实例出现问题，都会导致keepalived进行主备切换；
以实例组group至少包含一个vrrp实例。

如：两个同步组的配置样例

```bash
vrrp_sync_group G1 {
group {
    VI_1
    VI_2
    VI_5
   }
    notify_backup "/usr/local/bin/vrrp.bak arg1 arg2"
    notify_master "/usr/local/bin/vrrp.mast arg1 arg2"
    notify_fault "/usr/local/bin/vrrp.fault arg1 arg2"
    notify_stop "/usr/local/bin/vrrp.stop arg1 arg2"
  }
vrrp_sync_group G2 {
    group {
    VI_3
    VI_4
     }
  }

#其中：G1同步组包含VI_1，VI_2，VI_5三个VRRP实例，G2同步组包含VI_3，VI_4两个实例，这5个实例将在vrrp_instance段进行定义
#keepalived配置中的一个通知机制，也是keepalived包含的四种状态：
#notify_master：指定当keepalived进入MASTER状态时要执行的脚本，这个脚本可以是一个状态报警脚本，也可以是一个服务管理脚本，允许传入参数；
#notify_backup：指定当keepalived进入BACKUP状态时要执行的脚本；
#notify_fault：指定当keepalived进入FAULT状态时要执行的脚本；
#notify_stop：指定当keepalived程序终止时需要执行的脚本；
```

> Vrrp实例vrrp_instance。VRRP实例配置即keepalived的高可用功能

VRRP实例段主要用来配置节点角色(主从)，实例绑定端口，节点间验证机制，集群服务IP等，如：

```bash
vrrp_instance VI_1 { 
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
	mcast_src_ip 
	garp_master_delay 10
	track_interface {
	eth0
	eth1
	}
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.200.16
        192.168.200.17 dev eth1
        192.168.200.18 dev eth2
    }
	virtual_routers {
		src 192.168.100.1 to 192.168.109.0/24 via 192.168.200.254 dev eth1
		192.168.110.0/24 via 192.168.200.254 dev eth1
		192.168.111.0/24 dev eth2
		192.168.112.0/24 via 192.168.100.254
		192.168.113.0/24 via 192.168.100.252 or 192.168.100.253
	}
	nopreempt
	preemtp_delay 300
}
```

vrrp_instance:是VRRP实例的标识，后跟VRRP实例名称；
state：用于指定keepalived的角色，MASTER表示主服务器，BACKUP表示备用服务器；
interface：用于指定HA监测网络的接口；
virtual_router_id：虚拟路由标识，是一个数字，同一个VRRP实例使用唯一的标识，即在同一个vrrp_instance下，MASTER和BACKUP必须是一致的；
priority:节点优先级，数字越大优先级越高(在一个VRRP_instance下，MASTER的优先级必须大于BACKUP的优先级)；
advert_int:用于设定MASTER与BACKUP主机之间同步检查的时间间隔，单位秒；
mcast_src_ip:用于设置发送多播包的地址，若不设置，将使用绑定网卡对应的IP地址；
garp_master_delay：用于设置在切换到MASTER状态后延时进行Gratuitous arp请求的时间；
track_interface:用于设置一些额外的网络监控接口，其中任何一个接口出现故障，keepalived都会进入FAULT状态；
authentication:用于设定节点间通信验证码类型和密码 ,主要类型有PASS和AH两种，在一个vrrp_instance下，MASTER和BACKUP必须使用相同的密码才能正常通信；
virtual_ipaddress:用于设置虚拟IP地址(VIP)，可设置多个，每行一个；keepalived通过ip address add命令的形式将VIP添加进系统中，而且IP形式可多样；
virtual_routers:和virtual_ipaddress段一样，用来设置在切换时添加或删除相关路由信息；
nopreempt:设置高可用集群不抢占功能；在使用不抢占时，只能在state状态为BACKUP的节点上设置，而且这个节点的优先级必须高于其他节点
preemtp_delay：用于设置抢占的延时时间，单位秒，如系统启动或重启之后网络需要经过一段时间才能正常工作，这时进行主备切换是没有必要的，此选项就是来设置这种情况发生的时间间隔；
vs_sync_daemon_inteface。负载均衡器之间的监控接口，类似于HA HeartBeat的心跳线。但它的机制优于Heartbeat，因为它没有“裂脑”这个问题，它是以优先级这个机制来规避这个麻烦的。在DR模式中，lvs_sync_daemon_inteface 与服务接口interface 使用同一个网络接口。

> keepalived的LVS配置

LVS配置段以virtual_server为开始标识，此段分为两部分：real_server段和健康检测段

**real_server段**

```bash
virtual_server 192.168.12.200 80 {
	delay_loop 6 
	lb_algo rr
	lb_kind DR
	persistence_timeout 50 
	persistence_granularity 
	protocol TCP
	ha_suspend
	virtualhost 
	sorry_server 
}

```

- virtual_server:设置虚拟服务器开始的标识，后跟虚拟IP地址空格符服务端口；
- delay_loop：设置健康检查的时间间隔，单位秒；
- lb_algo：设置负载调度算法，常用的有rr,wrr,lc,wlc等；
- lb_kind:设置LVS实现负载均衡的机制，有NAT，TUN和DR模式；
- persistence_timeout：会话保持时间，单位秒，该选项使用户的请求会一直分发到某个服务节点，直到超过这个会话的保持时间；
- persistence_granularity：配合persistence_timeout使用，后面跟的值是子网掩码，表示持续连接的粒度，默认是255.255.255.255即一个单独的客户端IP，如果改为255.255.255.0那么客户端所在的整个网段的请求都会分发到同一台realserver上；
- protocol：指定转发协议类型，有TCP和UDP两种选型；
- ha_suspend：节点状态从MASTER到BACKUP切换时，暂不启用real server节点的健康检查；
- virtualhost：再通过HTTP_GET/SSL_GET做健康检测时，指定的web服务器的虚拟主机地址；
- sorry_server：相当于一个备用节点，在所有realserver失效后，启用这个节点，

```bash
real_server 192.168.12.132 80 {
	weight 3
	inhibit_on_failure
	notify_up | 
	notify_down | 
}
```

- real_server:real_server段开始的标识，用来指定real server节点，后跟real server的真实IP地址和端口(有空格)
- weight：配置real server节点的权值，数字越大权值越高；
- inhibit_on_failure:表示在检测到real server节点失效后，把他的weight值置为0，而不是从IPVS中删除；
- notify_up:和notify_master相同，后跟一个脚本，表示在检测到real server节点服务处于UP状态后只执行的脚本；
- notify_down:表示在检测到real server节点服务处于DOWN状态后只执行的脚本

**健康检测段**
常见的有HTTP_GET,SSL_GET,TCP_CHECK,SMTP_CHECK,MISC_CHECK
TCP_CHECK检测方式示例：

```bash
TCP_CHECK {
	connect_port 80
	connect_timeout 3
	nb_get_retry 3
	delay_before_retry 3
}
```

其中：
connect_port:健康状态检查的端口，如果不指定，默认是real_server指定的端口；
connect_timeout：表示无响应超时时间，单位秒；
nb_get_retry：重试次数；
delay_before_retry：重试间隔时间；

HTTP_GET和SSL_GET检测方式示例：
```bash
HTTP_GET | SSL_GET {
	url {
		path /index.html
		digest e6c271eb5f017f280cf97ec2f51b02d3
		status_code 200 
	}
	connect_port 80
	bindto 192.168.12.80
	connect_timeout 3
	nb_get_retry 3
    delay_before_retry 2
  }
```

其中：
url:用来指定HTTP/SSL检查的URL信息，可指定多个URL；
path：后跟详细的URL路径；
digest：SSL检查后的摘要信息，可通过genhash命令获取，如genhash -s 192.168.12.80 -p 80 -u /index.html
status_code：指定HTTP检查返回正常状态吗的类型，一般为200；
bindto：表示通过此地址来发送请求对服务器进行健康检查；
**配置示例**

```bash
[root@localhost ~]# cat /usr/local/keepalived/etc/keepalived/keepalived.conf
! Configuration File for keepalived
 
global_defs {					#全局配置
	notification_email {		#指定keepalived在发生切换时需要发送email到的对象，一行一个
		acassen@firewall.loc	#指定收件人邮箱
		failover@firewall.loc
		sysadmin@firewall.loc
	}
	notification_email_from Alexandre.Cassen@firewall.loc #指定发件人
	smtp_server 192.168.200.1	#指定smtp服务器地址
	smtp_connect_timeout 30		#指定smtp连接超时时间
	router_id LVS_DEVEL			#此处注意router_id为负载均衡标识，在局域网内应该是唯一的。
	vrrp_skip_check_adv_addr
	vrrp_strict
	vrrp_garp_interval 0
	vrrp_gna_interval 0
}

#如果这块没有，就不用管他 
vrrp_sync_group VG_1{				#监控多个网段的实例
	group {
		inside_network				#实例名
		outside_network
	}
	notify_master /path/xx.sh		#指定当切换到master时，执行的脚本
	netify_backup /path/xx.sh		#指定当切换到backup时，执行的脚本
	notify_fault "path/xx.sh VG_1" 	#故障时执行的脚本
	notify /path/xx.sh
	smtp_alert 						#使用global_defs中提供的邮件地址和smtp服务器发送邮件通知
}
 
vrrp_instance inside_network {
	state BACKUP 			#指定那个为master，那个为backup，如果设置了nopreempt这个值不起作用，主备考priority决定
	interface eth0 			#设置实例绑定的网卡
	dont_track_primary 		#忽略vrrp的interface错误（默认不设置）
	track_interface{ 		#设置额外的监控，里面那个网卡出现问题都会切换
		eth0
		eth1
	}
	mcast_src_ip			#发送多播包的地址，如果不设置默认使用绑定网卡的primary ip
	garp_master_delay		#在切换到master状态后，延迟进行gratuitous ARP请求
	virtual_router_id 50	#VPID标记
	priority 99				#优先级，高优先级竞选为master
	advert_int 1			#检查间隔，默认1秒
	nopreempt				#设置为不抢占 注：这个配置只能设置在backup主机上，而且这个主机优先级要比另外一台高
	preempt_delay			#抢占延时，默认5分钟
	debug					#debug级别
	authentication {		#设置认证
		auth_type PASS		#认证方式，类型主要有PASS、AH 两种
		auth_pass 111111	#认证密码
	}
	virtual_ipaddress {		#设置vip
		192.168.36.200
	}
}
 
vrrp_instance VI_1 {	#虚拟路由的标识符
	state MASTER		#状态只有MASTER和BACKUP两种，并且要大写，MASTER为工作状态，BACKUP是备用状态
	interface eth0		#通信所使用的网络接口
    lvs_sync_daemon_inteface eth0  #这个默认没有，相当于心跳线接口，DR模式用的和上面的接口一样，也可以用机器上的其他网卡eth1，用来防止脑裂。
    virtual_router_id 51  #虚拟路由的ID号，是虚拟路由MAC的最后一位地址
    priority 100		  #此节点的优先级，主节点的优先级需要比其他节点高
    advert_int 1		  #通告的间隔时间
    nopreempt			  #设置为不抢占 注：这个配置只能设置在backup主机上，而且这个主机优先级要比另外一台高
    preempt_delay		  #抢占延时，默认5分钟
    authentication {	  #认证配置
		auth_type PASS	  #认证方式
        auth_pass 1111	  #认证密码
    }
    virtual_ipaddress {	  #虚拟ip地址,可以有多个地址，每个地址占一行，不需要子网掩码，同时这个ip 必须与我们在lvs 客户端设定的vip 相一致！
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}
 
virtual_server 192.168.200.100 443 { #集群所使用的VIP和端口
    delay_loop 6					#健康检查间隔，单位为秒
    lb_algo rr						#lvs调度算法rr|wrr|lc|wlc|lblc|sh|dh
    nat_mask 255.255.255.0			#VIP掩码
    lb_kind NAT						#负载均衡转发规则。一般包括DR,NAT,TUN 3种
    persistence_timeout 50			#会话保持时间，会话保持，就是把用户请求转发给同一个服务器，不然刚在1上提交完帐号密码，就跳转到另一台服务器2上了
    protocol TCP					#转发协议，有TCP和UDP两种，一般用TCP，没用过UDP
    persistence_granularity <NETMASK> #lvs会话保持粒度
 
    real_server 192.168.201.100 443 { #真实服务器，包括IP和端口号
        weight 1					#默认为1,0为失效
        inhibit_on_failure			#在服务器健康检查失效时，将其设为0，而不是直接从ipvs中删除
        notify_up <string> | <quoted-string> #在检测到server up后执行脚本
        notify_down <string> | <quoted-string> #在检测到server down后执行脚本
 
		TCP_CHECK {					#通过tcpcheck判断RealServer的健康状态
            connect_timeout 3		#连接超时时间
            nb_get_retry 3			#重连次数
            delay_before_retry 3	#重连间隔时间
            connect_port 23			健康检查的端口的端口
            bindto <ip>  
        }
           
        HTTP_GET | SSL_GET {		#健康检测方式，可选有 SSL_GET、TCP_CHECK、HTTP_GET
            url {					#检查url，可以指定多个
              path /				#检查的url路径
              digest ff20ad2481f97b1754ef3e12ecd3a9cc  #需要检查到的内容。检查后的摘要信息。
              status_code 200		#检查的返回状态码
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            connect_timeout 3		#连接超时时间
            nb_get_retry 3			#检测尝试几次
            delay_before_retry 3	#检测的时间间隔
        }
    }
}
```

## nginx高可用

> **配置01虚拟机**：

keepalived.conf配置文件：

```bash
global_defs {
   router_id lb111 #起一个名字
}

vrrp_instance atguigu {# 后面atguigu是实例名字，随便起
    state MASTER # 表示是主机，不是备用
    interface ens33 #对应本机网卡的名字，用ifconfig查看
    virtual_router_id 50 #不用改
    nopreempt
    priority 100 #主备竞选时候的优先级
    advert_int 1 #间隔检测的时间
    authentication { #这个是分组，一个局域网内可能有多台机器配置
        auth_type PASS #keepalived了，总得区分开谁跟谁是一组配置
        auth_pass 1111 #这个只要同组保持一致就行
    }
    virtual_ipaddress {#虚拟ip，可以虚拟好几个，咱们就用一个
        192.168.200.11 #来实验就行
        #192.168.200.12
        #192.168.200.13
    }
}
```

> **配置04虚拟机**

```bash
global_defs {
   router_id lb110 #变一个
}

vrrp_instance atguigu {# atguigu，必须一致！！！
    state BACKUP #备用机
    interface ens33
    virtual_router_id 50 #必须一致！！！
    nopreempt
    priority 50 # 优先级低一点，因为是备用机
    advert_int 1
    authentication { #必须一致！！！
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.125.100
    }
}
```

