# 部署 Rails 应用

## 环境需求

* Ruby
* Gem packages: bundler
* Node.js, npm
* gcc, g++, make 等工具，编译 gem 扩展需要
* mariadb-devel（编译 [mysql2](https://github.com/brianmario/mysql2) gem 需要）
* postgresql-libs postgresql-devel（编译 [pg](https://rubygems.org/gems/pg/) gem 需要）

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
3. 若部署服务器没有访问远程仓库的权限，本地须有访问仓库的权限，且设置 `set :forward_agent, true`

## 应用服务器

* 若同一服务器上部署多个应用，或应用的访问量不大、高峰期很短，使用 [passenger](https://www.phusionpassenger.com/)，以减少资源占用

* 若部署的应用不多，或应用较重要，可使用 [unicorn](http://unicorn.bogomips.org/) 或 [puma](http://puma.io/)

* 若应用的响应较慢，避免使用 unicorn（只有进程模式，一个请求就要占用一个进程，响应较慢会导致请求排队时间过长）

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

Passenger 可随 nginx 自动启动

unicorn, puma, sidekiq 等由于以下特点：

1. 每个应用都需单独运行自己的实例，不像 passenger, php-fpm 那样有一个统一的调度进程同时服务多个应用
2. 一个系统上可能会同时部署多个应用，且应用的部署目录不固定
3. 这些服务通常由部署工具的插件（如 mina-unicorn，capistrano-puma）负责启动，而部署工具都是在本地运行的，在服务器上无法使用
4. 虽然那些插件本质上也是在服务器上执行这些服务各自的命令行工具（`bundle exec unicorn`, `bundle exec pumactl` 等）启动服务的，理论上说可以自己在服务器上手动执行，但这些插件一般会设置一些默认的启动参数，若手动启动时与这些插件的参数不同，可能会有意想不到的问题

所以，这些服务难以通过 systemd 等进程管理工具管理，需要手动启动。虽然以上问题都可以解决，但若要考虑到可维护性、扩展性就不那么简单了。
