# GitLab CI 安装、配置与维护

GitLab 自带 GitLab-CI，可用于自动构建、测试、部署

## 安装 GitLab Runner

GitLab Runner 是 GitLab-CI 的任务执行者，需单独安装

可安装在任意位置，并不需要与 GitLab 在同一台机器

官方源经常连接失败，使用[清华大学 gitlab-ci-multi-runner 源](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ci-multi-runner/)：

```
sudo tee /etc/yum.repos.d/gitlab-ci-multi-runner.repo <<-'EOF'
[gitlab-ci-multi-runner]
name=gitlab-ci-multi-runner
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ci-multi-runner/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
EOF

sudo yum install gitlab-ci-multi-runner

# 查看状态/启动/关闭/重启
sudo systemctl status/start/stop/restart gitlab-runner
# 启用/禁用自动启动。安装后默认已启用自动启动
sudo systemctl enable/disable gitlab-runner

# 更新
sudo yum update
# 更新后需重启
sudo systemctl restart gitlab-ci-multi-runner
```

## 安装 Docker

使用 docker 作为执行任务的容器

参看 [docker 安装与维护](../base/docker.md)

## 添加 runner

```
sudo gitlab-runner register
```

若添加共享型 runner, token 为 Admin -> Runners 页面所示；

若添加专用 runner, token 为项目的 Settings -> Runners 页面所示

## 参考资料

* [Install GitLab Runner](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/#install-gitlab-runner)
* [Install Docker](https://docs.docker.com/engine/installation/linux/centos/)
* [清华大学 gitlab-ci-multi-runner 源](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ci-multi-runner/)
