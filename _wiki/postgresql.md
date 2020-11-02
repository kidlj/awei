---
title: PostgreSQL
---

### Ubuntu installation

    $ sudo apt-get install curl ca-certificates
    $ curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

    $ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

    $ sudo apt-get update
    $ sudo apt-get install postgresql-11 postgresql-client-11

### CentOS installation[1]

    $ yum install -y postgresql13-server

### Init Database

    # mkdir -p /data/pgsql/13/data
    # chown -R postgres:postgres /data/pgsql/13/data

    # systemctl edit postgresql-13 # to change data directory
    # add the following to the override file:
    [Service]
    Environment=PGDATA=/data/pgsql/13/data

    # systemctl daemon-reload

    # /usr/pgsql-13/bin/postgresql-13-setup initdb
    # systemctl enable postgresql-13
    # systemctl start postgresql-13

### Client Authentication

默认 `pg_hba.conf` 配置了一条允许任何操作系统本地用户 peer 登录的条目：

    local   all      all    peer

切换到系统用户 postgres，可以直接登录，下面使用 postgres role 创建业务数据 role 和 database：

    $ su postgres
    $ psql

    postgres=# create role dev with login password 'xxx';
    postgres=# create database accounts_dev; # database owner 为 postgres

配置 pg_hba.conf 允许内网使用 role 密码连接数据库：

    host	accounts_dev	dev		samenet			password

然后 reload pg 生效:

    $ sudo systemctl reload postgresql-13

### Privileges

pg_hba.conf 用于控制 role member 在 database 级别的 access 权限。Privileges 控制 role member 在 objects 级别的 SQL 权限。

当一个 object 被创建时，被赋予一个 owner，一般来说 owner 就是这个对象的创建者(role)。owner 有对该对象的所有权限。

可以变更一个 object 的 owner：

    ALTER TABLE table_name OWNER TO new_owner;

该操作仅当当前 role 既是该 object 的 owner，又是 new_owner 的 role member 时可行。

### GRANT PRIVILEGES[1]

> If WITH GRANT OPTION is specified, the recipient of the privilege can in turn grant it to others. Without a grant option, the recipient cannot do that. Grant options cannot be granted to PUBLIC.

> There is no need to grant privileges to the owner of an object (usually the user that created it), as the owner has all privileges by default. (The owner could, however, choose to revoke some of their own privileges for safety.)

> GRANT and REVOKE can also be done by a role that is not the owner of the affected object, but is a member of the role that owns the object, or is a member of a role that holds privileges WITH GRANT OPTION on the object. In this case the privileges will be recorded as having been granted by the role that actually owns the object or holds the privileges WITH GRANT OPTION. For example, if table t1 is owned by role g1, of which role u1 is a member, then u1 can grant privileges on t1 to u2, but those privileges will appear to have been granted directly by g1. Any other member of role g1 could revoke them later.

### ALTER ROLE[2]

    => ALTER ROLE demo WITH CREATEDB;
    => ALTER ROLE demo WITH NOCREATEDB;

### 查看信息

列出数据库：

    => \l;

切换数据库：

    => \c dbname;

列出表：

    => \dt;

查看 roles：

    => \du;


[1]: https://www.postgresql.org/download/linux/redhat/
[2]: https://www.postgresql.org/docs/11/sql-grant.html
[3]: https://www.postgresql.org/docs/9.1/sql-alterrole.html

