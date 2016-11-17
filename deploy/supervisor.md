# Supervisor

[Supervisor](http://supervisord.org/) 是一个在类 UNIX 系统上管理进程的工具，它能让进程在后台执行，控制并发，进程崩溃时自动重启，捕获进程的标准输出等

尤其适合那些没有以守护模式运行功能的进程

它由 Python 编写，但可以用于任何应用

##　安装

Supervisor 3.x 只支持 Python 2，4.0 计划支持 Python 3。

```
sudo pip2 install supervisor
```

EPEL 源虽有 supervisor, 但版本较旧，不建议使用。

## 配置

需手动创建配置文件目录、日志目录：

```
sudo mkdir -p /etc/supervisor/programs
sudo mkdir /var/log/supervisor/
sudo chown $USER: /var/log/supervisor
```

`/etc/supervisor/supervisord.conf`:

```
[unix_http_server]
file = /var/log/supervisor/supervisor.sock

[supervisord]
pidfile=/var/log/supervisor/supervisord.pid
nodaemon=false
logfile=/var/log/supervisor/supervisord.log
logfile_maxbytes=50MB
logfile_backups=10
loglevel=info

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl = unix:///var/log/supervisor/supervisor.sock

[include]
files = programs/*.ini
```

各个项目的配置文件放在 `/etc/supervisor/programs/` 目录下，每个项目一个配置文件，以便管理。

## Systemd service

创建 systemd service 以便管理：

`/usr/lib/systemd/system/supervisord.service`:

```
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
# 用户名按实际情况修改
User=username
PIDFile=/var/log/supervisor/supervisord.pid
ExecStart=/usr/bin/supervisord
ExecStop=/usr/bin/supervisorctl shutdown
ExecReload=/usr/bin/supervisorctl reload
KillMode=process
Restart=on-failure
RestartSec=3s

[Install]
WantedBy=multi-user.target
```

使用 `systemctl` 管理 supervisord 进程：

```
# 添加/修改 service 文件后需执行以使生效
sudo systemctl daemon-reload

# 启动/停止/重启/重载配置/查看状态
sudo systemctl start/stop/restart/reload/status supervisord

# 设置/取消自动启动
sudo systemctl enable/disable supervisord
```

## 常用命令

```
# 查看所有进程状态
supervisorctl status

# 查看一个或多个进程状态
supervisorctl status [programe-name]*

# 重新加载配置
supervisorctl reload

# 启动/停止/重启一个或多个进程
supervisorctl start/stop/restart [program-name]*

# 启动/停止/重启所有进程
supervisorctl start/stop/restart all
```

执行 `supervisorctl help` 以了解更多。

## 添加应用

1. 在 `/etc/supervisor/programs/` 目录下创建配置文件
2. 执行 `sudo systemctl reload supervisord` 重载配置
3. 执行 `sudo supervisorctl start [programe-name]` 启动应用。若未禁止自动启动，则不需执行

配置文件示例：

```
[program:girls-day-carve]
directory=/data/app/girls-day-carve
command=python3.5 server.py --unix-socket=log/tornado.sock --subpath=/girls-day-carve
process_name=%(program_name)s_%(process_num)01d
numprocs=1
stdout_logfile=/data/app/girls-day-carve/log/stdout.log
stderr_logfile=/data/app/girls-day-carve/log/stderr.log
```

__日志文件路径须是绝对路径__

配置语法参考[官方文档](http://supervisord.org/configuration.html#program-x-section-settings)

## 参考资料

* [官方网站](http://supervisord.org/)
* [配置文档](http://supervisord.org/configuration.html)
* [自动启动](http://supervisord.org/running.html#running-supervisord-automatically-on-startup)
