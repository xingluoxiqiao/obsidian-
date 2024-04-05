## DDL：数据定义语言
创建修改或删除对象
CREATE DATABASE - 创建新数据库 
DROP DATABASE – 删除数据库
ALTER DATABASE - 修改数据库属性 
CREATE TABLE - 创建新表   
ALTER TABLE – 修改数据库表结构
DROP TABLE - 删除表
CREATE INDEX - 创建索引   
DROP INDEX - 删除索引 
## DML：数据操纵语言
增添修改删除数据
INSERT - 向数据库表中插入数据 
UPDATE - 更新数据库表中的数据
DELETE - 从数据库表中删除数据 
## DQL：数据查询语言
用于对数据库进行数据查询的语句
## DCL：数据控制语言
对数据库对象访问权进行控制
GRANT – 授予用户对数据库对象的权限
DENY – 拒绝授予用户对数据库对象的权限
REVOKE – 撤消用户对数据库对象的权限
## TPL：事务处理语言
用于数据库内部事务处理
BEGIN TRANSACTION – 开始事务
COMMIT – 提交事务
ROLLBACK – 回滚事务
## CCL：游标控制语言
用于数据库游标操作的语句
DECLARE CURSOR – 定义游标
FETCH INTO – 提交游标数据
CLOSE CURSOR– 关闭游标
## SQL语言基本数据类型
字符：CHAR、VARCHAR、TEXT
整数：SMALLINT、INTEGER
浮点数：NUMBER(n,d)、FLOAT(n,d)
日期：DATE、DATETIME
货币：MONEY

## 表
CREATE TABLE  <表名>
 （ <列名1>  <数据类型>  [列完整性约束]，
	<列名2>  <数据类型>   [列完整性约束]，	
    <列名3>  <数据类型>  [列完整性约束]，
    …
     CONSTRAINT  <约束名>  PRIMARY Key(主键列)(复合主键)
   )；
其中列完整性约束关键词：
PRIMARY KEY——主键
NOT NULL——非空值（必填）
NULL——空值
UNIQUE——值唯一
CHECK——有效性检查（eg:CHECK(CourseType IN('基础课','专业课','选修课')),）
DEFAULT——缺省值  (eg:DEFAULT  '闭卷考试')
eg:
CREATE  TABLE  Plan
( CourseID  	char(4)  	NOT  NULL,
  TeacherID  	char(4)  	NOT  NULL,
  CourseRoom  	varchar(30)    UNIQUE， 
  CourseTime  	varchar(30)    CHECK(CourseTime  IN ('...','...')),
  Note  		varchar(50)      DEFAULT    '...',
  CONSTRAINT	CoursePlan_PK	PRIMARY Key(CourseID,TeacherID)
);

### ALTER TABLE<表名><修改方式>
1.用于增加新列或列完整性约束
**ALTER TABLE <表名> ADD <新列名称><数据类型>|[完整性约束]**
2.用于删除指定列或列的完整性约束条件
**ALTER TABLE<表名> DROP  CONSTRAINT<完整性约束名>；**
**ALTER TABLE<表名> DROP  COLUMN <列名>；**
3.用于修改表名称、列名称
**ALTER TABLE <表名> RENAME TO <新表名>；**
**ALTER TABLE <表名> RENAME <原列名> TO <新列名>；**
4.用于修改列的数据类型
**ALTER TABLE <表名> ALTER  COLUMN <列名> TYPE<新的数据类型>；**
### DROP TABLE <表名>;
删除该表的所有数据及其结构

## 索引
索引（Index）是一种按照关系表中指定列的取值顺序组织元组数据存储的数据结构，使用它可以加快表中数据的查询访问。
![[Pasted image 20230515111107.png]]
### CREATE INDEX  <索引名>  ON <表名><（列名）>
eg:CREATE  INDEX  Birthday_Idx  ON  STUDENT （Birthday）；
### ALTER  INDEX  <索引名> <修改项>；
eg:ALTER  INDEX Birthday_Idx  RENAME TO Bday_Idx；
### DROP  INDEX  <索引名> ；
DROP  INDEX bday_idx；


### INSERT  INTO  <表名|视图名>[<列名表>]  VALUES （列值表）;
eg:INSERT INTO Student VALUES('2017220101105','柳因','女','1999-04-23','软件工程', 'liuyin@163.com');

### 数据更新
**UPDATE  <表名|视图名>
SET  <列名1>=<表达式1> \[，<列名2>=<表达式2>...]
[WHERE   <条件表达式>]；**
eg：
UPDATE  Student
SET  Email='zhaodong@163.com'
WHERE   StudentName='赵东';
### 数据删除
**DELETE 
FROM  <表名|视图名>
[WHERE   <条件表达式>]；**
eg：
DELETE 
FROM  STUDENT
WHERE   StudentName='张亮';
### 数据查询
**SELECT  [ALL|DISTINCT]  <目标列>[，<目标列>…]
[ INTO <新表> ]
FROM  <表名|视图名>[，<表名|视图名>…]
[ WHERE  <条件表达式> ]
[ GROUP BY  <列名> [HAVING <条件表达式> ]]
[ ORDER BY  <列名> [ ASC | DESC ] ];**
**DISTINCT 过滤重复数据**
**WHERE子句中可以使用如下方式，指定范围数据。**
1.使用BETWEEN..AND关键词来限定列值范围，还可以使用关键词LIKE与通配符来限定查询条件。
2.使用通配符来限定字符串数据范围。下划线_通配符用于代表一个未指定的字符。百分号%通配符用于代表一个或多个未指定的字符。
3.使用多个条件表达式，并通过逻辑运算符（AND、OR、NOT）连接操作，以及使用IN或NOT IN关键词，进一步限定结果集的数据范围。
**ORDER BY  DESC/ASC      降序、升序**
如果结果集需要按多个列排序，可以分别加入关键字ASC或DESC改变。
### 聚合函数
![[Pasted image 20230515112614.png]]
**GROUP BY**     **分组统计**
**HAVING 限定条件**
eg：
SELECT  Major  AS 专业,  COUNT（StudentID） AS 学生人数
FROM  Student
WHERE  StudentGender=’男’
GROUP  BY  Major
HAVING  COUNT(* )>2;

## 多表关联查询
### 子查询与多表关联
**SELECT    <目标列>[，<目标列>…]
FROM  <表名>
WHERE  <条件中嵌套另一关系表的SELECT 查询结果集>**
eg：
SELECT  TeacherID,  TeacherName,  TeacherTitle
FROM  Teacher
WHERE  CollegeID  IN
	    (SELECT  CollegeID  
	     FROM  College
	     WHERE  CollegeName=’计算机学院’);
### 连接关联多表查询
**SELECT    <目标列>[，<目标列>…]
FROM  <表名1>，<表名2>，…, <表名n>，
WHERE  <关系表之间的连接关联条件>**
eg：
SELECT  B.CollegeName AS 学院名称,  A.TeacherID  AS 编号, A.TeacherName  AS 姓名,  A.TeacherGender  AS 性别,  A. TeacherTitle  AS 职称
FROM  Teacher  AS  A，College  AS  B
WHERE  A.CollegeID=B.CollegeID 
ORDER  BY  B.CollegeName, A.TeacherID;
### JOIN …ON连接查询语句
**SELECT  <目标列>[，<目标列>…]
FROM  <表名1>  JOIN  <表名2>  ON <连接条件>;**
eg：
SELECT  B.CollegeName AS 学院名称,  A.TeacherID  AS 编号, A.TeacherName  AS 姓名,  A.TeacherGender  AS 性别,  A. TeacherTitle  AS 职称
FROM  TEACHER  AS  A  JOIN  COLLEGE  AS  B
ON  A.CollegeID=B.CollegeID 
ORDER  BY  B.CollegeName, A.TeacherID;

（前面介绍的多表连接方式在SELECT查询语句称为内部连接。 在一些特殊情况下，如关联表中一些行的主键与外键不匹配，查询结果集就会丢失部分数据。
LEFT JOIN: 左外连接，即使没有与右表关联列值匹配，也从左表返回所有的行。
RIGHT JOIN: 右外连接，即使没有与左表关联列值匹配，也从右表返回所有的行。
FULL JOIN: 全外连接，同时进行左连接和右连接，就返回所有行。）

## 数据控制
#### GRANT  <权限列表>  ON  <数据库对象>  TO  <用户或角色> \[ WITH GRANT OPTION ]；
由数据库对象创建者或管理员执行的权限授予语句，它可以把访问数据库对象权限授予给其他用户或角色。
eg：GRANT  SELECT, INSERT, UPDATE, DELETE  ON  REGISTER  TO  RoleS;
#### REVOKE  <权限列表>  ON  <数据库对象>  FROM  <用户或角色> ;
由数据库对象创建者或管理员将赋予其它用户或角色的权限进行收回语句，它可以收回原授予给其他用户或角色的权限。
eg：REVOKE  DELETE  ON  REGISTER  FROM  RoleS;
#### DENY  <权限列表>  ON  <数据库对象>  TO  <用户或角色> ;
拒绝给当前数据库内的用户或者角色授予权限，并防止用户或角色通过其组或角色成员继承权限。
eg：DENY  DELETE  ON  TEACHER  TO  RoleT;


## 视图
视图是一种通过基础表或其它视图构建的虚拟表。它本身没有自己的数据，而是使用了存储在基础表中的数据。
#### CREATE  VIEW  <视图名>[(列名1)，(列名2)，…]  AS  <SELECT查询>；视图创建
eg：
CREATE  VIEW  BasicCourseView  AS
SELECT  CourseName,  CourseCredit,  CoursePeriod,  TestMethod
FROM    COURSE
WHERE  CourseType=‘基础课’;
当视图在数据库中创建后，用户可以像访问关系表一样去操作访问视图。
#### DROP  VIEW  <视图名>；视图删除
### 作用：
1.使用视图简化复杂SQL查询操作
2.使用视图提高数据访问安全性
3.提供一定程度的数据逻辑独立性
4.集中展示用户所感兴趣的特定数据