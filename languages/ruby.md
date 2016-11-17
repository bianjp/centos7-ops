# Ruby

官方源中的版本较低，可使用 RVM 或 rbenv 安装

## RVM

RVM (Ruby Version Manager) 用来安装、管理不同的 Ruby 环境（Ruby 的不同实现、不同版本、不同的 GEM 包集）

参考 [rvm 官方安装文档](https://rvm.io/rvm/install)

```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable

# 允许 rvm 自动更新
echo rvm_autoupdate_flag=2 >> ~/.rvmrc

# 使用国内 Ruby 镜像
echo "ruby_url=https://cache.ruby-china.org/pub/ruby" >> ~/.rvm/user/db

source ~/.rvm/scripts/rvm
rvm install 2.3.1
rvm use 2.3.1 --default
gem install bundler
```

__注意：上述 rvm 安装步骤为单用户模式，部署 ruby 应用时使用的用户应为执行上述安装步骤的用户__

## 配置

```
cat <<EOF > ~/.gemrc
---
:backtrace: false
:bulk_threshold: 1000
:sources:
- https://gems.ruby-china.org/
:update_sources: true
:verbose: true
gem: "--no-user-install --no-document"
EOF
```

## 参考资料

* [Ruby China RVM 实用指南](https://ruby-china.org/wiki/rvm-guide)
* [Ruby China rbenv 实用指南](https://ruby-china.org/wiki/rbenv-guide)
