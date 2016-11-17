# Nexus Repository Manager OSS

[Nexus Repository Manager OSS](http://www.sonatype.com/nexus-repository-oss) 用于搭建私有源、镜像代理，支持 maven, docker, bower, npm 等

## 安装 Java Runtime

Nexus 依赖 Java 运行时环境，官方只支持 Oracle Java

但 openjdk 也应能正常工作：

```
sudo yum install java-1.8.0-openjdk
```

## 安装

没有 rpm 包可用，只能手动下载安装

```
curl -LO http://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar vxf latest-unix.tar.gz
sudo mv nexus-* /opt
sudo ln -s /opt/nexus-*/ /opt/nexus

# 创建单独的用户作为运行时用户，避免安全隐患
sudo useradd -M -c "Nexus repository manager" -s /sbin/nologin nexus
sudo chown -R nexus: /opt/nexus/

# 添加 systemd 管理服务
sudo tee /etc/systemd/system/nexus.service <<-'EOF'
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
ExecReload=/opt/nexus/bin/nexus force-reload
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
```

## 配置

配置文件保存在安装目录的 `etc`, `bin` 子目录下

### 监听地址

Nexus 默认监听所有 interface 的 8081 端口

生产环境下使用 Nginx 作为前端，反向代理到 nexus，因此 nexus 只需监听本地即可

`/opt/nexus/etc/org.sonatype.nexus.cfg`:

```
application-host=127.0.0.1

# 若部署在子路径，此处设置子路径
# nexus-context-path=/nexus/
```

### 数据目录

数据目录默认为安装目录下的 `data` 目录，改为其它目录，以便升级 nexus

`/opt/nexus/bin/nexus.vmoptions`:

```
-Dkaraf.data=/data/nexus
-Djava.io.tmpdir=/data/nexus/tmp
```

创建数据目录：

```
sudo mkdir /data/nexus
sudo chown nexus: /data/nexus
```

### SEO

禁止搜索引擎索引，以减少被发现、滥用的风险

`/opt/nexus/public/robots.txt`:

```
User-agent: *
Disallow: /
```

### Nginx

```
server {
    listen      80;
    server_name nexus.example.com;
    return 301 https://$host$request_uri;

    access_log /var/log/nginx/nexus.access.log main;
    error_log  /var/log/nginx/nexus.error.log warn;
}

server {
    listen       443 http2 ssl;
    server_name  nexus.example.com;

    ssl_certificate     /etc/letsencrypt/live/nexus.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/nexus.example.com/privkey.pem;

    proxy_send_timeout 120;
    proxy_read_timeout 300;
    proxy_buffering    off;
    keepalive_timeout  55;
    tcp_nodelay        on;

    # allow large uploads of files
    client_max_body_size 1G;

    # optimize downloading files larger than 1G
    proxy_max_temp_file_size 2G;

    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # 若未使用 https，注释该配置
        proxy_set_header X-Forwarded-Proto "https";
    }

    access_log /var/log/nginx/nexus.access.log main;
    error_log  /var/log/nginx/nexus.error.log warn;
}
```

## 运行

使用 systemd 管理 nexus 运行状态：

```
# 查看状态/启动/停止/重启
sudo systemctl start/stop/restart/status nexus

# 启用/禁用自动启动
sudo systemctl enable/disable nexus
```

网页端默认的管理账号为：

* username: admin
* password: admin123

首次运行后应立即修改密码

## 升级

步骤：

1. 下载新版本并解压缩到 `/opt/` 目录下，修改其中的配置文件
2. 停止运行 nexus
3. 修改 `/opt/nexus` 符号链接，指向最新版本
4. 启动 nexus

由于配置文件保存在安装目录中，每次升级都须重新修改配置文件，比较麻烦，可使用以下脚本简化操作：

```shell
#!/usr/bin/bash
# 升级 nexus 版本
# 参数：新版本目录

set -e

# nexus 运行时的用户
user=nexus
# 数据目录
data_dir=/data/nexus

# 新版本文件所在目录
new_version_path=$1

if [[ -z "$new_version_path" ]]; then
  echo "请指定新版本文件所在目录"
  exit 1
elsif [[ ! -d "$new_version_path" ]];
  echo "$new_version_path 不是一个正确的目录"
  exit 1
fi

base_name=`basename $new_version_path`

if [[ -d "/opt/$base_name" ]]; then
  echo "/opt/$base_name 已存在"
  exit 1
fi

# 安装文件目录移动到 /opt
sudo mv $new_version_path /opt/
sudo chown -R $user: /opt/$base_name

# 修改配置文件。根据实际的配置情况按需修改
cd /opt/$base_name
sudo sed -i 's#^application-host=.*$#application-host=127.0.0.1#' etc/org.sonatype.nexus.cfg
sudo sed -i 's#^-Dkaraf.data=.*$#-Dkaraf.data=/data/nexus#' bin/nexus.vmoptions
sudo sed -i 's#^-Djava.io.tmpdir=.*$#-Djava.io.tmpdir=/data/nexus/tmp#' bin/nexus.vmoptions
sudo tee public/robots.txt <<-'EOF'
User-agent: *
Disallow: /
EOF

# 重启 nexus
sudo systemctl stop nexus
sudo -sf /opt/$base_name /opt/nexus
sudo systemctl start nexus
```

## 参考资料

* [Nexus official site](http://www.sonatype.com/nexus-repository-oss)
* [Download nexus](http://www.sonatype.com/download-oss-sonatype)
* [System requirements](https://support.sonatype.com/hc/en-us/articles/213464208-Sonatype-Nexus-System-Requirements)
* [Documentation](http://books.sonatype.com/nexus-book/3.0/reference/index.html)
* [Oracle Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
