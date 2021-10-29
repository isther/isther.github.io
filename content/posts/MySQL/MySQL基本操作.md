---
title: "MySQL基本操作"
date: 2021-07-15T22:31:51+08:00
# weight: 1
# aliases: ["/first"]
categories: ["MySQL"]
tags: ["数据库","MySQL"]
# author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "简单介绍了一下高级SQL操作"
canonicalURL: "https://canonical.url/to/page"
disableShare: false
disableHLJS: false
hideSummary: true
searchHidden: false

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

### SQL语法规则

>  概念
>  **SQL语法规则**：SQL是一种结构化编程语言

* 基础SQL指令通常是以行为单位
* SQL指令需要语句结束符，默认是英文分号：`;`、`\g`、`\G`
    * `\G`：主要用于查询数据
* SQL指令类似自然语言
* 编写的SQL中如果用到了关键字或者保留字，需要使用反引号``来包括，让系统忽略

<!--more-->

> 示例

1. 结构创建

```mysql
create 数据类型 结构名 结构类型;
```

2. 显示结构

```mysql
#显示结构
show 结构类型（复数）;

#显示结构创建详细
show create 结构类型 结构名;
```

3. 数据操作（数据表）

```mysql
#新增数据
insert into 表名 values

#查看数据
select from 表名

#更新数据
update 表名 set

#删除数据
delete from 表名
```



根据数据库的对象层级，可以将基础SQL操作分为三类：

* 库操作：数据库相关操作
* 表操作：数据表（字段）相关操作
* 数据操作：数据相关操作

### SQL库操作

#### 创建数据库

* 使用`create database 数据库名字` 创建
    * 数据库层面可以指定字符集：`charset/character set`
* 数据库层面可以指定校对集：`collate`
* 创建数据库会在磁盘指定存放处产生一个文件夹
* 创建语法：

```mysql
create database 数据库名字 [数据库选项];
```

##### 示例

1. 创建一个指定名字的数据库

    ```mysql
    create database test_1;
    ```

2. 创建一个指定字符集的数据库

    ```mysql
    create database test_2 charset utf8MB4
    ```

3. 创建一个指定校对集的数据库

    ```mysql
    create database test_3 charset utf8MB4 collate utf8mb4_general_ci;
    ```

数据库命名规则：与C同

#### 显示数据库

* 数据库的查看是根据用户权限限定的
* 数据库的查看分为两种查看方式：
    * 查看全部数据库
    * 查看数据库创建指令

##### 示例

1. 显示数据库

    ```mysql
    show databases;
    ```

2. 显示数据库创建命令

    ```mysql
    show create databases test_1;
    ```

    

    

#### 使用数据库

* 数据库的操作通常是针对数据表或数据
* 通过使用数据库可以让后续指令默认针对具体数据库环境、简化后续命令
* 使用数据库语法：`use 数据库名字;`

##### 示例

```mysql
use test_1;
```

#### 修改数据库

* 数据库名字不可修改（老版本可以）

    * 先新增
    * 后迁移
    * 再删除

* 数据库修改分为两个部分

    * 字符集
    * 校对集

* 数据库修改指令（与创建指令差不多）

    ```mysql
    alter database 数据库名字 库选项
    ```

##### 示例

1. 修改数据库字符集

    ```mysql
    alter database test_2 charset dbk;
    ```

2. 修改数据库校对集（如果字符集修改必须同时改变字符集）

    ```mysql
    alter database test_3 charset gbk collate gbk_chinese_ci;
    ```

#### 删除数据库

* 删除数据库会删除数据库内所有的表和数据

* 删除数据库操作要慎重（删前备份）

* 删除数据库后，对应的存储文件夹就会消失

* 删除语法

    ```mysql
    drop database 数据库名字;
    ```

##### 示例

```mysql
drop database test_1;
```

### SQL表（字段）操作

#### 创建数据表

根据业务需求，确定数据表的字段信息，然后创建表结构

* 表与字段不分家，相辅相成

* 表的创建需要指定存储的数据库

    * 明确指定数据库
    * 先使用数据库

* 字段至少需要指定改名字、类型

* 数据库表不限定字段数量

    * 每个字段间使用逗号`,`分隔
    * 最后一个字段不需要逗号

* 表可以指定表选项（都有默认值）

    * 存储引擎：engine[=]具体存储引擎
    * 字符集：[default] charset 具体字符集 （继承数据库）
    * 校对集：collate（继承数据库）

* 表创建语法

    ```mysql
    create table [数据库名.]表名(
    	字段名 字段类型,
        ...
    	字段名 字段类型,
    )表选项;
    ```

##### 示例

1. 创建简单数据表

    ```mysql
    create table t_1(
    	name varchar(50)
    );
    ```

2. 创建数据表---多字段

    ```mysql
    create table t_2(
    	name varchar(50),
        age int,
        gender varchar(10)
    );
    ```

3. 创建数据表---表选项

    ```mysql
    create table t_3(
    	name varchar(50)
    )engine Innodb charset utf8MB4;
    ```

> **拓展**
>
> 存储引擎是指数据存储和管理方式，MySQL中提供了多种存储引擎，一般使用默认存储引擎。
>
> * InnoDB
>     * 默认存储引擎
>     * 支持事务处理和外键
>     * 数据统一管理
> * MyIsam
>     * 不支持事务和外键
>     * 数据、表结构、索引独立管理
>     * MySQL5.6以后不在维护
>
> 
>
> 如果想创建一个与已有表一样的数据表，MySQL提供了一个便捷的复制模式
>
> * `create table 表名 like 数据库名字.表名`



#### 显示数据表

* 数据表的显示与用户权限有关
* 显示数据表有两种方式
    * 显示所有数据表
    * 显示具体数据表的创建指令

##### 示例

1. 显示所有数据表---当前数据库下

    ```mysql
    show tables;
    ```

2. 显示所有数据表---指定数据库

    ```mysql
    show tables from test_1;
    ```

3. 显示部分关联数据表---匹配

    ```mysql
    show tables like '%like';
    #_：匹配一个字符(固定位置)
    #%：匹配N个字符
    ```

4. 显示数据表的创建指令

    ```mysql
    show create table t_1;
    ```

    

#### 查看数据表

* 通常是查看字段信息

* 详细的显示字段的各项信息

* 查看语法有三种

    ```mysql
    desc 表名;
    describe 表名;
    show columns from 表名;
    ```

#### 更改数据表

* 更改表名：`rename teble 表名 to 新表名`
* 修改表选项：`alter table 表名`

##### 示例

1. 修改表名

    ```mysql
    rename table t_1 to t1;
    ```

    **注意**：如果有时候要跨库修改的话，需要使用数据库名.表名

2. 修改表选项

    ```mysql
    alter table t1 charset gbk;
    ```

#### 更改字段

* 字段操作包含字段名字、类型和属性的操作
* 字段操作通常是在表已经存在数据后进行

##### 新增字段

* 字段的新增必须同时存在字段类型

* 新增语法

    ```mysql
    alter table 表名 add [column] 字段名 字段类型 [字段属性] [字段位置]
    ```

###### 示例

1. 给已经存在的表增加一个字段

    ```mysql
    alter table t1 add age int;
    ```

##### 字段位置

* 字段位置分为两种

    * 第一个字段：`first`
    * 某个字段后：`after`已经存在字段名

* 字段位置适用于追加字段、修改字段、更改字段名

* 字段位置语法

    ```mysql
    alter table 表名 字段操作 字段位置;
    ```

###### 示例

1. 为表增加一个字段，放在最前面

    ```mysql
    alter table t1 add id int first;
    ```

2. 在表某字段后增加一个字段

    ```mysql
    alter table t1 add card varchar(18) after name;
    ```

##### 更改字段名

* 字段名的修改必须跟上字段类型

* 字段名修改语法

    ```mysql
    alter table 表名 change 原字段名 新字段名 字段类型 [字段属性] [位置];
    ```

###### 示例

修改字段名card为sfz

```mysql
alter table t1 change card sfz varchar(18);
```

##### 修改字段

* 修改字段类型、字段属性和位置

* 修改字段语法

    ```mysql
    alter table 表名 modify 字段名 字段类型 [字段属性] [位置];
    ```

###### 示例

修改sfz类型为char(18)并且把位置放到id后面

```mysql
alter table t1 modify sfz char(18) after id;
```

#### 删除字段

* 删除字段会将数据也删除

* 删除字段语法

    ```mysql
    alter table 表名 drop 字段名;
    ```

##### 示例

```mysql
alter table t1 drop age;
```

### SQL数据操作

#### 新增数据

* 新增数据是根据表的字段顺序和数据类型要求将数据存放到数据表中
* 数据表中的数据以行(row)为存储单位，实际存储属于字段(field)存储数据
* 数据插入分两种方式
    * 全字段插入：`insert into 表名 values(字段列表顺序对应的所有值)`
    * 部分字段插入：`insert into 表名(字段列表) values(字段列表对应的值的顺序列表)`

##### 示例

1. 插入完整数据

    ```mysql
    insert into t1 values(1,"666","张三","小张");
    ```

2. 根据字段插入数据

    ```mysql
    insert into t1 (id,name ) values(1,"李四");
    ```

#### 查看数据

* 查到的数据显示出来是一张二维表

* 数据显示包含字段名和字段本身

* 数据查看分两种方式

    * 查看全部字段：使用`*`代替所有字段
    * 查看部分字段：明确字段名，使用逗号分隔

* 查看数据很多时候也是根据条件查询部分数据

* 查看语法

    ```mysql
    select */字段列表 from 表名;
    ```

##### 示例

1. 查看所有数据

    ```mysql
    select * from t1;
    ```

2. 查看部分字段数据

    ```mysql
    select id,name;
    ```

3. 查看表中id为1的信息

    ```mysql
    select * from t1 where id = 1;
    ```

#### 更新数据

* 更新数据通常时根据条件更新某些数据，而不是全部记录都更新

* 更新数据语法

    ```mysql
    update 表名 set 字段 = 新值[,字段 = 新值] [where 条件筛选];
    ```

##### 示例

1. 更新所有记录的身份信息

    ```mysql
    update t1 set sfz = "777";
    ```

2. 更新某个记录的多个字段数据

    ```mysql
    update t1 set name = "张三"，sfz = "666" where id = 777;
    ```

#### 删除数据

* 删除数据是一种不可逆操作

* 数据删除通常都是有条件删除

* 数据删除语法

    ```mysql
    delete from 表名 [where 条件];
    ```

##### 示例

删除数据

```mysql
delete from t1 where id = 777;
```

