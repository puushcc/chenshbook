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
