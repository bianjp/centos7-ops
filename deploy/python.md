# 部署 Python 应用

使用 [Supervisor](http://supervisord.org/) 监控和管理 Python 应用。

查看 [Supervisor 文档](./supervisor.md) 以安装和配置 supervisor。

## 部署工具

Python 应用部署较为简单，直接使用 Shell 脚本部署。

将以下两个文件保存到项目根目录，并根据项目实际需要进行修改：

`deploy.sh`:

```shell
#!/usr/bin/bash

user=your-deploy-user
server=example.com
deploy_dir=/data/app/weixin-vote
supervisor_program_name=weixin-vote

ssh $user@$server "[ -d $deploy_dir ] || mkdir $deploy_dir"
ssh $user@$server "[ -d $deploy_dir/log ] || mkdir $deploy_dir/log"
rsync -vrh --checksum --delete --exclude-from=deploy_exclude.list ./ $user@$server:$deploy_dir

if [[ "$1" != "setup" ]]; then
  ssh $user@$server "supervisorctl restart $supervisor_program_name:"
fi
```

`deploy_exclude.list`:

```
/.git
/.gitignore
/README.md
/config.py
/deploy_exclude.list
/deploy.sh
/log
__pycache__
*.pyc
```

部署只需在项目根目录执行：

```shell
./deploy.sh
```

* `deploy.sh` 须有执行权限 (`chmod 755 deploy.sh`)
* 用户须对服务器有 SSH 连接权限

## 部署步骤

初次部署：

1. 在服务器上创建数据库、安装依赖
2. 在项目根目录执行 `./deploy.sh setup`
3. 在服务器上进行必要的修改（如数据库配置），测试应用能否正常启动和运行
4. 在服务器上添加 supervisor 配置文件，并重载配置
5. 在服务器上配置 Nginx （若是 web 项目）

后续部署：

1. 若有新的依赖，或数据库更新，在服务器上手动处理
2. 在项目根目录执行 `./deploy.sh`
