# Passenger

## 安装

```
sudo yum install -y epel-release yum-utils
sudo yum install -y pygpgme curl

sudo curl --fail -sSLo /etc/yum.repos.d/passenger.repo https://oss-binaries.phusionpassenger.com/yum/definitions/el-passenger.repo
sudo yum install nginx passenger
```

修改 `/etc/nginx/conf.d/passenger.conf`:

```
passenger_root /usr/share/ruby/vendor_ruby/phusion_passenger/locations.ini;
# 使用 RVM/rbenv/系统 默认的 ruby 版本。username 根据实际情况修改
passenger_ruby /home/username/.rvm/wrappers/default/ruby;
# passenger_ruby /home/username/.rbenv/shims/ruby;
# passenger_ruby /usr/bin/ruby;
passenger_instance_registry_dir /var/run/passenger-instreg;
```

## Nginx 配置

单独域名：

```
server {
    listen      80;
    server_name example.com;
    root        /data/app/your-app/current/public;

    passenger_enabled on;

    access_log  /var/log/nginx/your-app.access.log main;
    error_log   /var/log/nginx/your-app.error.log warn;
}
```

子路径（sub-uri）：

```
location /sub-uri {
  alias /data/app/your-app/current/public;

  passenger_enabled on;
  passenger_base_uri /sub-uri;
  passenger_env_var RAILS_RELATIVE_URL_ROOT /sub-uri;
  passenger_app_root /data/app/your-app/current;
}
```

部署到子路径的详细文档请参考 [Deploy Rails appliction to sub-uri
](https://bianjp.com/deploy-rails-appliction-to-sub-uri/)

默认使用 `/etc/nginx/conf.d/passenger.conf` 中配置的 ruby 版本，可通过 [passenger_ruby](https://www.phusionpassenger.com/library/config/nginx/reference/#passenger_ruby) 指定要使用的 ruby 版本

__注意：__ Passenger 使用的 ruby 版本必须与部署工具部署（尤其 bundle install）使用 ruby 版本一致

## 参考资料

* [Installation](https://www.phusionpassenger.com/library/walkthroughs/deploy/ruby/ownserver/nginx/oss/el7/install_passenger.html)
* [Configurations](https://www.phusionpassenger.com/library/config/nginx/reference/)
