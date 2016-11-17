# 服务器基本配置

以下配置过程需使用 root 用户执行

## 基本配置

### 主机名称

```
echo example.com > /etc/hostname
```

### 语言

语言不正确可能会导致无法正常显示中文

```
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

### 时区

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 常用软件、软件源

```
yum update -y
yum group install -y development
yum install -y vim bash-completion git mailx deltarpm bind-utils yum-utils
yum install -y epel-release
```

若默认的软件源（阿里云、腾讯云默认都是使用他们自己维护的软件源）中软件版本较旧，可替换为中科大的源：[https://lug.ustc.edu.cn/wiki/mirrors/help/centos]()

## sshd

sshd 配置文件为`/etc/ssh/sshd_config`

```
# 禁止root登录
PermitRootLogin no

# 禁止密码登录
PasswordAuthentication no

# 忽略权限不正确的配置文件
StrictModes yes

# 避免长时间无操作后断线
ClientAliveInterval 10
ClientAliveCountMax 5
```

重启 sshd 使配置生效

```
systemctl restart sshd
```

## 防火墙

```
# 有些云服务商提供的 CentOS 镜像可能默认没有安装
yum install -y firewalld

# 检查是否正在运行
systemctl status firewalld

# 若未运行，运行并自启动
systemctl start firewalld
systemctl enable firewalld

# 查看默认的zone
firewall-cmd --get-default-zone

# 查看当前使用的zone
firewall-cmd --get-active-zones

# 设置默认使用的zone
firewall-cmd --set-default-zone=dmz

# 允许对外开放的服务。执行 ls /usr/lib/firewalld/services/ 以查看支持的服务列表
firewall-cmd --permanent --zone=dmz --add-service=http
firewall-cmd --permanent --zone=dmz --add-service=https
firewall-cmd --permanent --zone=dmz --add-service=ssh

# 允许对外开放的端口
# firewall-cmd --permanent --zone=dmz --add-port=3306/tcp

# 重载配置以生效
firewall-cmd --reload

# 查看配置
firewall-cmd --zone=dmz --list-all
```

__注意：__

若修改了所添加service（如ssh）的端口号，需修改 `/usr/lib/firewalld/services/` 目录下相应配置文件中的端口号，并在修改后执行 `firewall-cmd --reload` 重载配置

## SELinux

[SELinux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux) 是一种安全机制，但容易导致权限问题，最好禁用

```
# 查看状态
sestatus
```

修改配置文件 `/etc/sysconfig/selinux`：

```
SELINUX=disabled
```

重启系统以生效

## Ulimit

Ulimit (User Limits) 对用户能够使用的系统资源（如最大文件描述符数量）进行限制

若默认值较小，易导致 "Too many open files" 等错误

在 `/etc/security/limits.conf` 中加入：

```
* soft nofile 65535
* hard nofile 65535
```

修改后重启系统

## Yum 配置

修改 `/etc/yum.conf`:

```
# 缓存安装包
keepcache=1

# 内核数量
# 默认会保留 5 个旧内核，为减少空间占用，可改为两个
installonly_limit=2
```

立即删除旧内核：

```
yum install yum-utils
package-cleanup --oldkernels --count=2
```

## 数据存储

为便于管理、扩展，所有数据保存到 `/data` 目录

```
mkdir /data
```

若有额外的数据盘，将其挂载到 /data 目录

```
# 格式化硬盘。可不分区
mkfs.ext4 /dev/vdb

# 挂载
mount /dev/vdb /data
```

编辑 `/etc/fstab`，配置开机自动挂载：

```
/dev/vdb   /data   ext4   defaults   0   0
```

按需要创建子目录

```
# web 服务器根目录
mkdir /data/www

# 应用（如 rails 应用）目录
mkdir /data/app
```

__注意子目录的用户/组。__ Web 服务器根目录的用户/组应为 web 服务器运行时的用户/组

## 交换空间

若内存较大，可不设置交换空间

```
# 查看
swapon -s
free -m

# 创建交换空间文件
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile

# 启用
swapon /swapfile

# 禁用
swapoff /swapfile
```

编辑 `/etc/fstab`，以开机自动挂载交换空间：

```
/swapfile   swap   swap   sw   0   0
```

## logrotate

logrotate 用于日志文件的自动切割，避免日志文件过大而影响应用性能

其基本原理是通过 crontab 配置定时任务，每日执行一次，执行时读取各应用的配置文件，根据配置进行日志文件的切割、应用的重启等

各应用的配置文件在 `/etc/logrotate.d/` 目录下，可按需修改

## 用户

出于安全考虑，新增一个非 root 用户用于日常管理

```
useradd -m -G wheel username
passwd username
```

切换到新添加的用户，配置密钥, .bashrc, .vimrc, .gitconfig 等

注意，`.ssh` 权限需为 700，`.ssh/authorized_keys` 权限需为 600，否则无效
