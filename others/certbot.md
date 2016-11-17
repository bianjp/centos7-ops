# Certbot

Certbot 用于获取 [Let's Encrypt](https://letsencrypt.org/) SSL 证书

## 安装

```
sudo yum install certbot
```

## 配置

```
sudo mkdir -p /etc/letsencrypt

sudo tee /etc/letsencrypt/cli.ini <<EOF
# 2048或4096。越长越安全，但加解密时也会消耗更多资源
# rsa-key-size = 4096

# 同意服务条款（Terms of Service），避免让用户确认是否同意
agree-tos = True

# 使用文本模式（相对于图形模式）
text = True

# 已经申请过证书的域名默认进行续期操作
# 若开启，则 renew 总是会获取新证书，不管旧证书是否即将过期
#renew-by-default = True

# 邮箱，用于接收通知（如续期提醒）
email = admin@example.com

# 验证域名所用方式
authenticator = webroot
EOF
```

## 命令

```
# 测试获取证书
sudo certbot certonly --dry-run -d example.com --webroot-path /data/app/example.com

# 获取证书
sudo certbot certonly -d example.com --webroot-path /data/app/example.com

# 续期证书
sudo certbot renew

# 吊销证书
sudo certbot revoke --cert-path /etc/letsencrypt/archive/example.com/cert1.pem
```

## Nginx 配置

```
ssl_certificate         /etc/letsencrypt/live/example.com/fullchain.pem;
ssl_certificate_key     /etc/letsencrypt/live/example.com/privkey.pem;
ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
```

## 定时任务

添加定时自动续期，默认会自动续期所有快到期的证书

自动续期的配置在 `/etc/letsencrypt/renewal/` 目录

续期成功后需重载 Nginx 以使用新证书

crontab 配置如下：

```
# Minute Hour Day Month Day_of_week Command
    0     4    *    *        1      /bin/certbot renew --quiet --renew-hook "/bin/systemctl reload nginx"
```

## 参考资料

* https://letsencrypt.org/
* https://certbot.eff.org/docs/using.html
* https://github.com/certbot/certbot
