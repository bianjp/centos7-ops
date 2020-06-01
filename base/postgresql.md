# PostgreSQL

The world's most advanced open source database.

## 安装

官方源中版本过旧，使用 PostgreSQL 官方源

```
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# 禁用不需要的仓库
sudo yum-config-manager --disable pgdg*
sudo yum-config-manager --enable pgdg-common pgdg12

sudo yum install -y postgresql12-server

# Initialize the database
sudo /usr/pgsql-12/bin/postgresql-12-setup initdb

sudo systemctl enable postgresql-12
sudo systemctl start postgresql-12
```

## 创建用户

默认只有一个 `postgres` 用户，为方便使用，可添加一个与系统用户同名的新用户

```
sudo -u postgres -i
createuser --interactive

# 创建普通用户
createuser --no-createdb --no-createrole --no-superuser --pwprompt --interactive
# 创建数据库
createdb --owner USER_NAME DATABASE_NAME
```

## 配置

### 允许远程连接

修改 `/var/lib/pgsql/12/data/postgresql.conf`:

```
listen_addresses = '*'
```

### trust 认证

默认的验证方式是 ident，为方便本地应用连接，可考虑改为 trust

修改 `/var/lib/pgsql/12/data/pg_hba.conf`:

```
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
```

### 密码认证

修改 `/var/lib/pgsql/12/data/pg_hba.conf`:

```
# 出于安全考虑，建议只允许指定数据库或用户远程连接，尽量不要允许默认的管理员帐号 postgres 远程连接
host    all             all             0.0.0.0/0               password
```

## 参考资料

* [官网](https://www.postgresql.org/)
* [Install PostgreSQL](https://www.postgresql.org/download/linux/redhat/)
* [PostgreSQL Authentication Methods](https://www.postgresql.org/docs/current/static/auth-methods.html)
