# Node.js

## 安装

EPEL 源中已有 6.x 版本

```
sudo yum install nodejs
```

若仍想使用更新版本，可使用 [NodeSource 源](https://github.com/nodesource/distributions)：

```
curl -sL https://rpm.nodesource.com/setup_7.x | sudo bash -
# 使用 TUNA 源
sudo sed -i 's#https://rpm.nodesource.com/pub_6.x/#https://mirror.tuna.tsinghua.edu.cn/nodesource/rpm_6.x/#g' /etc/yum.repos.d/nodesource-el.repo

sudo yum install nodejs
```

## 配置

Npm 官方源速度较慢且不稳定，使用[淘宝源](http://npm.taobao.org/)：

```
cat <<EOF > ~/.npmrc
registry=https://registry.npm.taobao.org
EOF
```
