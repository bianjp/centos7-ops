# PostgreSQL

The world's most advanced open source database.

## 安装

官方源中版本过旧，使用 PostgreSQL 官方源

```
sudo yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-1.noarch.rpm
sudo yum install postgresql10 postgresql10-server

# Initialize the database
sudo /usr/pgsql-10/bin/postgresql-10-setup initdb

sudo systemctl enable postgresql-10
sudo systemctl start postgresql-10
```

## 创建用户

默认只有一个 `postgres` 用户，为方便使用，可添加一个与系统用户同名的新用户

```
sudo -u postgres -i
createuser --interactive
```

## 配置

默认的验证方式是 ident，为方便本地应用连接，可考虑改为 trust

修改 `/var/lib/pgsql/10/data/pg_hba.conf`:

```
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
```

## 参考资料

* [官网](https://www.postgresql.org/)
* [Install PostgreSQL](https://www.postgresql.org/download/linux/redhat/)
* [PostgreSQL Authentication Methods](https://www.postgresql.org/docs/current/static/auth-methods.html)
