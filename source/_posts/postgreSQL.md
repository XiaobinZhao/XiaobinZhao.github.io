---
title: PostgreSQL初探
date: 2022-08-11 18:40:01
tags:
- PostgreSQL
- psql
- db 
categories:
- 技术积累
---

# postgreSQL

PostgreSQL 是一个免费的对象-关系数据库服务器(ORDBMS)，在灵活的BSD许可证下发行。

PostgreSQL 开发者把它念作 **post-gress-Q-L**。

PostgreSQL 的 Slogan 是 "世界上最先进的开源关系型数据库"。

<!-- more -->

## 安装

1. windows安装 https://www.enterprisedb.com/downloads/postgres-postgresql-downloads

2. ubuntu/debian安装：

   1. ` sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'`
   2. `wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc --no-check-certificate | sudo apt-key add -`
   3. `apt-get update`
   4.  ` apt-get install postgresql postgresql-client`

3. 遇到的一些问题

   debian9上安装的postgresql14.

   1. psql命令得到 `错误: 连接到套接字"/var/run/postgresql/.s.PGSQL.5432"上的服务器失败:拒绝连接`

      psql默认使用5432端口连接，但是netstat查询发现启动的是5433端口。可修改配置文件为5432

      `vim /etc/postgresql/14/main/postgresql.conf`

   2. `psql -l -U postgres`得到输出：`psql: 错误: 连接到套接字"/var/run/postgresql/.s.PGSQL.5432"上的服务器失败:致命错误:  对用户"postgres"的对等认证失败`，安装的时候并没有让输入密码呀，怎么回事呢？

      其实是postgresql与mysql/sql server等的不同之处是：PostgreSQL默认创建了名为postgres数据库用户账户，还创建了名为postgres的Unix系统账户。
   
4. 外部链接

   1. 修改 `vim /etc/postgresql/14/main/pg_hba.conf`,增加一条`host    all             all             0.0.0.0/0               md5`
   2. 修改 `/etc/postgresql/14/main/postgresql.conf`,修改 `listen_addresses = '0.0.0.0'`
   3. 重启服务
   3. 注意防火墙允许相应的端口


## 概念

1. schema

   PostgreSQL 模式（SCHEMA）可以看着是一个**表的集合**。默认包含一个public的schema

   一个模式可以包含视图、索引、数据类型、函数和操作符等。

   相同的对象名称可以被用于不同的模式中而不会出现冲突，例如 schema1 和 myschema 都可以包含名为 mytable 的表。

   使用模式的优势：

   - 允许多个用户使用一个数据库并且不会互相干扰。
   - 将数据库对象组织成逻辑组以便更容易管理。
   - 第三方应用的对象可以放在独立的模式中，这样它们就不会与其他对象的名称发生冲突。

   模式类似于操作系统层的目录，但是模式不能嵌套。

2. 管理员用户postgres

   安装完成后，PostgreSQL默认创建了名为postgres数据库用户账户，其与MySQL的root以及SQLServer的sa账户一样，是超级管理员账户，但与MySQL不一样的是，PostgreSQL还创建了名为postgres的Unix系统账户。

   ```shell
   root@xview:~# cat /etc/passwd
   root:x:0:0:root:/root:/bin/bash
   ... 省略
   postgres:x:113:117:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
   ```

3. 默认数据库

   模板数据库就是创建新database时，PostgreSQL会基于模板数据库制作一份副本，其中会包含所有的数据库设置和数据文件。和SQLServer的master数据库一样，PostgreSQL的默认数据库是template1和template0，你可以用命令行管理工具psql来管理它，当然前提是要切换到postgres的系统账户.

   建库时如果不指定 TEMPLATE 属性，默认用的是 **template1** 模板库.
   
   ```shell
   # 默认模板库为 template1
   postgres=# create database db1;
   CREATE DATABASE
   # 备注：建库时如果不指定 TEMPLATE 属性，默认用的是 template1 模板库.
   
   # 手工指定模板库
   postgres=# create database db2 template template0;
   CREATE DATABASE
   # 备注：也可以指定模板库为 template0
   ```
   
   template1 和 template0 的区别？
   
   - template1 可以连接并创建对象，template0 不可以连接.相比 template0,template1被称为非干净数据库，而 template0被称为干净的数据库。
   
     ```shell
     postgres=# \c template1
     您现在已经连接到数据库 "template1",用户 "postgres".
     template1=# \c template0
     连接到套接字"/var/run/postgresql/.s.PGSQL.5432"上的服务器失败:致命错误:  数据库 "template0" 当前不接受联接
     保留上一次连接
     ```
   
   - 使用 template1 模板库建库时不可指定新的 encoding 和 locale，而 template0 可以
   
     ```shell
     postgres=# create database db3 TEMPLATE template0 ENCODING 'SQL_ASCII' ;
     CREATE DATABASE
     postgres=# create database db4 TEMPLATE template1 ENCODING 'SQL_ASCII' ;
     错误:  新的编码(SQL_ASCII)与模板数据库(UTF8)的编码不兼容
     提示:  在模版数据库中使用同一编码，或者使用template0作为模版.
     postgres=# create database db4 TEMPLATE template1 ENCODING 'UTF8' ;
     CREATE DATABASE
     ```
   
   - template0 库和 template1 都不可删除

## 客户端

postgresql默认安装完成之后有用户`postgres`

### windows

1. pgAdmin
2. [beekeeper studio](https://www.beekeeperstudio.io/download/?ext=exe&arch=&type=installer&edition=community)

### psql

1. 进入数据库 

   - `sudo -i -u postgres`切换到postgres unix用户
   - `psql`以postgres 超管进入

2. postgres安装完成后，会增加2个命令：createuser、createdb

   - `createuser --superuser user1`
   - `createdb -O user1  db1`
   
2. 也可以在psql命令内，创建数据库

   ```shell
   postgres=# create database db1;
   CREATE DATABASE
   ```

   建库时如果不指定 TEMPLATE 属性，默认用的是 template1 模板库.
   
4. 或者使用已经创建好的用户和数据库进入

   `psql -U user1-d db1`

5. 其他命令

   - 命令行操作的帮助`\?`
   - 查看数据库列表 `\l`
   - 切换到test数据库`\c test`
   - 查看当前schema中所有的表`\d`
   - 查看表的结构`\d [schema.]table`
   - 查询结果横纵显示切换`\x`
   - 执行SQL消耗时间开关`\timing`
   - 查看命令历史记录`\s`
   - 显示字符集`\encoding`
   - 查看所有的sql关键字 `\h`

   

