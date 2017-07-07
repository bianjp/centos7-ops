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

## rbenv

```
git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
git clone https://github.com/sstephenson/rbenv-gem-rehash.git ~/.rbenv/plugins/rbenv-gem-rehash
git clone https://github.com/rkh/rbenv-update.git ~/.rbenv/plugins/rbenv-update
# 使用 Ruby China 的镜像安装 Ruby, 国内用户推荐
git clone https://github.com/AndorChen/rbenv-china-mirror.git ~/.rbenv/plugins/rbenv-china-mirror

tee -a ~/.bash_profile <<-'EOF'
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
EOF
```

安装 ruby:

```
rbenv install 2.3.3
# 默认使用版本
rbenv global 2.3.3
```

## 手动编译安装

以 root 身份执行：

```
RUBY_VERSION=2.4.1
RUBY_DOWNLOAD_MIRROR=https://cache.ruby-lang.org/pub/ruby/
# RUBY_DOWNLOAD_MIRROR=https://cache.ruby-china.org/pub/ruby/

yum install -y autoconf gcc gcc-c++ make automake patch
yum install -y git openssh curl which tar gzip bzip2 unzip zip
yum install -y openssl-devel libyaml-devel libffi-devel readline-devel zlib-devel gdbm-devel ncurses-devel
yum install -y epel-release yum-utils
yum-config-manager --enable epel

mkdir -p /usr/local/etc
echo -e "install: --no-document\nupdate: --no-document" > /usr/local/etc/gemrc
mkdir /build && cd /build
curl -o ruby.tar.gz "$RUBY_DOWNLOAD_MIRROR/ruby-$RUBY_VERSION.tar.gz"
mkdir ruby && tar -xzf ruby.tar.gz -C ruby --strip-components=1
cd ruby
./configure --disable-install-doc --enable-shared
make -j"$(nproc)"
make install
cd / && rm -rf /build
```

```
gem update --system && gem install bundler
```

## 配置

使用[Ruby China gem 源](https://gems.ruby-china.org/)，禁止安装文档

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

* [RVM](https://rvm.io/)
* [rbenv GitHub Repository](https://github.com/rbenv/rbenv/)
* [Ruby China RVM 实用指南](https://ruby-china.org/wiki/rvm-guide)
* [Ruby China rbenv 实用指南](https://ruby-china.org/wiki/rbenv-guide)
* [Ruby China gem 源](https://gems.ruby-china.org/)
