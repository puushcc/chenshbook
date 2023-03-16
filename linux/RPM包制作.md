# RPM包制作

## 1、准备工作

> ### 安装基础软件

```bash
#gcc，make 用于编译c语言源代码
#dos2unix 保证文件可执行性
yum -y install gcc make dos2unix rpmlint
```

> ### Rpm打包软件安装

```bash
#RPM打包使用的是rpmbuild命令
yum -y install rpm-build
#也可以直接安装rpmdevtools
yum -y install rpmdevtools
```

## 2、简单案例

> 创建工作空间

```bash
[root@node2 ~]# rpmdev-setuptree
[root@node2 ~]# tree rpmbuild
rpmbuild
├── BUILD
├── RPMS
├── SOURCES
├── SPECS
└── SRPMS

5 directories, 0 files

#或者用mkdir命令也可以,二选一
mkdir -p rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
```

> ### 准备SPEC

```bash
cd ~/rpmbuild
vim SPECS/hello-world.spec
```

```bash
Name: hello-world
Version: 1.0.0
Release: 1.el7
Summary: A test demo
License: FIXME
 
%description
This is a test demo
 
%prep
# nothing
 
%build
cat > hello-world.sh <<EOF
#!/bin/bash
echo Hello World
EOF
 
%install
mkdir -p %{buildroot}/usr/bin/
install -m 755 hello-world.sh %{buildroot}/usr/bin/hello-world.sh
 
%files
/usr/bin/hello-world.sh
 
%changelog
# nothing
```

> ### RPM打包

```bash
rpmbuild -ba SPECS/hello-world.spec
```

> #### 安装RPM包

```bash
rpm -ivh /root/rpmbuild/RPMS/x86_64/hello-world-1.0.0-1.el7.x86_64.rpm
```

> #### 验证安装

```bash
[root@node2 ~/rpmbuild]# rpm -q hello-world
hello-world-1.0.0-1.el7.x86_64
[root@node2 ~/rpmbuild]# hello-world.sh 
Hello World
[root@node2 ~/rpmbuild]#
```

> #### 卸载rpm包

```bash
[root@node2 ~/rpmbuild]# rpm -e hello-world
[root@node2 ~/rpmbuild]# rpm -q hello-world
未安装软件包 hello-world 
[root@node2 ~/rpmbuild]# hello-world.sh 
-bash: /usr/bin/hello-world.sh: 没有那个文件或目录
[root@node2 ~/rpmbuild]# 
```

## 3、制作redis的rpm包

> 创建工作空间

```bash
[root@node2 ~]# rpmdev-setuptree
[root@node2 ~]# tree rpmbuild
rpmbuild
├── BUILD
├── RPMS
├── SOURCES
├── SPECS
└── SRPMS

5 directories, 0 files

#或者用mkdir命令也可以,二选一
mkdir -p rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
```

> 准备SPEC

```bash
cd ~/rpmbuild
vim SPECS/redis.spec
```

```bash
Name:           redis	
Version:	3.2.100
Release:	1%{?dist}
Summary:	redis-server

License:	GPLv3
Source0:	redis-3.2.100.tar.gz 

BuildRequires:	bash


%global		app_bin_dir		/usr/local/%{name}
%global		redis_password		ciei
%global		debug_package 		%{nil}

%description
redis server

%prep
%setup -q  -n %{name}

%install
mkdir -p %{buildroot}/%{app_bin_dir}/
cp -r ./* %{buildroot}/%{app_bin_dir}/

%files
%defattr(-,root,root,500)
%{app_bin_dir}/*

%post
# 修改redis配置
sed -i -n -e '/^bind/!p' -e '$a bind 0.0.0.0' %{app_bin_dir}/conf/redis.conf
sed -i -n -e "/^requirepass/!p" -e "\$a requirepass %{redis_password}" %{app_bin_dir}/conf/redis.conf

# 生成redis.service文件
cat > /etc/systemd/system/redis.service <<EOF
[Unit]
Description=redis service
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/conf/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF

systemctl enable redis.service
# 启动
systemctl start redis

# 创建软连接
ln -s /usr/local/redis/bin/redis-cli /usr/bin/

%preun
systemctl stop redis
systemctl disable redis

%postun
rm -fr %{app_bin_dir}
rm -fr /etc/systemd/system/redis.service
```

> 准备redis-3.2.4.tar.gz安装包，已编译, 解压即可使用

```bash
tar -czvf redis-3.2.100.tar.gz /usr/local/redis
mv /usr/local/redis-3.2.100.tar.gz ~/rpmbuild/SOURCES/
```

>  RPM打包

```bash
rpmbuild -ba SPECS/hello-world.spec
```

> 完整路径

```bash
[root@localhost rpmbuild]# tree 
.
├── BUILD
│   └── redis
│       ├── bin
│       │   ├── dump.rdb
│       │   ├── redis-benchmark
│       │   ├── redis-check-aof
│       │   ├── redis-check-rdb
│       │   ├── redis-cli
│       │   ├── redis-sentinel
│       │   └── redis-server
│       └── conf
│           └── redis.conf
├── BUILDROOT
├── RPMS
│   └── x86_64
│       └── redis-3.2.100-1.el7.x86_64.rpm
├── SOURCES
│   └── redis-3.2.100.tar.gz
├── SPECS
│   └── redis.spec
└── SRPMS
    └── redis-3.2.100-1.el7.src.rpm
```

## 4、制作nginx的rpm包

> 准备SPEC

```bash
Name:           nginx
Version:	1.23.3
Release:	1%{?dist}
Summary:	nginx-server

License:	GPLv3
Source0:	nginx-1.23.3.tar.gz 

BuildRequires:	bash


%global		app_bin_dir		/usr/local/%{name}
%global		debug_package 		%{nil}

%description
nginx server

%prep
%setup -q  -n %{name}

%install
mkdir -p %{buildroot}/%{app_bin_dir}/
cp -r ./* %{buildroot}/%{app_bin_dir}/

%files
%defattr(-,root,root,755)
%{app_bin_dir}/*

%post
# 修改redis配置
#sed -i -n -e '/^bind/!p' -e '$a bind 0.0.0.0' %{app_bin_dir}/conf/redis.conf
#sed -i -n -e "/^requirepass/!p" -e "\$a requirepass %{redis_password}" %{app_bin_dir}/conf/redis.conf

# 生成redis.service文件
cat > /etc/systemd/system/nginx.service <<EOF
[Unit]
Description=nginx 
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF

systemctl enable nginx.service
# 启动
systemctl start nginx

# 创建软连接
ln -s /usr/local/nginx/sbin/nginx /usr/bin/

%preun
systemctl stop nginx
systemctl disable nginx

%postun
rm -rf /usr/bin/nginx
rm -fr %{app_bin_dir}
rm -fr /etc/systemd/system/nginx.servic
```

其他步骤可参考redis

## 5、制作MySQL的rpm包

>  准备SPEC

```bash
Name:		mysql
Version:	8.0.32
Release:	1%{?dist}
Summary:	mysql-server

License:	GPLv3
Source0:	mysql-8.0.32.tar.gz 

BuildRequires:	bash

%global		app_bin_dir		/data/%{name}
%global		redis_password		ciei
%global		debug_package 		%{nil}

%description
mysql server

%prep
%setup -q  -n %{name}


%install
mkdir -p %{buildroot}/%{app_bin_dir}/
cp -r ./* %{buildroot}/%{app_bin_dir}/
%files
%defattr(-,mysql,mysql,755)
%{app_bin_dir}/*


%post
# 生成redis.service文件
cat > /usr/lib/systemd/system/mysql.service <<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql

# Have mysqld write its state to the systemd notify socket
Type=notify

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Start main service
ExecStart=/data/mysql/bin/mysqld --defaults-file=/data/mysql/conf/my.cnf $MYSQLD_OPTS 

# Use this to switch malloc implementation
EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 10000

Restart=on-failure

RestartPreventExitStatus=1

# Set environment variable MYSQLD_PARENT_PID. This is required for restart.
Environment=MYSQLD_PARENT_PID=1

PrivateTmp=false
EOF

systemctl enable mysql.service
# 启动
systemctl start mysql

# 创建软连接
ln -s /data/mysql/bin/mysql /usr/bin/

%preun
systemctl stop mysql
systemctl disable mysql

%postun
rm -fr %{app_bin_dir}
rm -fr /usr/lib/systemd/system/mysql.service
```

其他步骤可参考redis

## 6、zookeeper和kafka的rpm包

>  准备SPEC

```bash
cat > /usr/lib/systemd/system/zookeeper.service << EFO
[Unit]
Description=Zookeeper Service
After=network.target

[Service]
Type=forking
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/java/jdk1.8.0_131/bin"
User=root
Group=root

ExecStart=/data/kafka/zookeeper-3.4.14/bin/zkServer.sh start
ExecStop=/data/kafka/zookeeper-3.4.14/bin/zkServer.sh stop
ExecReload=/data/kafka/zookeeper-3.4.14/bin/zkServer.sh restart
Restart=on-failure

[Install]
WantedBy=multi-user.target
EFO



cat > /usr/lib/systemd/system/kafka.service << EFO
[Unit]
Description=Kafka Service
After=network.target zookeeper.service

[Service]
Type=simple
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/java/jdk1.8.0_131/bin"
User=root
Group=root

ExecStart=/data/kafka/kafka_2.12-2.3.0/bin/kafka-server-start.sh /data/kafka/kafka_2.12-2.3.0/config/server.properties
ExecStop=/data/kafka/kafka_2.12-2.3.0/bin/kafka-server-stop.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
EFO
```

其他步骤可参考redis

## 备查

> ### rpmbuild目录

| 默认位置             | 宏代码         | 名称              | 用途                                         |
| -------------------- | -------------- | ----------------- | -------------------------------------------- |
| ~/rpmbuild/SPECS     | %_specdir      | Spec 文件目录     | 保存 RPM 包配置（.spec）文件                 |
| ~/rpmbuild/SOURCES   | %_sourcedir    | 源代码目录        | 保存源码包（如 .tar 包）和所有 patch 补丁    |
| ~/rpmbuild/BUILD     | %_builddir     | 构建目录          | 源码包被解压至此，并在该目录的子目录完成编译 |
| ~/rpmbuild/BUILDROOT | %_buildrootdir | 最终安装目录      | 保存 %install 阶段安装的文件                 |
| ~/rpmbuild/RPMS      | %_rpmdir       | 标准 RPM 包目录   | 生成/保存二进制 RPM 包                       |
| ~/rpmbuild/SRPMS     | %_srcrpmdir    | 源代码 RPM 包目录 | 生成/保存源码 RPM 包(SRPM)                   |

> ### spec文件阶段

| 阶段d     | 读取的目录d     | 写入的目录d     | 具体动作                                                     |
| --------- | --------------- | --------------- | ------------------------------------------------------------ |
| %prepd    | %_sourcedird    | %_builddird     | 读取位于 %_sourcedir 目录的源代码和 patch 。之后，解压源代码至 %_builddir 的子目录并应用所有 patch。 |
| %buildd   | %_builddird     | %_builddird     | 编译位于 %_builddir 构建目录下的文件。通过执行类似 ./configure && make 的命令实现。 |
| %installd | %_builddird     | %_buildrootdird | 读取位于 %_builddir 构建目录下的文件并将其安装至 %_buildrootdir 目录。这些文件就是用户安装 RPM 后，最终得到的文件。注意一个奇怪的地方: 最终安装目录 不是 构建目录。通过执行类似 make install 的命令实现。 |
| %checkd   | %_builddird     | %_builddird     | 检查软件是否正常运行。通过执行类似 make test 的命令实现。很多软件包都不需要此步。 |
| bind      | %_buildrootdird | %_rpmdird       | 读取位于 %_buildrootdir 最终安装目录下的文件，以便最终在 %_rpmdir 目录下创建 RPM 包。在该目录下，不同架构的 RPM 包会分别保存至不同子目录， noarch 目录保存适用于所有架构的 RPM 包。这些 RPM 文件就是用户最终安装的 RPM 包。 |
| srcd      | %_sourcedird    | %_srcrpmdird    | 创建源码 RPM 包（简称 SRPM，以.src.rpm 作为后缀名），并保存至 %_srcrpmdir 目录。SRPM 包通常用于审核和升级软件包。 |

> ### 代表路径的宏列表

```bash
%{_sysconfdir}        /etc
%{_prefix}            /usr
%{_exec_prefix}       %{_prefix}
%{_bindir}            %{_exec_prefix}/bin
%{_libdir}            %{_exec_prefix}/%{_lib}
%{_libexecdir}        %{_exec_prefix}/libexec
%{_sbindir}           %{_exec_prefix}/sbin
%{_sharedstatedir}    /var/lib
%{_datarootdir}       %{_prefix}/share
%{_datadir}           %{_datarootdir}
%{_includedir}        %{_prefix}/include
%{_infodir}           /usr/share/info
%{_mandir}            /usr/share/man
%{_localstatedir}     /var
%{_initddir}          %{_sysconfdir}/rc.d/init.d
%{_var}               /var
%{_tmppath}           %{_var}/tmp
%{_usr}               /usr
%{_usrsrc}            %{_usr}/src
%{_lib}               lib (lib64 on 64bit multilib systems)
%{_docdir}            %{_datadir}/doc
%{buildroot}          %{_buildrootdir}/%{name}-%{version}-%{release}.%{_arch}
$RPM_BUILD_ROOT       %{buildroot}	
```

> ### rpm常用命令

- 手动安装 rpm 包 `rpm-ivh xxxxx.rpm`
- 查看 rpm 包信息 `rpm-qpi xxxxx.rpm`
- 查看 rpm 包依赖 `rpm -qpR xxxxx.rpm`
- 查看 rpm 包中包含那些文件 `rpm -qlp xxxxx.rpm` 可以加grep搜索 `rpm -qlp xxxxx.rpm|grep spec`
- 使用工具rpm2cpio提取文件： `rpm2cpio xxxxx.rpm |cpio -ivd xxx.jpg`
- 用rpm2cpio将rpm文件转换成cpio文件 `rpm2cpio xxxxxx.rpm >xxxxx.cpio`
- 用cpio解压cpio文件 `cpio -i --make-directories`
- 提取所有文件： `rpm2cpio xxx.rpm | cpio -vi` `rpm2cpio xxx.rpm | cpio -idmv` `rpm2cpio xxx.rpm | cpio --extract --make-directories`
- cpio 参数说明: **i** 和 **extract** 表示提取文件 **v** 表示指示执行进程 **d** 和 **make-directory** 表示根据包中文件原来的路径建立目录 **m** 表示保持文件的更新时间
- 查看rpm包里的pre和post install脚本： `rpm -qp --scripts xxxxx.rpm`
- 查看安装的过程中，代码的执行过程： `rpm -ih -vv xxxxx.rpm`

## 参考

[Linux二进制包(RPM包)制作教程(教程+资料+案例)](https://dysoso.gitee.io/post/linux/build/linux%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%8C%85rpm%E5%8C%85%E5%88%B6%E4%BD%9C%E6%95%99%E7%A8%8B/#%E4%BA%94%E7%AE%80%E5%8D%95%E6%A1%88%E4%BE%8B)

