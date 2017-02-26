# GitLab

为便于维护，使用 GitLab CE Omnibus package 安装方式

## 安装

### 安装依赖，配置环境

```
sudo yum install curl policycoreutils openssh-server openssh-clients
sudo systemctl enable sshd
sudo systemctl start sshd
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

若系统不支持 IPv6，须修改 `/etc/postfix/main.cf`

```
inet_interfaces = loopback-only
```

### 添加 GitLab 软件源

国内使用[清华大学源](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)：

```
sudo tee /etc/yum.repos.d/gitlab-ce.repo <<-'EOF'
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
EOF
```

国外使用 GitLab 官方源：

```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

### 安装 GitLab

```
sudo yum install gitlab-ce
```

## 配置

配置文件为 `/etc/gitlab/gitlab.rb`

```
# 访问地址。可以包含端口号、子路径
external_url 'http://git.example.com'

# 时区
gitlab_rails['time_zone'] = 'Asia/Shanghai'

# gitlab 主目录。不可更改！
# user['home'] = "/var/opt/gitlab"

# 仓库存储目录
# git_data_dir "/var/opt/gitlab/git-data"

# 备份文件存储目录
# gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"

# 备份文件保存时间（秒），避免备份文件过多而占用太多磁盘空间
gitlab_rails['backup_keep_time'] = 2592000 # 30 days

# 添加自定义 nginx 配置文件
# 目的是处理非域名请求，减轻外部攻击、扫描对服务器负载的影响
nginx['custom_nginx_config'] = "include /etc/gitlab/nginx-default.conf;"

# 日志文件轮滚频率。默认每天，太频繁
logging['logrotate_frequency'] = "monthly"
```

若使用 HTTPS，修改以下配置：

```
external_url "https://git.example.com"
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"
```

`/etc/gitlab/nginx-default.conf` 内容如下：

```
# GitLab >= 8.9 默认已配置，不能重复配置
# server_names_hash_bucket_size 64;

server {
    listen       80 default_server;
    server_name  _;
    return       404;

    access_log /var/log/gitlab/nginx/default_access.log;
    error_log  /var/log/gitlab/nginx/default_error.log;
}
```

更多配置，查看[配置文档][settings]

修改配置文件后需执行以下命令以使配置生效

```
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```

若修改了目录设置，可能需要手动迁移数据，具体操作查看相关文档

## 集成 Let's Encrypt SSL 证书

为避免影响运行中的 GitLab 服务，证书的申请和续期应使用 `webroot` 插件，但 GitLab 默认的 Nginx 配置中并未设置 `root`，所有请求都会转发给应用服务器，因此需额外配置：

1. 编辑 `/etc/gitlab/gitlab.rb`:

    ```
    nginx['custom_gitlab_server_config'] = "location ^~ /.well-known {\n    alias /srv/letsencrypt/.well-known;\n  }"
    ```

2. 创建 `/srv/letsencrypt/` 目录：

    ```
    sudo mkdir /srv/letsencrypt
    # 所有者改为 nginx 进程执行用户。用户名和组名查看 /var/opt/gitlab/nginx/conf/nginx.conf
    sudo chown gitlab-www:gitlab-www /srv/letsencrypt
    ```

Let's Encrypt 证书申请和续期时 `webroot-path` 设置为 `/srv/letsencrypt`

证书位置配置（`/etc/gitlab/gitlab.rb`）：

```
nginx['ssl_certificate'] = "/etc/letsencrypt/live/git.example.com/fullchain.pem"
nginx['ssl_certificate_key'] = "/etc/letsencrypt/live/git.example.com/privkey.pem"
```

使用 crontab 自动续期证书时，续期成功后需重启 Nginx：

```
# Minute Hour Day Month Day_of_week Command
    0     4    *    *        1      /bin/certbot renew --quiet --renew-hook "/bin/gitlab-ctl restart nginx"
```

Let's Encrypt 证书的申请和维护请查看 [Certbot 文档](../others/certbot.md)

## 命令行工具

```
# 重新配置/启动/查看状态/停止/重启
sudo gitlab-ctl reconfigure/start/status/stop/restart

# 启用/禁用自动启动。安装后默认已启用自动启动
sudo systemctl enable/disable gitlab-runsvdir

# 检查 gitlab 配置与环境
sudo gitlab-rake gitlab:check
```

## 升级

升级前需检查新版本及所有中间版本的[发布声明](https://about.gitlab.com/blog/archives.html)

```
sudo yum update gitlab-ce
```

## 备份和恢复

备份文件存储目录由配置文件中的 `gitlab_rails['backup_path']` 指定，默认为 `/var/opt/gitlab/backups`

备份文件名称为 `[TIMESTAMP]_gitlab_backup.tar`

### 备份

```
sudo gitlab-rake gitlab:backup:create
```

### 自动定期备份

使用 crontab 配置定时任务

切换到 root 用户，执行 `crontab -e`，添加以下内容（每周六凌晨4点备份）：

```
# Minute Hour Day Month Day_of_week Command
    0     4    *    *        6      /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1

# 将备份文件同步到数据盘
#  30     4    *    *        6      /bin/rsync -tpq /var/opt/gitlab/backups/* /data/gitlab-backup/
```

最好将备份文件自动同步到其它介质（如云存储）上

### 恢复

备份文件须在备份文件存储目录中

```
# 停止使用数据库的进程
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq

# 恢复。若有多个备份文件，需指定 timestamp
sudo gitlab-rake gitlab:backup:restore [BACKUP=timestamp_of_backup]

# 启动
sudo gitlab-ctl start

# 检查
sudo gitlab-rake gitlab:check SANITIZE=true
```

__注意：恢复时 GitLab 版本必须与备份时版本完全相同（x.x.x三段均相同）__

## 迁移

迁移到其它服务器上时，要迁移 `/etc/gitlab/gitlab-secrets.json`，否则 GitLab-CI Variables 的解密会失败，导致 Project Settings -> Variables 页面无法打开、自动构建失败

## 参考资料

* [Installation][download]
* [Documation](https://gitlab.com/gitlab-org/omnibus-gitlab/tree/master/doc/)
* [Settings][settings]
* [Backup and Restore][backup_restore]
* [Nginx Settings](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md)
* [清华大学 gitlab-ce 源](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)

[download]: https://about.gitlab.com/downloads/
[backup_restore]: https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/raketasks/backup_restore.md
[settings]: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/
