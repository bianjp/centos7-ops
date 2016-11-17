# PHP

使用 Nginx + PHP-FPM 部署 PHP 应用

## 安装

```
# 官方源中 php 版本过旧，使用 IUS 源
sudo rpm -Uvh https://centos7.iuscommunity.org/ius-release.rpm
sudo yum update

sudo yum install php70u-cli php70u-fpm php70u-json php70u-gd php70u-mbstring php70u-mcrypt php70u-opcache php70u-mysqlnd

sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

## 配置

`/etc/php.ini`:

```
# POST 请求体大小、上传文件大小。按应用需求修改
post_max_size = 8M
upload_max_filesize = 8M

# 避免 nginx + php-fpm 的安全问题
cgi.fix_pathinfo = 0

# 时区
date.timezone = Asia/Shanghai

# 是否自动开启 session
session.auto_start = 0
```

`/etc/php-fpm.d/www.conf`:

```
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
user = nginx
group = nginx

; 进程管理策略。根据内存大小、负载情况调整
pm = dynamic
pm.max_children = 200
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35

# 检查负载情况与可用性的路径。注意避免与应用冲突
# 若开启，注意限制访问权限以减少风险
#pm.status_path = /status
#ping.path = /ping

slowlog = /var/log/php-fpm/www-slow.log
; 如需开启访问日志，取消注释下一行。若非必要不要开启
;access.log = /var/log/php-fpm/$pool.access.log
access.format = "%t \"%m %r%Q%q\" %s %{mili}d %{kilo}M %C%%"

# 注释下一行，或创建该目录并设置用户和组为 nginx。因为 php-fpm 不会自动创建该目录，导致 session 无法保存
;php_value[session.save_path] = /var/lib/php/session
```

检查配置是否正确：

```
sudo php-fpm -t
```
