---
title: "MySQL多表操作"
date: 2021-07-29T20:02:51+08:00
# weight: 1
# aliases: ["/first"]
categories: ["MySQL"]
tags: ["数据库","MySQL","多表操作","联合查询","连接查询","交叉连接","内连接","外连接","自然连接","using","子查询","标量子查询","列子查询","行子查询","表子查询","exists子查询"]
# author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "简单介绍了一下MySQL多表操作"
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

**多表**：因为单表会出现数据冗余，所以采用多表的方式

<!--more-->

### 联合查询

#### 联合查询

* `union`，是指将多个查询结果合并成一个结果显示

* 联合查询是针对查询结果的合并（多条select语句合并）

* 联合查询语法

    ```mysql
    select 查询 # 决定字段表
    	union 查询选项
    select 查询
    ...
    ```

* 联合查询要求：联合查询是结果联合显示

    * 多个联合查询的字段结果数量一致
    * 联合查询的字段来源于第一个查询语句的字段

* 查询选项：与`select`选项一样

    * `all`：保留所有记录
    * `distinct`：保留去重记录（默认）

##### 示例

1. 创建一个表，并插入数据

    ```mysql
    create table t2 like t1;
    
    insert into t2 values(null, '犬夜叉', '男', 200, '神妖1班'),
      (null, '日暮戈薇', '女', 16, '现代1班'),
      (null, '桔梗', '女', 88, '法师1班'),
      (null, '弥勒', '男', 28, '法师2班'),
      (null, '珊瑚', '女', 20, '法师2班'),
      (null, '七宝', '保密', 5, '宠物1班'),
      (null, '杀生丸', '男', 220, '神妖1班'),
      (null, '铃', '女', 4, '现代1班'),
      (null, '钢牙', '男', 68, '神妖1班'),
      (null, '奈落', '男', 255, '神妖1班'),
      (null, '神乐', '女', 15, '神妖2班');
    ```

    	t1与t2结构一致，可以理解为因为数据量较大，拆分到两个表中
    
2. 使用联合查询两张表的数据拼接到一起显示

    ```mysql
    select * from t1
    union
    select * form t2;
    ```

3. 联合查询选项默认是`distinct`

    ```mysql
    select * from t1
    union
    select * from t2;
    
    select * from t1
    union all
    select * from t2;
    ```

4. 联合查询不要求字段类型一致，只对数量要求一致，而且字段名称与第一条查询语句相关

    ```mysql
    select name from t1
    union
    select age from t2;
    ```

    如果数据不能与字段对应，那么查询没有意义

    

  #### 联合查询排序

* 针对联合查询的结果进行排序
* `order by`本身是对内存结果进行排序，`union`的优先级高于`order by`，所以`order by`默认是对`union`结果进行排序
* 如果想对单独`select`的结果进行排序，需要两个步骤
    * 将需要排序的`select`指令进行括号包裹（括号使用order by）
    * `order by`必须配合`limit`才能生效（limit一个足够大的数值即可）

##### 示例

1. 将t1和t2表的结果使用年龄降序排序

    ```mysql
    select * from t1
    union all
    select * from t2
    order by age desc;
    ```

2. t1表按年龄降序排序，t2表按年龄升序排序

    ```mysql
    # 无效方式
    (select * from t1 order by age desc)
    union
    (select * from t1 order by age asc);
    
    # 有效方式
    (select * from t1 order by age desc limit 9999)
    union
    (select * from t2 order by age asc limit 9999);
    ```

### 连接查询

#### 交叉连接

* `cross join`，不需要任何条件的连接
* 交叉连接产生的效果就是笛卡尔积
    * 左表的每一条记录都会与右表的所有记录连接并保存
* 交叉连接没有实际数据价值，只是丰富了连接查询的完整性

##### 示例

```mysql
# 交叉连接t1和t2表
select * from t1 cross join t2;
```

#### 内连接

* `[inner] join`，将两张表根据指定的条件连接起来，严格连接
* 内连接是将一张表的每一个记录去另外一张表根据条件匹配
    * 匹配成功：保留连接的数据
    * 匹配失败：都不保留
* 内连接语法：`左表 join 右表 on 连接条件`

##### 示例

1. 设计学生表和专业表：学生对专业多对一关系

    ```mysql
    # 学生表
    create table t1(
    	id int primary key auto_increment,
    	name varchar(50) not null,
    	course_no int
    )charset utf8;
    
    insert into t1 values(null,'student1',1),
    (null,'student2',1),
    (null,'student3',2),
    (null,'student4',3),
    (null,'student5',1),
    (null,'student6',default);
    
    # 专业表
    create table t2(
    	id int primary key auto_increment,
    	name varchar(50) not null unique
    )charset utf8;
    
    insert into t2 values(null,'Computer'),(null,'Software'),(null,'Network');
    ```

2. 获取已经选择了专业的学生信息，包括所选专业

    ```mysql
    # 学生和专业在两个表中，所以需要连表
    # 学生必须有专业，而专业也必须存在，所以是内连接
    # 连接条件：专业编号
    # 两张表有两个字段冲突：id,name,所以需要使用别名
    select t1.*,t2.name as course_name from t1 inner join t2 on t1.course_no = t2.id;
    
    # 表名的使用也可以使用别名
    select s.*,c.name as c_name from t1 as s inner join t2 c on s.course_no = c.id;
    ```

#### 外连接

* `outer join`，是一种不严格的连接方式
* 外连接分为两种
    * 左连接：`left join`
    * 右连接：`right join`
* 外连接有主表和从表之分
    * 左连接：左表为主表
    * 右连接：右表为主表
* 外连接是将主表的记录去匹配从表的记录
    * 匹配成功保留
    * 全表匹配失败：也保留，只是从表字段置空

##### 示例

1. 查出学生所有信息，包括所在班级（左连接）

    ```mysql
    # 主要数据是学生，而且是全部学生：外连接、且学生表为主表
    select s.*,c.name c_name from t1 s left join t2 c on s.course_no = c.id;
    ```

2. 查出所有班级里的所有学生

    ```mysql
    #主表是班级
    select s.*,c.name c_name from t1 s right join t2 c on s.course_no = c.id;
    ```

#### 自然连接

* `natural join`，是一种自动寻找连接条件的连接查询
* 自然连接不是一种特殊的连接方式，而是自动匹配条件的连接
* 自然连接包含
    * 自然内连接：`natural join`
    * 自然外连接：`natural left/right join`
* 自然连接条件匹配模式：自动寻找相同字段名作为连接条件

##### 示例

1. 自然连接t1和t2表

    ```mysql
    select * from t1 natural join t2;
    ```

2. 自然连接是不管字段是否有关系，只管名字是否相同，如果想要自然连接成功，那么字段的设计就必须非常规范

    ```mysql
    create table t11(
    	s_id int primary key auto_increment,
    	s_name varchar(50) not null,
    	c_id int comment '课程id'
    )charset utf8;
    insert into t11 select * from t1;
    
    create table t22(
    	c_id int primary key auto_increment,
    	c_name varchar(50) not null unique
    )charset utf8;
    insert into t22 select * from t2;
    
    # 自然连接 成功
    select * from t11 natural join t22;
    ```

#### using 关键字

* 连接查询时，如果是同名字段作为连接条件，`using`可以代替`on`出现，且比`on`更好
    * `using`是针对同名字段（using(id) === A.id = B.id）
    * `using`关键字使用后会自动合并对应字段为一个
    * `using`可以同时使用多个字段作为条件

##### 示例

查询t11中所有学生信息，包括所在班级名字

```mysql
select s.*,c.c_name from t11 s left join t22 c using(c_id);
select * from t11 s left join t22 c using(c_id);
```

### 子查询

#### 子查询分类

* 根据子查询**出现的位置**或者**产生的数据效果**分类
    * 位置分类
        * `from`子查询：子查询出现在from后做数据源
        * `where`子查询：子查询出现在where后做数据条件
    * 按子查询得到的结果分类
        * 标量子查询：子查询返回的结果是一行一列（一个数据）
        * 列子查询：子查询返回的结果是一列多行（一列数据）
        * 行子查询：子查询返回的结果是一行多列（一行数据）
        * 表子查询：子查询返回的结果是一个二维表
        * `exists`子查询：子查询返回的结果是布尔结果（验证型）
* 子查询都需要使用括号`()`包裹，必要时需要对子查询结果进行别名处理（from子查询）

#### 标量子查询

* 通常是用来做其他查询的条件

##### 示例

获取computer专业的所有学生

```mysql
# 数据目标：学生表t11
# 条件：专业名字，不在t11中

select * from t11 where c_id = (select c_id from t_22 where c_name = 'Computer');
```

#### 列子查询

* 通常是用来做查询条件的

##### 示例

获取所有有学生的班级信息

```mysql
# 数据获取目标是班级信息
# 数据获取条件是在学生表中的班级id，是多个

select * from t22 where c_id in (select c_id from t11);
```

#### 行子查询

* 子查询返回的结果是一行多列
* 行子查询需要条件中构造元素
    * `(元素1),(元素2),...(元素N)`
* 行子查询通常也是用来作为主查询的条件

##### 示例

获取学生表中性别和年龄都和弥勒相同的学生信息

```mysql
# 查询条件有多个：性别和年龄
# 数据的条件的来源在另一张表中

# 解决思路：两个标量子查询
select * from t1 where gender = (select gender from t2 where name = '弥勒') and age = (select from t2 where name = '弥勒');
```

* 以上查询方式使用了两次子查询，效率降低

```mysql
# 构造条件行元素(gender,age)
select * from t1 where (gender,age) = (select gender,age from t2 where name = '弥勒');
```

#### 表子查询

* 表子查询多出现在`from`之后，当作数据源
* 表子查询通常是为了想对数据进行一次加工处理，然后再交给外部进行二次加工处理

##### 示例

获取学生表中每个班级里年龄最大的学生信息，然后按年龄降序排序显示

```mysql
# 尝试直接解决
select any_value(name),max(age) m_age, clas_name from t1 group by class_name order by m_age desc;
```

* 分组统计中`any_value()`取的是分组后的第一条数据，而需要的是最大

```mysql
# 解决方案：要入在分组之前将所有班级里的学生本身是降序排序，那么分组的第一条数据就是满足条件的数据，但是问题是order by必须出现在group by之后
select any_value(name),max(age),class_name from (select name,age,class_name from t1 order by age desc) as t group by class_name;
```

* 依然无效，原因是MySQL7之后若要子查询中``order by`生效，需要像联合查询一样，加上`limit`

```mysql
select any_value(name),max(age),class_name from (select name,age,class_name from t1 order by age desc limit 99999) as t group by class_name;
```

#### exists子查询

* `exists`子查询通常是作为`where`条件使用
    * `where exists(子查询)`

##### 示例

获取所有学生的班级信息

```mysql
# 获取的数据是班级表t22
# 班级是否有学生需要在t11中确认，并不需要t11提供任何数据显示
select * from t22 c where exists(select c_id from t11 where c.c_id = c_id);
```



#### 比较方式

* 特定的比较方式都是基于比较符号一起使用的
* `all`：满足后面全部条件
    * `>all(结果集)`：数据要大于结果集中的全部数据
* `any`：满足任何条件
    * `=any(结果集)`：数据只要与结果集中的任何一个元素相等
* `some`：满足任意条件(与any完全一样)
* 结果集：可以是直接的数据也可以是子查询的结果（通常是列子查询）

##### 示例

找出t1表中与t2表中年龄相同的信息

```mysql
# 数据获取在t1表
# 数据条件在t2表

# 解决方案1：使用in列子查询
select * from t1 where age in (select distinct age from t2);

# 解决方案2：使用exists子查询
select * from t1 where exists(select id from t2 where t1.age = age);

# 解决方案3：使用any或者some匹配（列子查询）
select * from t1 where age = some(select age from t2);
```



