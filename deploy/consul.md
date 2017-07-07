# Consul

## 安装

```
version=0.18.5
curl -O https://releases.hashicorp.com/consul-template/$version/consul-template_$version_linux_amd64.tgz
tar xf consul-template_$version_linux_amd64.tgz
sudo cp consul-template /opt/
```

## 配置

使用 systemd 管理 consul template 进程

创建 `/etc/systemd/system/consul-template.service`，内容如下（按需修改）：

```
[Unit]
Description=Consul template
After=syslog.target network.target

[Service]
Type=simple
EnvironmentFile=/app/shop/deploy/consul-template-env
ExecStart=/opt/consul-template -consul-addr=${CONSUL_ADDR} -consul-token=${CONSUL_TOKEN} -template "${TEMPLATE}:${NGINX_CONF}:systemctl reload nginx"
ExecStop=/usr/bin/kill -INT $MAINPID
ExecReload=/usr/bin/kill -HUP $MAINPID
SuccessExitStatus=12
Restart=always

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl start consul-template
sudo systemctl enable consul-template
```

__注意：修改模板后需重载 consul template 才能使新模板生效：`sudo systemctl reload nginx consul-template`__


## 参考资料

* https://github.com/hashicorp/consul-template
* 