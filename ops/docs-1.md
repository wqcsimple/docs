# 常规命令

## 登陆配置

### root登录配置

> 切换到 root 修改密码

```shell
sudo -i
passwd
```

> 配置ssh公钥

```shell
ssh-keygen
ssh-keygen -t rsa
ssh-keygen -t rsa -C "youremail@example.com"
```

> 复制`id_rsa.pub`内容到服务器`~/.ssh/authorized_keys`

```shell
mkdir ~/.ssh
vi ~/.ssh/authorized_keys
```

> 新创建的authorized_keys文件需配置权限

```shell
chmod 600 ~/.ssh/authorized_keys
```

> 允许 root 登陆及修改端口

```shell
systemctl stop firewalld
systemctl disable firewalld
# 关闭防火墙或放行修改后的 SSH 端口
vi /etc/ssh/sshd_config
Port 22222
PermitRootLogin yes
PasswordAuthentication yes
ClientAliveInterval 30
MaxSessions 100
# 重启 sshd 生效
systemctl restart sshd
```

> 禁用其他账户登陆

```shell
vi /etc/passwd
# 注释不允许登陆的账号
```

> 配置防火墙

- 国内服务器 默认关闭系统防火墙,由控制台管理.
- 国外服务器 视情况开启配置防火墙.

> 启用防火墙

```shell
systemctl status firewalld.service
# 查看防火墙状态
systemctl start firewalld.service
# 启用防火墙
systemctl enable firewalld
# 设置开机启动
systemctl stop firewalld.service
# 关闭防火墙,根据实际需求来
sudo firewall-cmd --list-all
# 查看防火墙列表
firewall-cmd --query-port=666/tcp
# 查看指定端口是否已打开
```

> 添加防火墙

```shell
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --add-port=443/tcp --permanent
sudo firewall-cmd --add-port=4000-50000/tcp --permanent
# 开永久端口号
sudo firewall-cmd --remove-port=XXXX/tcp --permanent
# 移除端口
sudo firewall-cmd --remove-service=XXXX --permanent
sudo firewall-cmd --reload
# 重新载入配置 比如添加规则之后，需要执行此命令
sudo firewall-cmd --list-all
```

> 修改主机名

```shell
hostnamectl set-hostname xxx
```

> 其他

```shell
uname -a
# 查看服务器系统
vi /etc/resolv.conf
# 修改DNS 加上4.4.4.4
```

> `-bash: ifconfig: command not found`遇到该问题，`yum install net-tools` 后执行 `ifconfig`

## 软件配置

### yum repo

```shell
yum repolist
yum repolist all
# 查看 yum 源
yum -y install epel-release
# 如无 epel/x86_64 推荐先安装 epel 源
yum clean all
yum makecache
# 清除并重建缓存
yum update -y
# 更新
# 如需使用 epel 源国内阿里云镜像加速,执行以下命令:
mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum clean all
yum makecache
```

### yum install

```shell
yum -y install wget zip unzip tar git screen vnstat telnet nethogs net-tools iptables-services bash-completion
```

### swap

```shell
free -m
# 检查 swap 状态
wget https://files.ioiox.com/projects/swap/swap.sh && chmod +x swap.sh && ./swap.sh
# 或
wget https://files.ioiox.com/projects/swap/gceswap.sh && chmod +x gceswap.sh && ./gceswap.sh
# gce swap
vi /etc/fstab
# 检查 swap 挂载
```

### sendMail

```shell
# 安装sendmail
yum install -y sendmail*
rpm -aq | grep sendmail

# 安装mailx
yum install -y mailx

# 发送测试
echo "this is my test mail" | mail -s 'mail test' xxx@xxx.com
# 文本测试
mail -s 'mail test' xxx@xxx.com < xxx.txt
# 文件测试

# 配置 smtp
vi /etc/mail.rc
# 添加以下配置,注意 smtp 可能因为云服务商之间限制,如无法发送,尝试加上80,465端口,再次测试.
set from=xxx@xxx.com
set smtp=smtp.xxx.com
set smtp-auth-user=username
set smtp-auth-password=password
set smtp-auth=login

# 配置 root 登陆邮件提醒
vi ~/.bash_profile
# 添加以下模板
IP="$(echo $SSH_CONNECTION | cut -d " " -f 1)"
HOSTNAME=$(hostname)
NOW=$(date +"%e %b %Y, %a %r")
echo 'Someone from '$IP' logged into '$HOSTNAME' on '$NOW'.' | mail -s 'SSH Login Notification' YOUR_EMAIL_ADDRESS
# YOUR_EMAIL_ADDRESS 修改为接收通知的邮箱地址

# 关闭您在 /var/spool/mail/root 中有新邮件提示
echo "unset MAILCHECK">> /etc/profile
```

### Docker

```shell
# 安装 docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
# 国内镜像
sudo sh get-docker.sh --mirror Aliyun
# 启动
sudo systemctl start docker
sudo docker version
sudo systemctl enable docker
# docker-compose
# 国外
curl -L https://github.com/docker/compose/releases/download/1.27.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
# 国内加速
curl -L https://get.daocloud.io/docker/compose/releases/download/1.28.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
    
docker-compose version
```
