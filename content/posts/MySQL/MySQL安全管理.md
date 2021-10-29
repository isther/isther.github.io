---
title: "MySQL安全管理"
date: 2021-07-30T10:02:51+08:00
# weight: 1
# aliases: ["/first"]
categories: ["MySQL"]
tags: ["数据库","MySQL","外键约束","事务安全","预处理","视图","数据备份与还原"]
# author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "简单介绍了一下数据库"
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

结构->执行->管理->用户

### 外键约束

**概念**：`foreign key`，表中**指向外部主键**的字段

* 外键必须要通过语法指定才能称之为外键
    * `[constraint 外键名] foreign key (当前表字段名) references 外部表(主键字段)`
* 外键构成条件
    * 外键字段必须与对应表的主键字段类型一致
    * 外键字段本身要求是一个索引（创建外键会自动生成一个索引）

<!--more-->

##### 示例

1. 创建专业表和学生表，学生表中的专业id指向专业表id

    ```mysql
    # 专业表
    CREATE table t1 (
    	id int primary key AUTO_INCREMENT,
    	name varchar(50) not null unique
    ) charset utf8;
    
    # 学生表
    create table t2(
    	id int primary key AUTO_INCREMENT,
    	name varchar(50) not null,
    	c_id int comment '指向专业表t1的主键'
    	constraint `c_id` FOREIGN KEY(c_id) REFERENCES t1(id)
    ) charset utf8;
    ```

    外键可以不指定名字，系统会自动生成

#### 外键约束

* 当表建立外键关系后，外键就会对主表和子表里的数据产生约束效果
* 外键约束的是写操作（默认操作）
    * 新增：子表插入的数据对应的外键必须在主表存在
    * 修改：主表的记录如果在子表存在，那么主表的主键不能修改（其他字段可修改）
    * 删除：主表的记录如果在子表存在，那么主表的主键不能删除
* 外键约束控制：外键可以定义时控制外键的约束作用
    * 控制类型
        * `on update`：父表更新时子表的表现
        * `on delete`：父表删除时子表的表现
    * 控制方式
        * `cascade`：级联操作，父表操作后自编跟随操作
        * `set null`：置空操作，父表操作后，子表关联的外键字段置空
        * `restrict/no action`：严格模式，不允许父表操作（默认）

##### 示例

1. 子表不能插入主表不存在的数据

    ```mysql
    insert into t2 values(null,'Tony',2);# 此时父表中还没有id=2的记录，错误
    
    insert into t1 values(null,'English');
    insert into t2 values(null,'Peny',1);
    ```

2. 默认的外键产生后，主键不能更新被关联的主键字段或者删除被关联的主键记录

    ```mysql
    # 错误
    update t1 set id = 2;
    delete from t1 where id = 1;
    ```

3. 限制外键约束，一般使用更新级联，删除置空

    * `on update cascade`：更新级联
    * `on delete set null`：删除置空

    ```mysql
    create table t3(
        id int primary key AUTO_INCREMENT,
        name varchar(50) not null unique
    )charset utf8;
    
    create table t4(
        id int primary key AUTO_INCREMENT,
        name varchar(50) not null,
        c_id int , # 如果允许置空，就不能not null
        FOREIGN KEY(c_id) references t3(id) on update cascade on delete set null
    )charset utf8;
    
    insert into t3 values(null,'Chinese'),(null,'Computer');
    insert into t4 values(null,'Tony',1),(null,'Lily',2);
    ```

    * 子表依然不允许插入父表不存在的外键
    * 但是可以插入外键为Null的数据

    ```mysql
    # 错误
    insert into t2 values(null,'Lilei',3);
    
    insert into t2 vlaues(null,'Lilei',null);
    ```

    * 父表的更新（主键）会让关联的外键自动级联更新

    ```mysql
    update t1 set id  = 3 where id = 1;
    ```

#### 外键管理

* 在表创建后期维护外键

* 新增外键

    ```mysql
    alter table 表名 add [constraint `外键名`] foreign key(外键字段) references 表名(主键) [on 外键约束]
    ```

* 删除外键

    ```mysql
    alter table 表名 drop foreign key 外键名;
    ```

* 更新外键：先删除后新增

### 事务安全

**事务**：要做的某个事情

* 计算机中的事务是指某个程序执行单元（写操作）
* 事务安全：当事务执行后，保障事务的执行是有效的，而不会导致数据错乱
* 事务安全通常针对的是一连串的操作（多个事务）而产生的统一结果

* MySQL中默认的写操作是直接写入的
    * 执行写操作SQL
    * 同步到数据表

##### 示例

银行转账

```mysql
create table t1(
	id int primary key auto_increment,
	`name` varchar(50) not null,
	account decimal(10,2) default 0.00
)charset utf8;

insert into t1 values(null,'Tom',10000),(null,'Lucy',100);

# Tom扣钱
update t1 set account = account - 100 where id = 1;
# Lucy收钱
update t1 set account = account + 1000;
```

* 扣钱，收钱两步都成功转账才叫成功
* 为了保障两步都成功才能叫事务安全

#### 事务处理

* 利用手动或者手动方式实现事务管理
* 自动事务处理：系统默认，操作结束直接同步到数据表（事务关闭状态）
    * 系统控制：变量`autocommit`（值为ON，自动提交）
* 手动事务处理
    * 开启事务：`start transaction`
    * 关闭事务：
        * 提交事务：`commit`，同步到数据表，同时清空日志数据
        * 回滚事务：`rollback` 只清空日志数据
* 事务回滚：在长事务执行中，可以在某个已经成功的节点设置回滚点，后续回滚的可以回到某个成功点
    * 设置回滚点：`savepoint 回滚点名字`
    * 回滚到回滚点：`rollback to 回滚点名字` 

##### 示例

1. 手动事务：启用事务转账，成功提交事务

    ```mysql
    # 开启事务
    start transaction;
    
    # Tom扣钱
    update t1 set account = account - 1000;
    
    # Lucy收钱
    update t1 set account = account + 1000;
    
    # 提交事务
    commit;
    ```

2. 手动事务：启用事务转账，成功提交事务（回滚点）

    ```mysql
    # 开启事务
    start transaction;
    
    # Tom扣钱
    update t1 set account = account - 1000 where id = 1;
    
    # 设置回滚点
    savepoint spl;
    
    # Lucy收钱
    update t1 set account = account + 1000 where id = 2;
    
    # 操作失败回到回滚点
    rollback spl;
    
    update t1 set account = account - 1000 where id = 2;
    
    # 提交事务
    commit;
    ```

3. 自动事务

    * MySQL默认自动提交事务，事务一旦发生就会立即写入数据表

    ```mysql
    show variables like 'autucommit';
    ```

    * 关闭自动提交事务（当前设置级别用户级：当前用户当此连接有效）

    ```mysql
    set autocommit = 0;
    ```

    * 手动提交事务

    ```mysql
    insert into t1 values(null,'Liu',1000);
    commit;
    ```

#### 事务特点

* ACID四大特性
* 原子性(Atomicity)：一个事务操作是一个整体，不可拆分，要么都成功，要么都失败
* 一致性(Consistency)：事务执行之前和执行之后都必须处于一致性状态，数据的完整性没有被破坏（事务逻辑的准确性）
* 隔离性(Isolation)：事务操作过程中，其他事务不可见
* 持久性(Durability)：事务一旦提交，结果不可改变

### 预处理

**预处理**：prepare statement，一种预先编译SQL指令的方式

* 预处理不同于直接处理，是将要执行的SQL指令先发送给服务器编译，然后通过指令执行
    * 发送预处理：`prepare 预处理名字 from '要执行的SQL指令'`
    * 执行预处理：`execute 预处理名字`
* 预处理管理
    * 预处理属于会话级别：即当前用户当次连接有效（断开会被服务器清理掉）
    * 删除预处理：`deallocate | drop 预处理名字`

##### 示例

查询学生的SQL指令需要重复执行很多次

```mysql
# 普通操作
select * from t1;

# 预处理操作：发送预处理
prepare pl from 'select * from t1';

# 预处理操作：执行预处理
execute pl;
 
# 删除预处理
deallocate | drop pl;
```

#### 预处理传参

* 在执行预处理的时候传入预处理需要的可变数据

* 一般预处理都不会是固定死的SQL指令，而是具有一些数据可变的执行（条件）

    * 可变数据的位置使用占位符`?`占位

        ```mysql
        prepare 预处理名字 from '预处理指令  变化部分使用?代替'
        ```

* 在执行预处理的时候将实际数据传进去代替占位符执行SQL

    * 数据存储到变量（预处理传入的值必须是变量保存的）

        ```mysql
        set @变量名 = 值
        ```

    * 使用using关键字传参

        ```mysql
        execute 预处理名字 using @变量名
        ```

    * 数据传入的顺序与预处理中占位符的顺序一致

##### 示例

向表中插入数据

```mysql
# 准备预处理：涉及参数
prepare t1 insert from 'insert into t1 values(null,?,?,?,?)';

# 设置变量并传入参数
set @name = '药师兜';
set @gender = '男';
set @age = 23;
set @class_name = '木叶1班';

#执行预处理
execute t1 insert using @name,@gender,@age,@@class_name;
```



### 视图

**视图**：view，一种由select指令组成的虚拟表

* 视图时虚拟表，可以使用表管理（结构管理）

    * 为视图提供数据的表叫做基表
```mysql
# 创建视图
create view 视图名字 as select指令;

# 访问视图：一般都是查询
select */字段名 from 视图名字;
```

* 视图有结构，但不存储数据
    * 结构：select选择的字段
    * 数据：访问视图时执行的select指令

* 对外部系统提供数据支撑(保护基表数据)

##### 示例

1. 需要对外提供一个学生详情的数据，经常使用，可以利用视图实现

    ```mysql
    # 对外提供数据，要保护数据本身的安全
    # 需要长期使用
    
    # 创建视图
    create view v_student_info as select * from t1 left join t2 using(c_id);
    
    # 使用视图：像表一样使用
    select * from v_student_info;
    ```

2. 有些复杂的SQL又是经常用到的，如多张表的连表操作：可以利用视图实现

    ```mysql
    # 院系表
    create table t1(
    	id int primary key auto_increment,
    	`name` varchar(50) not null
    )charset utf8;
    
    insert into t1 values(null,'语言系'),(null,'考古系');
    
    # 专业表
    create table t2(
    	id int primary key auto_increment,
    	`name` varchar(50) not null,
    	s_id int not null comment '学院id'
    )charset utf8;
    
    insert into t2 values(null,'English',1),(null,'Chinese',1);
    
    # 学生表
    create table t3(
    	id int primary key auto_increment,
    	`name` varchar(50) not null,
    	s_id int not null comment '专业id'
    )charset utf8;
    
    insert into t3 values(null,'Lilei',2),(null,'Hanmeimei',2),(null,'Tony',1);
    
    
    # 获取所有学生的明细信息
    select stu.*,sub.name as sub_name,sub.s_id as sch_id,sch.name as sch_name from t3 as stu left join t2 sub on stu.s_id = sub.id left join t1 sch on sub.s_id = sch.id;
    
    # 以视图保存这类复杂指令，后续可以直接访问视图
    create view v_student_detail as select stu.*,sub.name as sub_name,sub.s_id as sch_id,sch.name as sch_name from t3 as stu left join t2 sub on stu.s_id = sub.id left join t1 sch on sub.s_id = sch.id;
    ```

#### 视图管理

* 视图查看：显示视图结构和具体视图信息

    ```mysql
    show tables; #查看全部视图
    show create table/view 视图名字; # 查看视图创建指令
    desc 视图名字; # 查看视图结构
    ```

* 视图修改：更改视图逻辑

    ```mysql
    alter view 视图名 as 新的查询命令;
    create or replace view 视图名 as 新的查询命令 # 创建新的或者替换新的
    ```

* 删除视图

    ```mysql
    drop view 视图名;
    ```

#### 视图的数据操作

* 视图所有的数据操作都是最终对基表的数据操作

* 视图操作条件

    * 多基表视图：不允许操作

    * 单基表视图：允许增删改

        * 新增条件：视图的字段必须包含基表中所有不允许为空的字段

    * `with check option`：操作检查规则

        * 默认不需要这个规则（创建视图时指定）：视图操作只要满足前面上述条件即可
        * 增加此规则：视图的数据操作后，必须要保证该视图还能把通过视图操作的数据查出来（否则失败）

        

    ##### 示例

1. 增加一个单表视图和多表视图

    ```mysql
    create view v_student_1 as select s_id,s_name from t1;
    create view v_student_2 as select s.*,c.c_name from t1 s left join t2 c using(c_id);
    create or replace view v_student_3 as select * from t1 where c_id is not null with check option;
    ```

2. 新增数据

    ```mysql
    insert into v_student_1 values(null,'student7'); # 正确：视图包含所有必有字段
    insert into v_student_2 values(null,'student8',null,null) # 错误：不可插入
    insert into v_student_3 values(null,'student8',null) # 错误：check option，因为第三个字段c_id为null,不符合视图筛选条件，查不出来
    insert into v_student_3 valeus(null,'student9',1) # 正确
    ```

3. 更新数据

    ```mysql
    update v_student_1 set s_name = 'boy' where s_id = 8;
    update v_student_2 set s_name = 'boy' where s_id = 7; # 错误：不可修改 
    update v_student_3 set c_id = null where s_id = 1; # 错误：check option，修改后c_id为null，变得不符合视图筛选条件
    update v_student_3 set s_name = 'boy' where s_id = 1;
    ```

4. 删除数据

    ```mysql
    delete from v_student_1 where s_id = 2;
    delete from v_student_2 where s_id = 3; # 错误：不可修改
    delete from v_student_3 where s_id = 1; # 可以删除，check option不影响删除操作
    ```

#### 视图算法

* 视图在执行过程中对于内部的`select`指令的处理方式

* 视图算法在创建视图时指定

    ```mysql
    create ALGORITHM = 算法 view 视图名字 as select指令;
    ```

* 视图算法一共有三种

    * `undefined`：默认的，未定义算法，即系统自动选择算法
    * `merge`：合并算法，就是将视图外部查询语句跟视图内部select语句合并执行，效率高（系统优先选择）
    * `temptable`：临时表算法，即系统将视图的select语句查出来先得出一张临时表，然后外部再查询（temptable算法视图不允许写操作）

### 数据备份与还原

#### 表数据备份

* 单独针对表里的**数据部分**进行备份（数据导出）
* 将数据从表中查出，按照一定格式存储到外部文件
    * 字段格式化：`fields`
        * `terminated by`：字段数据结束后使用的符号，默认是空格
        * `enclosed by`：字段数据包裹，默认什么都没有
        * `escaped by`：特殊字符的处理，默认是转义
    * 行格式化：`lines`
        * `terminated by`：行结束符号，默认是\n，自动换行
        * `starting by`：行开始符号，默认没有

```mysql
select 字段列表|* into outfile '外部文件路径'
	[fields terminated by 格式 enclosed by 格式]
	[lines terminated by 格式 starting by 格式]
from 数据表;
```

* 表数据备份不限定数据的来源是一张表还是多张表（可以连表）

#### 表数据还原

* 将**符合数据结构**的数据导入到数据表中（数据导入）
* 将一定格式的数据按照一定的解析方式解析成符合表字段格式的数据导入到数据表
    * 字段处理
    * 行处理

```mysql
load data infile '数据文件所在路径' into table 表名
	[fields terminated by 格式 enclosed by 格式]
	[lines terminated by 格式 starting by 格式]
	[(字段列表)];
```

* 数据文件来源
    * 表数据备份的数据文件
    * 外部获取或者制作的符合格式的数据

   

#### 文件备份

* 直接对数据表进行文件保留，属于物理备份
* 文件备份操作简单，直接将数据表（或者数据库文件夹）进行保存迁移
* MySQL中不同表存储引擎产生的文件不一致，保存手段也不一致
    * InnoDB：表结构文件再ibd文件夹中，数据和索引存储在外部统一的ibdata文件夹中
    * MyIsam：每张表的数据、结构和索引都是独立文件，直接找到三个文件迁移即可

#### 文件还原

* 利用备份的文件，替换出现问题的文件，还原到备份前的良好状态
* 直接将备份文件放到相应位置即可
* 文件还原影响
    * InnoDB：单表结构，整库数据，知识和整库备份还原，否则会影响其他InnoDB存储表
    * MyIsam：单表备份，单表还原，不影响其他任何数据

​    
