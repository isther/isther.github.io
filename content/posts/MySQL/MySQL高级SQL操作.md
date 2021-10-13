---
title: "MySQL高级SQL操作"
date: 2021-07-28T10:31:51+08:00
# weight: 1
# aliases: ["/first"]
categories: ["MySQL"]
tags: ["数据库","MySQL","蠕虫赋值","主键冲突","查询选项","别名","数据源","where子句","运算符","group by子句","回溯统计","having子句","order by子句","limit子句","限制删除","限制更新","清空数据"]
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

常见的SQL高级操作，主要集中在数据操作（增删改查），基于基础操作之上实现一些复杂业务的数据操作

<!--more-->

### 数据新增

主要是在新增数据时的高级操作技巧，提升数据插入的效率问题、安全问题。

#### 批量插入

* 批量插入分两种

    * 全字段批量插入

        ```mysql
        insert into 表名 values(值列表1),(值列表2)...(值列表n);
        ```

    * 部分字段批量插入

        ```mysql
        insert into 表名 (字段列表) values(值列表1),(值列表2)...(值列表n);
        ```

        

##### 示例

1. 批量插入学生成绩(全字段)

    ```mysql
    insert into t1 values(null,'Tom','Computer','90'),(null,'Lily','Computer','100');
    ```

2. 批量插入学生考试信息(不含成绩)

    ```mysql
    insert into t1 (stu_name,course) values('Tony','English'),('Ray','Math');
    ```

#### 蠕虫复制

* 从已有的表中复制数据直接插入到另外一张表

* 目的是快速增加表中的数据

    * 实现表中数据复制（用于数据备份和迁移）
    * 实现数据的指数级递增（多用于测试）

* 蠕虫复制语法

    ```mysql
    insert into 表名 [(字段列表)] select 字段列表 from 表名;
    ```

* 注意事项

    * 字段列表必须对应
    * 字段类型必须匹配
    * 数据冲突需要事先考虑

##### 示例

1. 创建一张新表，将t1表中的数据迁移到新表中

    ```mysql
    create table t2(
    	id int primary key auto_increment,
        stu_name varchar(20) not null,
        course varchar(20) not null,
        score decimal(5,2)
    )charset utf8;
    
    insert into t2 select * from t1;
    ```

2. 快速让t1表中的数据达到超过100条（重复执行）

    ```mysql
    insert into t2 (stu_name,course,score) select stu_name,course,score from t1;
    # 
    ```

#### 主键冲突

* 在数据进行插入时包含主键指定，而主键在数据表已经存在

* 主键冲突的业务通常是发生在业务主键上（业务主键本身有业务意义）

* 主键冲突的解决方案

    * 忽略冲突：保留原始记录

        ```mysql
        insert ignore into 表名 [(字段列表)] values(值列表);
        # 产生主键冲突后，保留原始记录，插入数据无效
        ```

    * 冲突更新：冲突后部分字段变成更新后的值

        ```mysql
        insert into 表名 [(字段列表)] values(值列表) on duplicate key update 字段 = 新值[,字段=新值...];
        # 1.尝试新增
        # 2.失败，更新
        ```

    * 冲突替换：先删除原有记录，后新增记录

        ```mysql
        replace into 表名 [(字段列表)] values(值列表);
        #效率没有insert高（需要检查是否冲突）
        ```

##### 示例

1. 用户名作为主键的用户注册（冲突不能覆盖）：username，password，regtime

    ```mysql
    create table t1(
    	`username` varchar(50) primary key,
    	`password` char(32) not null,
    	regtime int unsigned not null
    )charset utf8;
    
    insert into t1 values('username','password',1234546789);
    #冲突忽略，且数据不插入
    insert ignore into t1 values('username','password',123456789);
    ```

2. 用户名作为主键的记录用户使用信息（不存在新增，存在则更新时间）：username，logintime

    ```mysql
    create table t1(
    	`username` varchar(50) primary key,
        logintime int unsigned
    )charset utf8;
    
    insert into t1 values('username',12345678);
    
    #冲突更新（替换部分字段数据）
    insert into t1 values('username',12345678) on duplicate key update logintime = unix_timestamp();
    ```

    * 如果主键不冲突：新增
    * 如果主键冲突：更新指定字段
    * 上述方式适用于字段较多，但是可能冲突时数据变化的字段较少

3. 用户名作为主键，记录用户登录信息（不存在新增、存在则更新全部）：username、login time、clientinfo

    ```mysql
    create table t1(
    	`username` varchar(50) primary key,
    	logintime int unsigned,
    	clientinfo varchar(255) not null
    )charset utf8;
    
    replace into t1 values('username',unix_timestamp(),'{PC:chrome}');
    replace into t1 values('username',unix_timestamp(),'{phone:uc}');
    ```

    * `replace`遇到主键重复就会先删除、后新增

### 数据查询

#### 查询选项

* 用于对查询结果进行简单数据筛选
* 查询选项时在`select`关键字之后，有两个互斥值
    * `all`：默认，表示保留所有记录
    * `distinct`：去重，重复的记录（所有字段都重复）

##### 示例

查看商品表中所有品类的商品信息，重复商品只保留一次（名字、价格、属性都一致）

```mysql
create table t1(
	id int primary key auto_increment,
	goods_name varchar(50) not null,
	goods_price decimal(10,2) default 0.00,
	goods_color varchar(20),
	goods_weight int unsigned comment '重量，单位克'
)charset utf8;

insert into t1 values(null,'mate10',5499.00,'blue',320),
(null,'mate10',5499.00,'gray',320),
(null,'nokia3301',1299,'black',420);

# 考虑所有字段的去重（不含逻辑主键）
select distinct goods_name,goods_price,goods_color,goods_weight from t1;

# 不考虑颜色去重
select distinct goods_name,goods_price,goods_weight from t1;
```

#### 字段选择&别名

* **字段选择**：根据实际需求选择的要获取数据的字段信息
* 根据实际需求，明确所需要的字段名字，使用英文逗号，分隔
* 获取所有字段，使用星号`*`通配所有字段
* 字段数据可以不一定是来自数据源（select只要有结果即可）
    * 数据常量：`select 1`
    * 函数或变量：`select unix_timestamp(),@@version`(@@是系统变量的前缀，后跟变量名)
* **字段别名**：给字段取的临时名字
* 字段别名使用`as`语法实现
    * 字段名 `as` 别名
    * 字段名 别名
* 字段别名的目的通常为了保护数据
    * 字段冲突：多张表同时操作有同名字段（系统默认覆盖），想保留全部
    * 数据安全：对外提供数据不使用真实字段名字

##### 示例

1. 查询商品信息

    ```mysql
    # 全部查询
    select * from t1;
    
    # 需求为商品名字和商品
    select goods_name,goods_price from t1;
    
    # 别名使用
    select goods_name as gn,goods_price as gp from t1;
    ```

2. 不需要数据源的数据获取：select的表达式本身能算出结果

    ```mysql
    # 获取当前时间戳和版本号
    select unix_timestamp() as now,@@version as version,@@version;
    ```

#### 数据源

* 数据来源 `from`之后
* 单表数据源：数据源就是一张表 `from 表名`
* 多表数据源：数据来源是多张表（逗号分隔） `from 表名1,表名2,...`
* 子查询数据源：数据来源是一个查询结果`from (select 字段列表 from 表名) as 别名`
    * 数据源要求必须是一个表
    * 如果是查询结果必须给其一个表别名
* 数据表也可以指定别名
    * 表名 `as` 别名
    * 表名 别名

##### 示例

1. 单表数据源：最简单的数据源，直接从一个数据表获取

    ```mysql
    select * from t1;
    ```

2. 多表数据源：

    ```mysql
    select * from t1,t2;
    # 利用一张表的一条数据匹配另外一张表的所有记录
    # 记录数=表1记录数*表2记录数
    # 字段数=表1字段数+表2字段数
    ```

3. 子查询数据源：数据来源是一个select对应的查询结果

    * 查询语句需要使用括号
    * 查询结果需要指定别名

    ```mysql
    select * from (select * from t1,t2) t;
    ```

4. 如果有时候名字较长或使用不方便，可以利用表别名

    ```mysql
    select * from table1 as t1;
    
    select t1.*,t2.stu_name from table1 as t1 table2 as t2;
    ```

#### where子句

* 跟在`from`数据源之后，对数据进行条件匹配
* `where`是在磁盘读取后，进入内存之前进行筛选
    * 不符合条件的数据不会进入内存
* `where`筛选的内容因为还没进入内存，所以数据是没有被加工过的
    * 字段别名不能在`where`中使用

##### 示例

1. 查询t1表中学生名字为Lily的成绩信息

    ```mysql
    select * from t1 where stu_name = 'Lily';
    ```

2. 因为where是在磁盘取数据时进行条件筛选，此时数据没有进入内存，所以字段别名是无效的

    ```mysql
    # 错误
    select stu_name sn,score from t1 where name = 'Lily';
    ```

#### 运算符

* 比较运算符
    * `>`、`<`、`=`（即是赋值又是等于）、`>=`、`<=`、`<>`(不等于)
    * `between A and B`  ：A和B之间，包括A和B本身，数值比较
    * `in (数据1,数据2,...数据N)`：在列举的数据之中
	* `like 'pattern'`：上面这样的，用于字符串比较
		* `_`：单下划线，匹配对应位置的一个任意字符
		* `%`：匹配当前位置往后任意数量任意字符
* 逻辑运算符
    * `and(逻辑与)`、`or(逻辑或)`、`not(逻辑非)`
* null运算符
    * `is null(为空)`、`is not null(不为空)`

##### 示例

1. 查询成绩不及格的所有学生信息

    ```mysql
    select * from t1 where score < 60;
    ```

2. 查询成绩在60-90之间的学生信息

    ```mysql
    select * from t1 where between 60 ans 90;
    select * from t1 where score >= 60 and score <=90;
    ```

3. 查询没有成绩的学生

    ```mysql
    select * from t1 where score is null;
    ```

#### group by 子句

* 分组统计，根据某个字段所有的**结果分类**，并进行**数据统计分析**
* 分组的目的不是为了显示数据，一定是为了统计数据
* `group by`子句一定是出现在`where`字句之后（如果同时存在）
* 分组统计可以进行统计细分：先分大组，然后大组分小组
* 分组统计需要使用统计函数
    * `group_concat()`：将组里的某个字段全部保留
    * `any_value()`：不属于分组字段的任意一个组里的值
    * `count()`：求对应分组的记录数量
        * `count(字段名)`：统计某个字段值的数量（NULL不统计）
        * `count(*)`：统计整个记录的数量（较多）
    * `sum()`：求对应分组中某个字段的和
    * `max()/min()`：求对应分组中某个字段的最大/最小值
    * `avg()`：求对应分组中某个字段的平均值

##### 示例

1. 创建一张表，存储学生信息

    ```mysql
    create table t1(
    	id int primary key auto_increment,
        `name` varchar(10) not null,
    	`gender` enum('男','女','保密'),
        age tinyint unsigned not null,
        class_name varchar(10) not null comment '班级名称'
    )charset utf8;
    
    insert into t1 values(null,'鸣人','男',18,'木叶1班'),
    (null,'佐助','男',18,'木叶1班'),
    (null,'小樱','女',18,'木叶1班'),
    (null,'佐井','男',18,'木叶1班'),
    (null,'大蛇丸','男',28,'木叶0班'),
    (null,'卡卡西','男',29,'木叶2班'),
    (null,'雏田','女',18,'木叶1班'),
    (null,'我爱罗','男',18,'木叶1班'),
    (null,'向日葵','女',8,'木叶10班'),
    (null,'博人','男',8,'木叶10班'),
    (null,'鼬','男',28,'木叶0班'),
    (null,'带土','男',28,'木叶2班'),
    (null,'琳','女',27,'木叶2班')；
    ```

2. 统计每个班的人数

    ```mysql
    SELECT COUNT(*),class_name from t1 group by class_name;
    ```

3. 多分组：统计每个班的男女学生数量

    ```mysql
    SELECT COUNT(*),class_name,gender from t1 group by class_name,gender;
    ```

4. 统计每个班里的人数，并记录班级学生的名字

    ```mysql
    SELECT COUNT(*),group_concat(`name`) from t1 group by class_name;
    ```

#### 回溯统计

* 在进行分组时（通常是多分组），每一次结果的回溯都进行一次汇总统计
* 回溯统计语法：在统计之后使用`with rollup`

##### 示例

统计每个班的男女同学数量，同时要知道班级人数总数

```mysql
# 只统计每个班的男女同学数量，没有班级汇总
SELECT COUNT(*),class_name,gender,group_concat(`name`) from t1 group by class_name,gender;

# 汇总统计：回溯
SELECT count(*),class_name,gender,group_concat(`name`) from t1 group by class_name,gender with rollup;
```



#### 分组排序

* 在分组后统计结果，根据分组字段进行升序或者降序显示数据
* 默认系统自动升序排序
* 可以设定分组结果的排序方式
    * `group by 字段名 [ASC]`：升序
    * `group by 字段名 DESC`：降序

##### 示例

```mysql
SELECT COUNT(*),class_name,gender,group_concat(`name`),any_value(`name`) from t1 group by class_name,gender desc;
```



#### having子句

* 类似于`where`子句，是用来进行条件筛选数据的
* `having`子句本身是针对**分组统计结果进行条件筛选**的
* `having`子句必须出现在`group by`子句之后（如果同时存在）
* `having`针对的数据是在内存里已经加载的数据
* `having`几乎能做`where`能做的所有事，但是`where`却不一定
    * 字段别名（where针对磁盘数据，在处理时还没有别名）
    * 统计结果（where在group by之前）
    * 分组统计函数（having通常是针对group by存在的）

##### 示例

1. 获取班级人数小于3的班级

    ```mysql
    SELECT COUNT(*) as `count`,class_name,group_concat(`name`) from t1 group by class_name having count < 3;
    
    SELECT COUNT(*) as `count` ,class_name,group_concat(`name`) from t1 group by class_name having COUNT(*) < 3;
    # 多用了一次函数（效率降低）
    
    SELECT class_name,group_concat(`name`) from t1 group by class_name having COUNT(*) < 3;
    ```



#### order by子句

* 排序，根据某个指定的字段进行升序或者降序
* 排序的参照物是校对集
* `order by`子句在having子句字后（如果同时存在）
* 排序分为升序和降序：默认升序
    * `order by 字段 [ASC]`：升序
    * `order by 字段 DESC`：降序
* 多字段排序：在根据某个字段排序好之后，可以再细分

##### 示例

1. 单字段：按照年龄升序

    ```mysql
    select * from t1 order by age;
    select * from t1 order by age asc;
    ```

2. 多字段：先性别降序，后年龄升序

    ```mysql
    select * from t1 order by gender desc, age;
    select * from t1 order by gender desc, age asc;
    ```



#### limit子句

* 限制数据的获取数量
* `limit`子句必须在`order by`子句之后（如果同时存在）
* `limit`限制数量方式有两种
    * `limit 数量`
    * `limit 起始位置,数量`

##### 示例

1. 获取表中前3条

    ```mysql
    select * from t1 limit 3;
    ```

2. 获取表中第三条开始之后的3条数据

    ```mysql
    select * from t1 limit 3,3;
    ```

    

### 数据更新

#### 限制更新

* 更新时对更新的记录数进行限制
* 限制更新通过limit实现
* 其实是局部更新的一种手段，一般更多情况下依据条件精确更新

#### 示例

对会员选3个发送10元红包

```mysql
CREATE table t1(
    id int primary key auto_increment,
    `username` varchar(50) not null unique,
    `password` char(12) not null,
    account decimal(10,2) default 0.00
)charset utf8;

INSERT INTO t1 VALUES(null,'username1','password',default),
(null,'username2','password',default),
(null,'username3','password',default),
(null,'username4','password',default),
(null,'username5','password',default),
(null,'username6','password',default),
(null,'username7','password',default);

UPDATE t1 set account = account + 10 LIMIT 3;

SELECT * from t1;
```

### 数据删除

#### 限制删除

* 限制要删除的记录数
* 使用`limit`限制
* 一般很少使用，通常使用`where`条件精确删除

##### 示例

删除没有账户余额的一个用户

```mysql
delete from t1 where account = 0 limit 1;
```

#### 清空数据

* 将表中的数据清楚，表的所有状态回到原始状态
* 本质是先删除，后创建
* 清空后可以使表的一些变化状态回到原始状态，例如自增长回归初值
* 清空语法：`truncate 表名`

##### 示例

清空用户数据表

```mysql
truncate t1;
```

