# GitLab-CI 使用方法

GitLab 自带 GitLab-CI，可用于自动构建、测试、部署

## 基本概念

* Job: 任务，执行命令的单元
    * 可配置执行环境、依赖、执行的条件、要执行的 shell 命令等
    * shell 命令执行成功（返回值为 0）则任务成功，否则失败
    * 不同任务完全独立，因此可配置不同的环境、依赖，可并行执行，但需注意每个任务中都要执行安装依赖等命令（为避免重复，可放在 `before_script` 中）

* Stage: 任务的类别
    * 任务的 stage 默认为 test
    * 一个 stage 可以包含多个任务
    * 不同 stage 的任务按 `stages` 指令指定的顺序（默认为build, test, deploy）执行
    * 同一 stage 的多个任务可并行执行

* Executor: 执行任务的容器类别，如 ssh, docker, virtualbox等。__我们使用 docker__

* Runner: 执行任务的容器
    * 分共享和专用两种，共享的可供所有项目使用，专用的只能用于特定项目（一个或多个）
    * 项目必须有可用的 runner 才能执行 GitLab-CI 任务
    * 创建 runner 只能在安装了 gitlab-ci-multi-runner 的服务器上执行 shell 命令
    * 查看/管理项目的 runner 在项目的 Settings -> Runners

* Deploy key: 对 git 仓库有只读权限的密钥对的公钥，可用于在部署服务器上拉取代码。在项目的 Settings -> Deploy keys 中配置

* Secret Variable: GitLab-CI 中执行任务时的环境变量，只有对仓库有 master 及以上权限的用户才能查看和管理。可用于保存敏感信息（如部署使用的 SSH 私钥），避免将敏感信息保存在人人可见的 `.gitlab-ci.yml` 配置文件中。在项目的 Settings -> Variables 中配置

## 配置

1. 在项目的 Settings -> Project Settings -> Features 中启用 `Builds`

2. 确保有可用 runner。创建 runner 只能由 GitLab 服务器管理员操作。尽量使用共享型 runner 以便管理

    如使用共享 runner，在项目的 Settings -> Runners 中启用共享 runner

3. 在项目代码的根目录下添加 `.gitlab-ci.yml`，配置自动构建的环境（Docker镜像名称）、依赖的服务（MySQL、Redis等，也需是 Docker 镜像名称）、需要执行的任务（stage）等。将文件提交到版本库

    配置文件语法参考[官方文档](http://doc.gitlab.com/ee/ci/yaml/README.html)

4. 如需自动部署，生成一对 SSH 密钥（在任意机器上均可）

    ```
    ssh-kengen -t rsa -C 'project_name@gitlab-ci'
    ```

   * 为使 GitLab-CI 有权限操作部署服务器：
      * 将公钥添加到要部署服务器的 `~/.ssh/authorized_keys` 中
      * 将私钥添加到项目的 Secret Variables 中，名称随意，如 "SSH_PRIVATE_KEY"
      * `.gitlab-ci.yml` 配置文件中，执行部署命令前从环境变量中读取 SSH 私钥

   * 为使部署服务器有权限拉取代码（若不需要在部署服务器上拉取代码，略过）：
      * 若自动部署工具（如 mina）启用了 SSH agent forwarding， 将公钥添加到项目的 Deploy Keys 中
      * 若未启用，将部署服务器上的 `~/.ssh/id_rsa.pub` 添加到项目的 Deploy Keys 中

## 技巧/注意事项

以下部分命令通过修改环境变量 PATH 生效，若放在外部文件中，须通过 `source` 命令引入方有效

### 处理 SSH 私钥

私钥属敏感信息，不应保存在人人可见的 `.gitlab-ci.yml` 或其它项目文件中，应保存在项目配置的 Variables 中

使用以下命令在任务中使用私钥：

```
if [ -n "$SSH_PRIVATE_KEY" ]; then
  eval $(ssh-agent -s)
  ssh-add <(echo "$SSH_PRIVATE_KEY")
  mkdir -p ~/.ssh && chmod 700 ~/.ssh
  # disable host key checking
  echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
fi
```

### RVM

由于 GitLab-CI 使用 docker 容器时使用的是 non-login shell，导致默认无法使用 RVM

使用以下命令解决：

```
if [ -x '/etc/profile.d/rvm.sh' ]; then
  source /etc/profile.d/rvm.sh
  rvm use default
fi
```

### Locale

同样是由于 non-login shell 的问题，导致任务执行时环境的 locale 不是 utf-8，若有命令输出非 ASCII 字符，可能会导致命令执行失败

使用以下命令解决：

```
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

### 使用服务

可以使用任何 docker 镜像作为服务，如 MySQL, Redis

这些提供服务的 docker 容器会被链接到执行 GitLab-CI 任务的容器，主机名称通过以下规则从 docker 镜像名称转换：

* 忽略":"及其后面的
* "/" 替换为 "__"

如 "bianjp/mariadb-alpine:latest" 的主机名为 "bianjp__mariadb-alpine"，"redis:alpine" 的主机名为 "redis"

在应用代码中可通过这些主机名使用这些服务

为了便于维护，建议在 `.gitlab-ci.yml` 中把主机名定义为变量，在应用代码中动态获取环境变量而非硬编码

为减少带宽、内存占用，尽量使用轻量级的 docker 镜像

## 示例

项目根目录下添加 `.gitlab-ci-before-script.sh`:

```
#!/bin/bash

# SSH private key
if [ -n "$SSH_PRIVATE_KEY" ]; then
  eval $(ssh-agent -s)
  ssh-add <(echo "$SSH_PRIVATE_KEY")
  mkdir -p ~/.ssh && chmod 700 ~/.ssh
  # disable host key checking
  echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
fi

# RVM
if [ -x '/etc/profile.d/rvm.sh' ]; then
  source /etc/profile.d/rvm.sh
  rvm use default
fi

# locale
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

`.gitlab-ci.yml` 示例：

```yaml
image: bianjp/rails-env-centos7:latest

# 服务、任务运行时的环境变量
variables:
  # 用于 bianjp/mariadb-alpine 镜像初始化
  MYSQL_ROOT_PASSWORD: 'root'
  MYSQL_INITDB_SKIP_TZINFO: 'true'

  # 用于 Rails 配置，在配置文件中动态获取这些环境变量
  DB_HOST: 'bianjp__mariadb-alpine'
  DB_USERNAME: 'root'
  DB_PASSWORD: 'root'

  REDIS_HOST: 'redis'
  REDIS_PORT: '6379'

# 在不同构建之间缓存部分内容
cache:
  key: 'global'
  paths:
    - vendor/bundle

services:
  - bianjp/mariadb-alpine:latest
  - redis:alpine

stages:
  - test
  - deploy

before_script:
  # 必须使用 source 命令
  - source .gitlab-ci-before-script.sh

  # 处理配置文件
  - cp config/database.gitlab-ci.yml config/database.yml
  - cp config/secrets.gitlab-ci.yml config/secrets.yml

  # 安装依赖
  - bundle install --path vendor/bundle

# 测试
test:
  stage: test
  script:
    - RAILS_ENV=test bundle exec rake db:setup
    - bundle exec rake spec

# 部署到生产服务器
production:
  stage: deploy
  script:
    - bundle exec mina production deploy
  only:
    - master

# 部署到开发服务器
development:
  stage: deploy
  script:
    - bundle exec mina dev deploy
  only:
    - dev
```

## 参考资料

* [GitLab-CI docs](http://doc.gitlab.com/ce/ci/)
* [Using Docker Images](http://doc.gitlab.com/ee/ci/docker/using_docker_images.html)
* [.gitlab-ci.yml usage](http://doc.gitlab.com/ce/ci/yaml/README.html)
* [Using SSH keys](https://gitlab.com/help/ci/ssh_keys/README.md)
