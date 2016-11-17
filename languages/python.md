# Python

## Python 3

官方源中不包含 Python 3，可使用 EPEL 源中的 Python 3.4，或 [IUS 源](https://ius.io/) 中的 Python 3.5

EPEL:

```
sudo yum install epel-release
sudo yum install python34 python34-devel
sudo yum install python34-setuptools
sudo easy_install-3.4 pip
```

IUS:

```
sudo rpm -Uvh https://centos7.iuscommunity.org/ius-release.rpm
sudo yum update
sudo yum install python35u python35u-devel
sudo yum install python35u-setuptools python35u-pip
```

IUS 源安装的 `python35u` 不会自动创建 `/usr/bin/python3`，为方便使用，手动创建符号链接：

```
sudo ln -s /usr/bin/python3.5 /usr/bin/python3
sudo ln -s /usr/bin/pip3.5 /usr/bin/pip3
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
sudo pip3.5 install glances
# For Python 2
sudo pip2 install glances
```
