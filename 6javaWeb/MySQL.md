# 单表设计
## DDL
**DDL英文全称是Data Definition Language，数据定义语言，用来定义数据库对象(数据库、表)。**
### 数据类型
#### 数值类型
![[Pasted image 20230726153339.png]]
#### 字符串类型
![[Pasted image 20230726153459.png]]
char(10)：最多只能存10个字符，不足10个字符占用10个字符空间 性能高，浪费空间
varchar(10)：最多只能存10个字符，不足10个字符，按照实际长度存储 性能低，节省空间
确定字符串长度用varchar，不确定用char
#### 日期类型
![[Pasted image 20230726153803.png]]
#### 根据页面原型/需求创建表
通过页面原型和需求获取原型字段（选择合适的类型和约束）
加上基础字段如id、create_time、update_time等
其中create_time记录当前这条数据插入的时间update_time记录当前这条数据最后更新的时间
最终获得合理的表结构

### 数据库操作
查询所有数据库: show databases;
查询当前数据库: select database();
使用数据库:use 数据库名;
创建数据库:create database \[ if not exists ] 数据库名;
删除数据库:drop database \[ if exists ] 数据库名;
注意：上述语法中的database，也可以替换成schema。如: create schema db01;

### 表操作
#### 创建
![[Pasted image 20230726152908.png]]
约束是作用于表中字段上的规则，用于限制存储在表中的数据，
目的是保证数据库中数据的正确性、有效性和完整性，
分为not null、unique、primary key（auto_increment自增）、default、foreign key
![[Pasted image 20230726153144.png]]
#### 查询
查询当前数据库所有表: show tables;
查询表结构:desc表名;
查询建表语句: show create table表名;
#### 修改
添加字段: alter  table  表名  add  字段名类型(长度)  \[comment注释]  \[约束];
修改字段类型: alter  table  表名  modify  字段名  新数据类型(长度);
修改字段名和字段类型: alter  table  表名  change  旧字段名  新字段名  类型(长度)  \[comment注释]  \[约束];
删除字段: alter  table  表名  drop  column  字段名;
修改表名:rename  table  表名  to  新表名;
![[Pasted image 20230726154822.png]]
#### 删除
删除表:drop table \[ if exists ]表名;
删除表时，表中的全部数据也会被删除

## DML
**DML英文全称是Data Manipulation Language(数据操作语言)，用来对数据库中表的数据记录进行增、删、改操作。**
### INSERT(增加)
![[Pasted image 20230726155221.png]]
![[Pasted image 20230726155347.png]]
调用now( )为create_time和update_time赋值
注意：
1.插入数据时，指定的字段顺序需要与值的顺序是一一对应的。
2.字符串和日期型数据应该包含在引号中。
3.插入的数据大小，应该在字段的规定范围内。
### UPDATE（修改）
![[Pasted image 20230726155611.png]]
![[Pasted image 20230726155637.png]]
修改表的条件可以有，也可以没有，没有条件会修改整张表的所有数据
### DELETE(删除)
![[Pasted image 20230726155858.png]]
![[Pasted image 20230726155944.png]]
1.DELETE语句的条件可以有，也可以没有，如果没有条件，则会删除整张表的所有数据。
2.DELETE 语句不能删除某一个字段的值(如果要操作，可以使用UPDATE，将该字段的值置为NULL)。


## DQL
**DQL英文全称是Data Query Language(数据查询语言)，用来查询数据库表中的记录。**
![[Pasted image 20230726160258.png]]
### 基本查询
![[Pasted image 20230726160313.png]]
其中查询所有字段使用通配符虽然比较简洁，但是性能比较低，不推荐，建议用将表中的所有字段列出的方式查询
### 条件查询
![[Pasted image 20230726160601.png]]
### 聚合函数
![[Pasted image 20230726160719.png]]
![[Pasted image 20230726160726.png]]
count的三种使用方式：
1.conut（字段）字段必须是非空的，因为聚合函数不对null值进行运算
2.count（任意不为null的常量）
3.count（\*）推荐使用，mysql底层有优化
### 分组查询
![[Pasted image 20230726161051.png]]
where与having区别：
1．执行时机不同: where是分组之前进行过滤，不满足where条件，不参与分组;而having是分组之后对结果进行过滤。
2.判断条件不同:where不能对聚合函数进行判断，而having可以。
注意：
1.分组之后，查询的字段一般为聚合函数和分组字段，查询其他字段无任何意义。
2.执行顺序: where >聚合函数>having 。

### 排序查询
![[Pasted image 20230726161302.png]]
排序方式：ASC升序（默认）、DESC降序
如果是多字段排序，当第一个字段值相同时，才会根据第二个字段进行排序
### 分页查询
![[Pasted image 20230726161439.png]]
注意：
1.起始索引从0开始，起始索引=（查询页码–1)  * 每页显示记录数。
2.分页查询是数据库的方言，不同的数据库有不同的实现，MySQL中是LIMIT。
3.如果查询的是第一页数据，起始索引可以省略，直接简写为limit 10。
### DQL小结
![[Pasted image 20230726161617.png]]

# 多表设计
项目开发中，在进行数据库表结构设计时，会根据业务需求及业务模块之间的关系，分析并设计表结构，由于业务之间相互关联，所以各个表结构之间也存在着各种联系
## 一对多
在数据库表中多的一方，添加字段，来关联一的一方的主键
为了避免数据的不完整、不一致问题，这个字段定义为多的一方的外键
![[Pasted image 20230726162008.png]]
![[Pasted image 20230726162030.png]]
## 一对一
关系:一对一关系，多用于单表拆分，将一张表的基础字段放在一张表中，其他字段放在另一张表中，以提升操作效率
实现:在任意一方加入外键，关联另外一方的主键，并且设置外键为唯一的(UNIQUE)
## 多对多
关系:一个学生可以选修多门课程，一门课程也可以供多个学生选择
实现:建立第三张中间表，中间表至少包含两个外键，分别关联两方主键
# 多表查询
**从多张表中查询数据**
## 连接查询
### 内连接
![[Pasted image 20230726162420.png]]
### 外连接
![[Pasted image 20230726162449.png]]
左外连接:查询左表所有数据(包括两张表交集部分数据)
右外连接:查询右表所有数据(包括两张表交集部分数据)
左外连接和右外连接可以互相转换，因此多用左外连接
![[Pasted image 20230726162608.png]]

## 子查询
![[Pasted image 20230726162642.png]]
分析时建议一步步查询，最后再将变量替换即可
### 标量子查询
![[Pasted image 20230726162907.png]]
### 列子查询
![[Pasted image 20230726162916.png]]
```
//select id from tb_dept where name ='教研部' or name = '咨询部'
//select * from tb_emp where dept_id in(3,2)

select * from tb_emp where dept_id in (select id from tb_dept where name ='教研部' or name = '咨询部'); 
```
### 行子查询
![[Pasted image 20230726163250.png]]
```
//select entrydate,job from tb_emp where name = '韦一笑';
//select * from tb_emp where entrydate = "2007-01-01' and job=2;
//select * from tb_emp where (entrydate,job) = ("2007-01-01',2);
select * from tb_emp where (entrydate,job) = (select entrydate,job from tb_emp where name = '韦一笑');
```
### 表子查询
![[Pasted image 20230726163732.png]]
```
select e.*,d.name from (select * from tb_emp where entrydate >'2006-01-01') e,tb_dept d where e.dept_id = d.id;
```
# 事务和索引
## 事务
一组操作的集合，这组操作要么全部成功，要么全部失败

![[Pasted image 20230726163756.png]]
![[Pasted image 20230726163819.png]]
## 索引
索引时帮助数据库高效获取数据的数据结构，在mysql中，默认使用的是B+树（多路平衡搜索树）
没有索引，查询数据需要进行全表扫描，在数据量大时效率极低；通过索引，可以大幅缩短查询所需的时间
### 优点
提高数据查询的效率，降低数据库的lO成本。
通过索引列对数据进行排序,降低数据排序的成本，降低CPU消耗。
### 缺点
索引会占用存储空间。
索引大大提高了查询效率，同时却也降低了insert、update、delete的效率。
### 语法
![[Pasted image 20230726164532.png]]
主键字段，在建表时，会自动创建主键索引。
添加唯一约束时，数据库实际上会添加唯一索引。
### B+树的特点
1.每一个节点，可以存储多个key(有n个key，就有n个指针)。
2.所有的数据都存储在叶子节点,非叶子节点仅用于索引数据。
3.叶子节点形成了一颗双向链表,便于数据的排序及区间范围查询。




