---
title: PostgreSQL
---

基本操作
=======

### 创建用户

    CREATE ROLE name LOGIN;

### 创建数据库

    CREATE DATABASE dbname OWNER rolename;

### 查看信息

列出数据库：

    \l

切换数据库：

    \c dbname

列出表：

    \dt
