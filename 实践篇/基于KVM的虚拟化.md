# 基于KVM的虚拟化

## 1、基础准备

- CentOS7.x操作系统
- 一台物理机服务器

## 2、服务器基本配置

配置网卡IP信息

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33

TYPE=Ethernet #网卡类型：为以太网
PROXY_METHOD=none #代理方式：关闭状态
BROWSER_ONLY=no #只是浏览器
BOOTPROTO=static # 需修改，网卡的引导协议，设置为静态IP
DEFROUTE=yes #默认路由
IPV4_FAILURE_FATAL=no #是不开启IPV4致命错误检测
IPV6INIT=yes #IPV6是否自动初始化
IPV6_AUTOCONF=yes #IPV6是否自动配置
IPV6_DEFROUTE=yes #IPV6是否可以为默认路由
IPV6_FAILURE_FATAL=no #是否开启IPV6致命错误检测
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=em1 #网卡物理设备名称
UUID=333114a0-3213-4a94-1132-72473adf11e1 #通用唯一识别码
DEVICE=em1 #网卡设备名称, 必须和`NAME`值一样
ONBOOT=yes #需修改，是否开机启动
#重点关注这部分，需添加
IPADDR=192.168.16.10 #IP地址
NETMASK=255.255.255.0 #子网掩码
GATEWAY=192.168.16.254  #局域网网关
DNS1=8.8.8.8 #DNS解析服务器
```

```bash
systemctl restart network
```

```bash
ping www.baidu.com保证连通外网环境
```

关闭 selinux

```bash
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/sysconfig/selinux
setenforce 0
```

关闭 iptables

```bash
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl status firewalld.service
```

## 3、Libvirt、KVM安装

> 检测当前环境是否支持KVM安装

```bash
cat /proc/cpuinfo | egrep 'vmx|svm'
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch epb invpcid_single intel_pt ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt xsaveopt xsavec xgetbv1 dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear spec_ctrl intel_stibp flush_l1d
......
#有vmx信息输出即可，如果没有任何的输出，说明cpu不支持，无法使用KVM虚拟化。
```

> 确保BIOS里开启虚拟化功能，即查看是否加载KVM模块

```bash
lsmod | grep kvm
#如果没有加载，运行以下命令：
modprobe kvm
modprobe kvm-intel
lsmod | grep kvm

#出现下面内容
kvm_intel             188688  8 
kvm                   636965  1 kvm_intel
irqbypass              13503  5 kvm
```

> 安装libvirt及kvm

```bash
yum -y install libcanberra-gtk2 qemu-kvm.x86_64 
qemu-kvm-tools.x86_64  libvirt.x86_64 libvirt-cim.x86_64 libvirt-client.x86_64
libvirt-java.noarch  libvirt-python.x86_64 libiscsi-1.7.0-5.el6.x86_64  dbus-devel
virt-clone tunctl virt-manager libvirt libvirt-python python-virtinst


qemu-kvm               # KVM基础模块
qemu-kvm-tools         # KVM调试工具，可不安装
virt-install           # 构建虚拟机的命令行工具
qemu-img               # qemu组件，创建磁盘、启动虚拟机等
bridge-utils           # 网络支持工具
libvirt                # 虚拟机管理工具
virt-manager           # 图形界面管理虚拟机
libguestfs-tools       # 用来管理虚拟机磁盘格式

```

> kvm服务开启

```bash
systemctl start libvirtd.service
systemctl enable libvirtd.service
systemctl status libvirtd
```

## 4、虚拟机安装

使用`virt-manager`进行可视化操作

```bash
virt-manager
```

后续操作与VMware workstation创建虚拟机类似，不做说明

## 5、WEB管理工具kimchi

>  下载wok安装包和kimchi安装包

　　wok下载地址：https://github.com/kimchi-project/kimchi/releases/download/2.5.0/wok-2.5.0-0.el7.centos.noarch.rpm

　　kimchi下载地址：https://github.com/kimchi-project/kimchi/releases/download/2.5.0/kimchi-2.5.0-0.el7.centos.noarch.rpm

> 更新软件包并安装epel扩展源

```bash
yum update
yum install epel*
```

> 安装wok和kimchi

```bash
yum install ./wok-2.5.0-0.el7.centos.noarch.rpm ./kimchi-2.5.0-0.el7.centos.noarch.rpm
```

> 启动nginx和wokd服务

```bash
systemctl daemon-reload
systemctl start wokd
nginx
```

> 访问kimchi界面

用https协议去访问宿主机的8001端口

账号密码与系统账号一致

## 6、命令行

- ### virt-manager

> 打开虚拟机图形化管理页面

- ### virsh

​		命令行工具virsh，创建虚拟机，查看虚拟机，动态热插拔硬盘，给虚拟机做快照，迁移、启动、停止、挂起、暂停、删除虚拟机等等操作

​		virsh命令大概分了，Domain Management（域管理），Domain Monitoring（域监控）、 Host and Hypervisor（主机及虚拟化）、Interface（网卡接口）、Network Filter（网络防火墙）、Networking（网络）、Node Device（节点设备驱动）、Secret、Snapshot（快照）、Storage Pool（存储池或存储策略）、Storage Volume（存储卷）、Virsh itself（virsh shell自身相关）这些组，如果查看某一组帮助信息，我们可以使用virsh help +组名

> virsh list

​	列出当前宿主机上的虚拟机列表，默认不加任何选项表示列出当前处于运行状态的虚拟机列表（活跃的）

```bash
virsh list
 Id    名称                         状态
----------------------------------------------------

virsh list --all
 Id    名称                         状态
----------------------------------------------------
 -     centos7.0                      关闭
```

> **创建pool**

```bash
#【例】创建名为extra_images的pool，volume存放于/opt/data/kvm/
virsh pool-define-as extra_images dir - - - - "/opt/data/kvm/"
virsh pool-build extra_images
virsh pool-start extra_images
virsh pool-info extrat_images
#如果 Autostart: no，执行 virsh pool-autostart <pool> 可以让它自启动
```

> **创建volume**

```bash
virsh vol-create-as <pool> <image name>.qcow2 <image size> --format qcow2
#说明：volume会保存在default pool，即 /var/lib/libvirt/images，如果该路径挂载的磁盘空间不能满足要求，需要在其它挂载点的路径保存volume，得创建其它pool
```

> **删除虚拟机**

```bash
virsh destroy <VM Name>   强制停止虚拟机

virsh undefine <VM Name>  删除虚拟机

updatedb

locate <VM Name>
```

> 查看、编辑及备份KVM 虚拟机配置文件

```bash
#KVM 虚拟机默认的配置文件在 /etc/libvirt/qemu 目录下，默认是以虚拟机名称命名的.xml 文件
ls /etc/libvirt/qemu/
#KVM 虚拟机配置文件的修改。可以使用vi 或 vim 命令进行编辑修改，但不建议。正确的做法为 virsh edit KVM-NAME
virsh edit snale 
#备份KVM 虚拟机配置文件
mkdir /data/kvmback
virsh dumpxml snale >/data/kvmback/snale_back.xml
```

> KVM 虚拟机开启（启动）

```bash
virsh start snale2
```

> KVM 虚拟机关机

```bash
virsh shutdown snale2
#注：KVM 虚拟机默认是无法用virsh shutdown 进行关机的，如果要想使用该命令关机，则必须在kvm 虚拟机上安装acpid acpid-sysvinit 两个包，启动acpid 服务，并且加入随机启动，如下：
yum install -y acpid acpid-sysvinit
service acpid start
```

> 强制关机（强制断电）

```bash
virsh destroy snale
域 snale 被删除
```

> 暂停（挂起）KVM 虚拟机

```bash
virsh suspend snale
```

> 恢复被挂起的 KVM 虚拟机

```bash
virsh resume snale
```

> 删除KVM 虚拟机

```bash
virsh undefine snale
```

该方法只删除配置文件，磁盘文件未删除，相当于从虚拟机中移除。

> KVM 设置为随物理机启动而启动（开机启动）

```bash
virsh autostart snale
```

- qemu-img

> 创建磁盘

```bash
qemu-img create -f qcow2 ./centos7.qcow2 10G
```

- virt-install

> 创建虚拟机

```bash
virt-install --virt-type kvm --name c7 --ram 1024 --vcpus 2 --cdrom=/kvm/iso/CentOS-7-x86_64-Minimal-1708.iso --disk path=/kvm/images/centos7.qcow2 --network=default --graphics vnc,listen=0.0.0.0 --noautoconsole	

```
