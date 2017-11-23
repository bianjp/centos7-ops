# Nginx

使用 Nginx 作为 Web server

## 安装

__若需使用 Passenger，请使用 [Passenger 文档](../deploy/passenger.md)中的方式安装 Nginx__

EPEL 源中版本过旧，使用 Nginx 官方源

```
sudo tee /etc/yum.repos.d/nginx.repo <<-'EOF'
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
EOF

sudo yum makecache
sudo yum install -y nginx

sudo systemctl enable nginx
sudo systemctl start nginx
```

## 配置

修改配置文件 `/etc/nginx/nginx.conf`:

```
user nginx;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log warn;

worker_rlimit_nofile 65535;
events {
    worker_connections 65535;
}

http {
    include       /etc/nginx/mime.types;
    include       /etc/nginx/conf.d/*.conf;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr [$time_local] "$request" '
                      '$status $body_bytes_sent $request_time "$http_referer" '
                      '"$http_user_agent" $http_x_forwarded_for';

    access_log  /var/log/nginx/access.log main;

    # 域名较多时需设置
    server_names_hash_bucket_size 64;

    # 不把 404 请求记录到错误日志中
    log_not_found       off;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    # 限制了上传文件大小。根据应用需求相应修改
    client_max_body_size 10m;

    # 启用 gzip
    gzip              on;
    gzip_min_length   1k;
    gzip_comp_level   5;
    gzip_http_version 1.0;
    gzip_types        text/plain application/x-javascript application/javascript text/css text/xml application/xml application/json;
    gzip_proxied      any;
    gzip_vary         on;
    gzip_disable      msie6;

    # 须放在最后，否则上面的配置可能无效
    include       /etc/nginx/sites/*.conf;
}
```

为便于管理，每个 server 的配置保存为一个文件，放在 `/etc/nginx/sites/` 目录下

```
sudo mkdir /etc/nginx/sites
sudo touch /etc/nginx/sites/default.conf
```

`/etc/nginx/sites/default.conf`:

```
server {
    listen       80 default_server;
    server_name  _;
    return 404;
}
```

Nginx 的 logratete 配置文件为 `/etc/logrotate.d/nginx`，默认每日一次，为避免过于频繁，建议修改频率，或加入文件大小限制：

```
minsize 300M
```

## HTTPS

```
# 默认是 1024 位，不够安全
sudo openssl dhparam -out /etc/ssl/dhparam.pem 2048

sudo mkdir -p /etc/nginx/includes

# 添加配置文件
sudo tee /etc/nginx/includes/https.conf <<-'EOF'
# certs
#ssl_certificate /path/to/fullchain.pem;
#ssl_certificate_key /path/to/private_key;
#ssl_trusted_certificate /path/to/chain.pem;

ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;

# Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
ssl_dhparam /etc/ssl/dhparam.pem;

# modern configuration. tweak to your needs.
ssl_protocols TLSv1.2;
ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
ssl_prefer_server_ciphers on;

# OCSP Stapling ---
# fetch OCSP records from URL in ssl_certificate and cache them
ssl_stapling on;
ssl_stapling_verify on;

# name servers used to resolve names of upstream servers into addresses
# 国内使用 DNSPod Public DNS, 阿里 DNS
resolver 119.29.29.29 182.254.116.116 223.5.5.5 223.6.6.6;
# resolver 8.8.8.8 8.8.4.4;
EOF

sudo tee /etc/nginx/includes/https-hsts.conf <<-'EOF'
include /etc/nginx/includes/https.conf;

# HSTS
# 15768000 seconds = 6 months
add_header Strict-Transport-Security max-age=15768000;
EOF
```

网站配置示例：

```
server {
    listen      80;
    server_name example.com;
    return 301  https://example.com$request_uri;

    access_log /var/log/nginx/example-http.access.log main;
    error_log  /var/log/nginx/example-http.error.log warn;
}

server {
    listen      443 ssl http2;
    server_name example.com;
    root        /data/app/example;

    include /etc/nginx/includes/https.conf;
    # if want to enable hsts
    # include /etc/nginx/includes/https-hsts.conf;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

    access_log /var/log/nginx/example.access.log main;
    error_log  /var/log/nginx/example.error.log warn;
}
```

## 参考资料

* [Nginx Linux packages](http://nginx.org/en/linux_packages.html)
* [Mozilla SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/)
