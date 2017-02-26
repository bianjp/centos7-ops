# Docker

## 安装

添加 docker 镜像源：

```
# 国内使用清华大学源
sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[docker]
name=Docker Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker/yum/repo/centos7
enabled=1
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker/yum/gpg
EOF

# 国外使用官方源
sudo yum-config-manager \
    --add-repo \
    https://docs.docker.com/engine/installation/linux/repo_files/centos/docker.repo
```

```
sudo yum install docker-engine

sudo systemctl start docker
sudo systemctl enable docker

# 将自己添加到 docker 组以便无需 sudo 权限执行 docker 命令。需要重新登录
sudo usermod -aG docker `whoami`
```

## 配置

Docker 守护进程的配置文件为 `/etc/docker/daemon.json`

参考 [Daemon configuration file](https://docs.docker.com/engine/reference/commandline/dockerd/#/daemon-configuration-file)

## 存储位置

Docker 镜像等数据默认保存在 `/var/lib/docker`，若系统盘空间较小，可改为其它目录

创建或修改配置文件 `/etc/docker/daemon.json`，添加：

```
{
  "graph": "/data/docker"
}
```

注意配置文件为 JSON 格式，花括号、逗号不要漏掉或多余

```
# 将原目录的数据移动到新目录：
sudo mv /var/lib/docker /data/docker

# 重启 docker daemon
sudo systemctl restart docker
```

### 镜像加速

国内拉取 docker 镜像很慢且失败率很高，可使用阿里云镜像加速器

登录 [阿里云](https://cr.console.aliyun.com/#/accelerator) 获取加速地址（需认证为开发者），添加到 `/etc/docker/daemon.json` 中：

```
{
  "registry-mirrors": ["your_mirror_address"]
}
```

也可考虑使用其它加速服务：

* [网易 DockerHub 加速](https://c.163.com/wiki/index.php?title=DockerHub%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F)
* [Daocloud 加速器](https://www.daocloud.io/mirror)

## 参考资料

* [Docker Documentation](https://docs.docker.com/)
* [Install Docker Engine](https://docs.docker.com/engine/installation/)
* [清华大学 docker 源](https://mirror.tuna.tsinghua.edu.cn/help/docker/)
* [Daemon configuration file](https://docs.docker.com/engine/reference/commandline/dockerd/#/daemon-configuration-file)