# Shadowsocks

Python 版实现已不再更新，建议使用 [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)

## 安装

```base
sudo curl -o /etc/yum.repos.d/shadowsocks.repo https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo
sudo yum install shadowsocks-libev
```

## 配置

创建 `/etc/shadowsocks/config.json`:

```
sudo mkdir /etc/shadowsocks
sudo tee /etc/shadowsocks/config.json <<EOF
{
    "server": "0.0.0.0",
    "server_port": 8388,
    "password": "Your password",
    "timeout": 300,
    "method": "aes-256-cfb",
    "fast_open": true,
    "workers": 2
}
EOF
```

酌情修改配置

若使用了防火墙，应配置防火墙允许所用端口的流量（TCP + UDP）通过

## 启动

```bash
ss-server -c /etc/shadowsocks/config.json
```

若 `server_port` 小于 1024，需要以 root 权限启动

## 自动启动

自带的 `shadowsocks-libev.service` 不太好用，创建自定义的 `/etc/systemd/system/shadowsocks.service`:

```
[Unit]
Description=Shadowsocks Server Service
Documentation=man:shadowsocks-libev(8)
After=network.target

[Service]
# Require root if server_port < 1024
User=root
LimitNOFILE=32768
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
Restart=always
ExecStart=/usr/bin/ss-server -u -c /etc/shadowsocks/config.json

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable shadowsocks
sudo systemctl start shadowsocks
```

## 参考资料

* [Shadowsocks](https://shadowsocks.org/)
* [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)
* [Optimizing Shadowsocks](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks)
