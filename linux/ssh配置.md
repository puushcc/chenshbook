# SSH配置

ssh基于key验证有如下好处

- 更加安全方便。我们不用去记繁琐的用户密码，也不担心密码泄露。（我们可以把sshd服务配置成只允许基于KEY验证登录）

- 基于key验证实现免密登录，可以实现远程批量操作服务器，方便脚本编写，使得我们在执行远程操作命令时就好像在本地执行命令简单（如scp,ssh）

- 有效防止暴力猜口令的威胁。

## 1、免密配置

1、在客户端生成用户密钥对

```bash
ssh-keygen -t rsa

ll .ssh/
```

2、ssh-copy-id来把用户公钥文件信息拷贝到服务端对应的用户家目录

```bash
ssh-copy-id -i .ssh/id_rsa.pub root@192.168.0.151
```

3、自动生成密钥，并自动发送到指定的主机脚本

```bash
cat ssh_keygen.sh
#!/bin/bash
 
remote_host_ip=$1
remote_host_user=$2
remote_host_port=$3
remote_host_passwd=$4
local_rsa_file=~/.ssh/id_rsa
local_rsa_pub_file=~/.ssh/id_rsa.pub
  
[ $# -ne 4 ] && echo "Usage: sh $0 RemotehostIp RemotehostUser RemotehostPort RemotehostPasswd" && exit 5
 
[ ! -e ${local_rsa_file} ] && ssh-keygen -t rsa -P '' -f ${local_rsa_file} >/dev/null 2>&1
 
expect << EOF
set timeout 10
spawn ssh-copy-id -i ${local_rsa_pub_file} $remote_host_user@$remote_host_ip -p $remote_host_port
expect {
        "(yes/no)?" {send "yes\n";exp_continue}
        "password: " {send "$remote_host_passwd\n"}
}
expect eof
EOF
```

发送到更多的服务器上，调用上面此脚本，实现批量创建和分发密钥文件的功能

## 2、openssh配置文件

