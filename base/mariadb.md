# MariaDB

MariaDB 是 MySQL 的主要分支之一，推荐替代 MySQL 使用

## 安装

CentOS 官方源版本较旧，使用 MariaDB 官方源

```bash
# 官方镜像
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash

# 手动配置官方镜像
# 注意修改 MariaDB 版本号
sudo tee /etc/yum.repos.d/mariadb.repo <<-'EOF'
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.5/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
EOF

# 中科大镜像
sudo tee /etc/yum.repos.d/mariadb.repo <<-'EOF'
[mariadb]
name = MariaDB
baseurl = https://mirrors.ustc.edu.cn/mariadb/yum/10.5/centos7-amd64
gpgkey=https://mirrors.ustc.edu.cn/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck=1
EOF

sudo yum makecache
sudo yum install MariaDB-client MariaDB-server MariaDB-devel

sudo systemctl enable mariadb
sudo systemctl start mariadb

sudo mysql_secure_installation
```

## 配置

将默认字符集改为 utf8mb4，避免意外的问题：

`/etc/my.cnf.d/mysql-clients.cnf`:

```ini
[mysql]
default-character-set = utf8mb4

[mysqldump]
default-character-set = utf8mb4
```

`/etc/my.cnf.d/server.cnf`:

```ini
[mysqld]
character-set-server = utf8mb4
log-error = /var/log/mariadb/error.log
#general_log = 1
#general_log_file = /var/log/mariadb/general.log
slow_query_log = 1
slow_query_log_file = /var/log/mariadb/slow.log
```

创建日志目录：

```bash
sudo mkdir /var/log/mariadb
sudo chown mysql: /var/log/mariadb
```

日志轮滚配置：

```bash
sudo tee /etc/logrotate.d/mysql <<-'EOF'
/var/log/mariadb/*log /var/lib/mysql/mysqld.log {
    su mysql mysql
    missingok
    daily
    minsize 300M
    rotate 10
    compress
    delaycompress
    copytruncate
}
EOF
```

## 参考资料

* [官方主页](https://mariadb.org/)
* [MariaDB Package Repository Setup and Usage](https://mariadb.com/kb/en/library/mariadb-package-repository-setup-and-usage/)
* [MariaDB Repository Configuration Tool](https://downloads.mariadb.org/mariadb/repositories)
* [中科大镜像源配置](https://lug.ustc.edu.cn/wiki/mirrors/help/mariadb)
