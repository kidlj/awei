---
title: PostgreSQL
---

### Ubuntu installation

    $ sudo apt-get install curl ca-certificates
    $ curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

    $ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

    $ sudo apt-get update
    $ sudo apt-get install postgresql-11 postgresql-client-11

### CentOS installation

See https://www.postgresql.org/download/linux/redhat/

    $ yum install -y postgresql13-server

### Init Database

    # systemctl edit postgresql-13 # to change data directory
    # add the following to the override file:
    [Service]
    Environment=PGDATA=/data/pgsql/13/data

    # systemctl daemon-reload

    # /usr/pgsql-13/bin/postgresql-13-setup initdb
    # systemctl enable postgresql-13
    # systemctl start postgresql-13


### 初始用户

PostgreSQL 初始化以后，一般会以 `postgres` 用户运行，同时系统内建了该用户为 superuser，连接认证方式为 `local` 和 `peer`(即与操作系统 user 对应)。

    $ sudo su postgres
    $ psql

### Authentication

PostgreSQL 使用 `pg_hba.conf` 文件控制客户端的连接认证。

更改此文件后需要发送 SIGHUP 信号 reload 进程：

    $ sudo su postgres
    $ pg_ctlcluster 11 main reload

举例：允许某个 ip 的 demo 用户连接 demo 数据库，使用 密码：

        TYPE    DATABASE    USER    IP              METHOD
        Host    demo        demo    172.17.8.1/24   md5

从上到下，匹配到一个项就开始进行认证，而后连接或者拒绝连接，不会 fallthrough。

### 创建业务用户和业务数据库

    $ psql // as postgres user
    (postgres) => CREATE ROLE demo WITH LOGIN CREATEDB PASSWORD 'demo';
    (postgres) => CREATE DATABASE demo OWNER demo;


### Client Authentication

默认 `pg_hba.conf` 配置了一条允许任何操作系统本地用户 peer 登录的条目：

    local   all      all  <>  peer

切换到系统用户 postgres，可以直接登录，下面使用 postgres role 创建业务数据 role 和 database：

    $ su postgres
    $ psql

    postgres=# create role dev with login password 'xxx';
    postgres=# create database accounts_dev; # database owner 为 postgres

配置 pg_hba.conf 允许内网使用 role 密码连接数据库：

    host	accounts_dev	dev		samenet			password

然后 reload pg 生效:

    $ sudo systemctl reload postgresql-13

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


[1]: https://www.postgresql.org/docs/11/sql-grant.html
[2]: https://www.postgresql.org/docs/9.1/sql-alterrole.html

