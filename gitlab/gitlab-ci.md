# GitLab CI 安装、配置与维护

GitLab 自带 GitLab-CI，可用于自动构建、测试、部署

## 安装 GitLab Runner

GitLab Runner 是 GitLab-CI 的任务执行者，需单独安装

可安装在任意位置，并不需要与 GitLab 在同一台机器

### 添加软件源

国内使用[清华大学源](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-runner/)：

```bash
sudo tee /etc/yum.repos.d/gitlab-ci-multi-runner.repo <<-'EOF'
[gitlab-runner]
name=gitlab-runner
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-runner/yum/el$releasever-$basearch/
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
EOF
```

国外使用 GitLab 官方源：

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash
```

### 安装

```bash
export GITLAB_RUNNER_DISABLE_SKEL=true
sudo -E yum install gitlab-runner
```

## 管理命令

```bash
# 查看状态/启动/关闭/重启
sudo systemctl status/start/stop/restart gitlab-runner
# 启用/禁用自动启动。安装后默认已启用自动启动
sudo systemctl enable/disable gitlab-runner
````

## 更新

更新前检查[更新日志](https://gitlab.com/gitlab-org/gitlab-runner/-/blob/master/CHANGELOG.md)

```bash
sudo yum update
```

## 安装 Docker

使用 docker 作为执行任务的容器

参看 [docker 安装与维护](../base/docker.md)

## 添加 runner

添加后才能在 CI 中使用

```
sudo gitlab-runner register
```

若添加共享型 runner, token 为 Admin -> Runners 页面所示；

若添加专用 runner, token 为小组或项目的 Settings -> Runners 页面所示

## 并发度

默认每个 runner 的并发度是 1，即同一时间只能有一个构建使用该 runner

参考 [Advanced configuration options](https://docs.gitlab.com/runner/configuration/advanced-configuration.html)，设置全局的 `concurrent` 和每个 runner 的 `limit`

根据系统负载能力酌情设置

__`limit` > 1 可能会导致缓存经常失效__

## 参考资料

* [GitLab Runner Documentation](https://docs.gitlab.com/runner/)
* [Install GitLab Runner](https://docs.gitlab.com/runner/install/linux-repository.html)
* [Advanced configuration options](https://docs.gitlab.com/runner/configuration/advanced-configuration.html)
* [清华大学 gitlab-runner 源](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-runner/)
