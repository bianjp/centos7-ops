# Python

## Python 3

```bash
# 需启用 EPEL 源
sudo yum install python36 python36-devel python36-setuptools
sudo easy_install-3.6 pip

sudo ln -s /usr/bin/python3.6 /usr/bin/python3
```

## PyPI 国内镜像

```
# 安装包时需要以 root 用户执行，因此配置文件保存在 root 用户目录
sudo mkdir -p ~root/.pip
sudo touch ~root/.pip/pip.conf
```

编辑 `~root/.pip/pip.conf`，添加下列源中的一个：

中国科技大学源：

```
[global]
index-url = https://pypi.mirrors.ustc.edu.cn/simple
trusted-host=pypi.mirrors.ustc.edu.cn
```

阿里云源：

```
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
trusted-host=mirrors.aliyun.com
```

阿里云内网源：

```
[global]
index-url = http://mirrors.aliyuncs.com/pypi/simple/
trusted-host=mirrors.aliyuncs.com
```

## 常用包

```
# 系统监控工具
sudo pip3 install glances
# For Python 2
sudo pip2 install glances
```
