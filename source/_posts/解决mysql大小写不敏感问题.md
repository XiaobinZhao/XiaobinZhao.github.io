---
title: mysql大小写敏感问题
date: 2020-10-20 09:49:01
tags:
- mysql
- 大小写敏感
categories:
- 技术积累
---

# 前言

Mysql/MariaDB 默认是大小写不敏感的，有时会比较影响业务，我们期望能够大小写敏感。

<!-- more -->

# 问题来源
python开发过程中会使用sqlalchemy 来做 ORM，但是发现常规做法会出现查询语句对查询条件大小写不敏感问题，比如：
> `session.query(desktop_pool)filter_by(name='b3').one_or_none()`   
> `session.query(desktop_pool)filter_by(name='B3').one_or_none()`

查询的结果是一样的。这样对有些业务就会造成困扰， 那么怎么做到查询时能够 **大小写敏感** 呢

# 排查过程
1. 首先想到是不是sqlalchemy本身的问题对大小写不敏感，查询sqlalchemy 中对字段的定义，比如name字段：

> name = Column("name", String(20), nullable=False,doc="桌面池的名称")

查看sqlalchemy对String()方法定义：
``` 
class String(Concatenable, TypeEngine):

    """The base for all string and character types.

    In SQL, corresponds to VARCHAR.  Can also take Python unicode objects
    and encode to the database's encoding in bind params (and the reverse for
    result sets.)

    The `length` field is usually required when the `String` type is
    used within a CREATE TABLE statement, as VARCHAR requires a length
    on most databases.

    """

    __visit_name__ = 'string'

    def __init__(self, length=None, collation=None,
                 convert_unicode=False,
                 unicode_error=None,
                 _warn_on_bytestring=False
                 ):
        """
        Create a string-holding type.

        :param length: optional, a length for the column for use in
          DDL and CAST expressions.  May be safely omitted if no ``CREATE
          TABLE`` will be issued.  Certain databases may require a
          ``length`` for use in DDL, and will raise an exception when
          the ``CREATE TABLE`` DDL is issued if a ``VARCHAR``
          with no length is included.  Whether the value is
          interpreted as bytes or characters is database specific.

        :param collation: Optional, a column-level collation for
          use in DDL and CAST expressions.  Renders using the
          COLLATE keyword supported by SQLite, MySQL, and PostgreSQL.
          E.g.::

            >>> from sqlalchemy import cast, select, String
            >>> print select([cast('some string', String(collation='utf8'))])
            SELECT CAST(:param_1 AS VARCHAR COLLATE utf8) AS anon_1

          .. versionadded:: 0.8 Added support for COLLATE to all
             string types.

        ...其他代码...
```
发现有一个collation属性可以设置，并且可以修改字段的DDL定义，也许可以试一下。
2. 在搜索引擎搜索`sqlalchemy 大小写敏感`问题，发现几个关键字：collation、binary
3. 查找[官方文档](https://www.osgeo.cn/sqlalchemy/core/type_basics.html)，用关键字collation、binary搜索，在最后发现这样一段：
```
from sqlalchemy.dialects.mysql import VARCHAR, TEXT

table = Table('foo', meta,
    Column('col1', VARCHAR(200, collation='binary')),
    Column('col2', TEXT(charset='latin1'))
)
```
这样设置本质上是修改mysql属性，可能意味着 大小写敏感的问题不在 sqlalchemy， 而在mysql
4. 搜索引擎搜索`mysql大小写敏感`, 有以下结果：
   1. 设置mysql大小写敏感参数   
    `lower_case_table_names=0` 表名存储为给定的大小和比较是区分大小写的   
    `lower_case_table_names = 1` 表名存储在磁盘是小写的，但是比较的时候是不区分大小写   
    `lower_case_table_names=2` 表名存储为给定的大小写但是比较的时候是小写的   
    `unix,linux下lower_case_table_names`默认值为 0 .Windows下默认值是 1 .Mac OS X下默认值是    
   查看mysql配置发现
   ``` 
    MariaDB [(none)]> show variables like 'lower_case%';
    +------------------------+-------+
    | Variable_name          | Value |
    +------------------------+-------+
    | lower_case_file_system | OFF   |
    | lower_case_table_names | 0     |
    +------------------------+-------+
   ```
    我们的mysql 配置是对的，无需修改
    2. 设置表或行的collation，使其为binary或case sensitive
    collate规则：   
    *_bin: 表示的是binary case sensitive collation，也就是说是区分大小写的   
    *_cs: case sensitive collation，区分大小写   
    *_ci: case insensitive collation，不区分大小写   
    查看mysql的默认collation   
    ```
    MariaDB [(none)]> select * from information_schema.collations where IS_DEFAULT='Yes' and COLLATION_NAME like 'utf8%';
    +--------------------+--------------------+----+------------+-------------+---------+
    | COLLATION_NAME     | CHARACTER_SET_NAME | ID | IS_DEFAULT | IS_COMPILED | SORTLEN |
    +--------------------+--------------------+----+------------+-------------+---------+
    | utf8_general_ci    | utf8               | 33 | Yes        | Yes         |       1 |
    | utf8mb4_general_ci | utf8mb4            | 45 | Yes        | Yes         |       1 |
    +--------------------+--------------------+----+------------+-------------+---------+
    ```
    我们一般使用utf8编码的数据库，所以只关注utf8、utf8mb4编码的排序规则（collation）   
    按照collate规则分析`utf8_general_ci`是ci结尾，所以是不区分大小写的。
    然后查看我们自己的数据库：   
    ```
    MariaDB [(none)]> select * from information_schema.SCHEMATA where SCHEMA_NAME='xview';
    +--------------+-------------+----------------------------+------------------------+----------+
    | CATALOG_NAME | SCHEMA_NAME | DEFAULT_CHARACTER_SET_NAME | DEFAULT_COLLATION_NAME | SQL_PATH |
    +--------------+-------------+----------------------------+------------------------+----------+
    | def          | xview       | utf8                       | utf8_general_ci               | NULL     |
    +--------------+-------------+----------------------------+------------------------+----------+
    ```
    查看表：   
    ```
    MariaDB [(none)]> select TABLE_SCHEMA, TABLE_NAME, TABLE_COLLATION from information_schema.tables where TABLE_NAME='desktop_pool';
    +--------------+--------------+-----------------+
    | TABLE_SCHEMA | TABLE_NAME   | TABLE_COLLATION |
    +--------------+--------------+-----------------+
    | xview        | desktop_pool | utf8_general_ci |
    +--------------+--------------+-----------------+

    ```
    查看字段： 
    ```
    MariaDB [(none)]> select TABLE_SCHEMA, TABLE_NAME, COLLATION_NAME  from information_schema.columns where TABLE_NAME='desktop_pool' and COLUMN_NAME='name';
    +--------------+--------------+-----------------+
    | TABLE_SCHEMA | TABLE_NAME   | COLLATION_NAME  |
    +--------------+--------------+-----------------+
    | xview        | desktop_pool | utf8_general_ci |
    +--------------+--------------+-----------------+
    ```
    **至此，我们已经找到根本原因是：字段设置的排序规则utf8_general_ci不区分大小写。**
   
# 解决方案
查询[mariadb文档](https://mariadb.com/kb/zh-cn/setting-character-sets-and-collations/)中有关排序规则设置可以知道:   
```
在MariaDB中，默认的字符集character set为latin1，默认的排序规则为latin1_swedish_ci(但不同的发行版可能会不同，例如Differences in MariaDB in Debian)。
字符集和排序规则都可以**从server端一直指定到字段级别**，client到server连接时也可以指定。**
当修改字符集但却没有指定排序规则时，将总是使用字符集的默认排序规则**。

**字符集和排序规则总是级联向下的，所以当没有为字段指定排序规则时，将查找表的排序规则，同样对于表来说会上查到数据库，对数据库来说会上查到server级**。
因此，可以使用极细粒度的字符集和排序规则来控制控制你的数据。
```
所以，为了能够区分大小写，请做以下配置：
1. 修改mysql/MariaDB的配置文件：`/etc/mysql/mariadb.conf.d/50-server.cnf`修改配置项：
```
character-set-server  = utf8
collation-server      = utf8_bin
```
2. 创建数据库的时候最好指定排序规则：`create database xview character set utf8 collate utf8_bin;`
3. 如果数据库已经存在，可以使用`alert`命令进行修改，参看[mariadb文档](https://mariadb.com/kb/zh-cn/setting-character-sets-and-collations/)
4. 你可以试试sqlalchemy用法，设置字段：`name = Column("name", String(20, collation='binary'), nullable=False, doc="桌面池的名称")`


**以上**
**至此MySQL大小写问题搞定！**

# 遗留问题
查看mysql的可用的utf8编码的排序规则，其中区分大小写的只有utf8_bin/utf8mb4_bin/utf8mb4_nopad_bin/utf8_nopad_bin 没有 utf8_xxx_cs 很是神奇！