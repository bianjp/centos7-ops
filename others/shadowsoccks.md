# Shadowsocks

## 安装

需先安装 Python 和 pip, 2.x 或 3.x 版本均可

```bash
sudo pip3 install shadowsocks
```

## 配置

创建 `/etc/shadowsocks/config.json`:

```
{
    "server": "0.0.0.0",
    "server_port": 8388,
    "password": "Your password",
    "timeout": 300,
    "method": "aes-256-cfb",
    "fast_open": true,
    "workers": 2
}
```

酌情修改配置

## 启动

```bash
ssserver -c /etc/shadowsocks/config.json
```

若 `server_port` 小于 1024，需要以 root 权限启动

## 自动启动

创建 `/etc/systemd/system/shadowsocks.service`:

```
[Unit]
Description=Shadowsocks Server Service
After=network.target

[Service]
# Require root if server_port < 1024
User=root
ExecStart=/usr/bin/ssserver -q -c /etc/shadowsocks/config.json
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable shadowsocks
sudo systemctl start shadowsocks
```
