# git配置

## 1、配置用户名称和登录邮箱

```bash
git config --global user.name "name"
git config --global user.email "email"
```

## 2、生成SSH公钥

命令执行完后，一直回车

```bash
ssh-keygen -t rsa -C "emile.emile.com"
```

### 3、查看复制SSH密钥

```bash
cd ~./ssh/
cat id_rsa.pub
```

​	