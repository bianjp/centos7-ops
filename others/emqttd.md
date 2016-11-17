# Emqttd

基于高并发的 Erlang 语言和 OTP 平台设计，支持百万级连接和分布式集群，发布订阅模式的开源 MQTT 消息服务器

## 安装

目前无 RPM 包可用，只能从[官网下载](http://emqtt.io/downloads)压缩包，解压缩后即可直接运行

```
# 下载、解压缩
cd /data/app
curl -Lo emqttd.zip http://emqtt.io/downloads/latest/centos
unzip emqttd.zip
cd emqttd/

# 查看命令帮助
./bin/emqttd --help
./bin/emqttd_ctl --help
```

## 配置

### systemd service

创建 systemd service 以便管理

```
sudo tee /etc/systemd/system/emqttd.service <<-'EOF'
[Unit]
Description=Emqttd server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Environment="HOME=/data/app/emqttd"
Type=forking
GuessMainPID=true
ExecStartPre=/data/app/emqttd/bin/emqttd chkconfig
ExecStart=/data/app/emqttd/bin/emqttd start
ExecStop=/data/app/emqttd/bin/emqttd stop

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
```

管理命令：

```
# 查看状态/启动/停止/重启
sudo systemctl status/start/stop/restart emqttd
# 启用/禁用自动启动
sudo systemctl enable/disable emqttd
```

### 防火墙

emqttd 默认使用以下端口：

* 1883: MQTT Port
* 8883:  MQTT(SSL) Port
* 8083:  MQTT(WebSocket), HTTP API Port
* 18083: Web Dashboard

需配置防火墙以允许外网连接这些端口：

```
sudo tee /etc/firewalld/services/emqtt.xml <<-'EOF'
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Emqtt</short>
  <description>The massively scalable MQTT broker</description>
  <port protocol="tcp" port="1883"/>
  <port protocol="tcp" port="8803"/>
  <port protocol="tcp" port="8083"/>
  <port protocol="tcp" port="18083"/>
</service>
EOF

sudo firewall-cmd --reload
sudo firewall-cmd --permanent --zone=dmz --add-service=emqtt
sudo firewall-cmd --zone=dmz --add-service=emqtt
```

若使用了非默认端口，需相应修改 `/etc/firewalld/services/emqtt.xml`

### 认证

安全起见，要对客户端（包括发布者、订阅者）进行认证

emqttd 支持 ClientID、用户名/密码、IP地址、HTTP Cookie 等多种认证方式

我们使用用户名/密码认证方式，并使用 [ACL](https://github.com/emqtt/emqttd/wiki/ACL-Design) 控制权限（发布、订阅）

不同类型的客户端使用不同的账号，分配尽量少的权限（操作，topic），以减少安全隐患

1. 添加用户名/密码

    修改 `etc/emqttd.config` 中的 emqttd -> access -> auth：

    ```
    {username, [{"username", "password"}, {"username2", "password2"}]},
    ```

2. 控制权限

    修改 `etc/acl.config`：

    ```
    {allow, all}.

    {deny, {user, "subscriber"}, publish, ["$SYS/#", "#"]}.
    ```

    __ACL 中的用户名必须存在于 `etc/emqttd.config` 中，否则 emqttd 启动失败__

### Web 管理

插件 [emqttd_dashboard](https://github.com/emqtt/emqttd_dashboard) 提供了一个 Web 管理界面，访问地址为 http://server_addr:10803

默认账号为：

* 用户名：admin
* 密码：public

安装后应立即修改默认密码

## 参考资料

* [emqtt 官网](http://emqtt.io)
* [emqtt 中文官网](http://emqtt.com)
* [官方文档](http://emqtt.io/docs/index.html)
* [官方 Wiki](https://github.com/emqtt/emqttd/wiki)
