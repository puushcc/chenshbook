# 安装DOCKER

## 1、yum安装docker

> 卸载旧版本

```bash
yum remove docker
```

> 需要的安装包

```bash
yum install -y yum-utils
```

> 设置镜像的仓库

```bash
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo  #默认是国外的
    
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  #阿里云
```

> 安装docker        docker-ce  社区 ee   企业版

```bash
yum install docker-ce docker-ce-cli containerd.io
```

> 启动docker

```bash
systemctl start docker
```

> 使用dockers version 是否安装成功

```
dockers version
```

> 卸载docker

```bash
yum remove docker-ce docker-ce-cli containerd.io
```

> 删除资源   /var/lib/docker   docker的默认工作路径

```shell
rm -rf /var/lib/docker
```
> 配置镜像加速器
> 针对Docker客户端版本大于 1.10.0 的用户
> 您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://qenyrjks.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
## 2、二进制安装dokcer
>  将Docker 二进制压缩包解压

```bash
tar xf docker-19.03.9.tgz 
ls docker
```
>  将二进制程序放到 Docker 的安装目录并链接到 $PATH 包含的目录下

```bash
mv docker/* /opt/apps/docker/bin/
find /opt/apps/docker/bin/ -type f | xargs -i ln -s {} /usr/local/bin/
```
> docker配置

创建 Docker 的配置文件（根据实际情况修改）

```bash
vim /opt/apps/docker/conf/daemon.json
{
  "registry-mirrors": ["https://7hsct51i.mirror.aliyuncs.com"],
  "log-driver":"json-file",
  "log-opts": {"max-size":"500m", "max-file":"3"},
  "insecure-registries": ["192.168.0.3"]  
}
```
> 配置docker启动管理

```bash
vim /usr/lib/systemd/system/docker.service
```

```bash
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service containerd.service
Wants=network-online.target

[Service]
Type=notify
ExecStart=/opt/apps/docker/bin/dockerd \
  --config-file /opt/apps/docker/conf/daemon.json \
  --data-root /opt/apps/docker/data
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
```

> 启动 Docker 并设置开机自启

```bash
systemctl start docker 
systemctl enable docker
```
> 检查数据目录如果正常生成数据则说明 Docker 运行正常

```bash
ls /opt/apps/docker/data
```

