# 部署 Rails 应用

## 环境需求

* Ruby
* Gem packages: bundler
* Node.js, npm
* gcc, g++, make 等工具，编译 gem 扩展需要
* mariadb-devel（编译 [mysql2](https://github.com/brianmario/mysql2) gem 包需要）

部分非必须

## 部署工具

使用 [mina](https://github.com/mina-deploy/mina) 部署，相比 [capistrano](https://github.com/capistrano/capistrano)，mina 更轻量级，速度更快

多个 stage 的配置方法：

```
desc 'Production server'
task :production do
  set :domain, 'server'
  set :user, 'user'
  set :branch, 'master'
end

desc 'Development server'
task :development do
  set :domain, 'server'
  set :user, 'user'
  set :branch, 'master'
end
```

使用时在命令前加上 stage 名称，如

```
bundle exec mina production deploy
```

SSH 配置：

1. 本地须有 ssh 连接到部署服务器的权限
2. 若部署服务器有访问远程仓库的权限，设置 `set :forward_agent, false`
3. 若部署服务器有访问远程仓库的权限，本地须有访问仓库的权限，且设置 `set :forward_agent, true`

## 应用服务器

* 若同一服务器上部署多个 Rails 应用，使用 [passenger](https://www.phusionpassenger.com/)，以减少资源占用

* 若部署的应用不多，可使用 [unicorn](http://unicorn.bogomips.org/) 或 [puma](http://puma.io/)

## 日志轮滚

为避免日志文件过大，影响性能，应定期切割日志文件

使用系统自带功能 logrotate 实现

创建 `/etc/logrotate.d/rails` 文件，内容如下：

```
/data/app/*/shared/log/*.log {
  su user group
  weekly
  size 100M
  missingok
  rotate 10
  compress
  delaycompress
  notifempty
  copytruncate
}
```

修改其中的 `user group` 为日志文件所属的用户和组，按需修改

## 开机自动启动

Passenger 可随 nginx 自动启动，但 unicorn, puma, sidekiq 等都需要手动启动
