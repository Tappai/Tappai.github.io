---
title: MySQL基础入门
date: 2019-10-24 19:46:30
tags: [原创,数据库]
categories: '工学'
---

本内容基于实验楼网站提供的[ MySQL 基础课程 ](https://www.shiyanlou.com/courses/9) ，共六个章节加一个挑战内容。

<!--more-->

##  SQL 介绍及 MySQL 安装

### 连接并进入数据库

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191020-1571582165687)

### 查看数据库信息

输入
>show databases;  //不要漏了分号“;”

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191020-1571582263095)



##  创建数据库并插入数据

### 进入数据库

>$ sudo service mysql start

>$ mysql -u root

### 创建数据库

> create database mysql_shiyan;

创建后查看数据库列表

>show databases;

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191021-1571645124537)

### 切换到新创建的数据库下

>use mysql_shiyan;

查看该数据库中的表信息

>show tables;

### 创建新的表

mysql_shiyan 中新建一张表 employee，包含姓名，ID 和电话信息，语句为：

>create table employee (id int(10),name char(20),phone int(12));

可以一次性输入一整句，也可以每小段分行回车输入。

创建两张新表后，输入：

>show tables;

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191021-1571645750873)

### 插入数据

向表employee中插入数据，输入：

> insert into employee(id,name,phone) values(01,'Tom',110110110);

如果继续添加对应id,name,phone的数据，则可省略该声明，例如：

> insert into employee values(02,'Jack',119119119);

但如果继续添加的对应数据有变，比如接下来只添加新的id和name，则要重新声明，输入：

> insert into employee(id,name) values(03,'Rose');

### 查看表中信息

> select *from employee;

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191021-1571646503968)

### 课后习题

新建一个名为 library 的数据库，包含 book、reader 两张表，根据自己的理解安排表的内容并插入数据。保存截图。

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191021-1571646745965)

> 这个表过于简陋，book内还可以分为,book_name和book_id



##  SQL 的约束

### 下载测试数据库文件到桌面

~~~
cd Desktop
~~~

~~~
git clone https://github.com/shiyanlou/SQL3.git
~~~


### 加载数据库文件

先按照之前的步骤登陆数据库，之后输入

~~~
source /home/shiyanlou/Desktop/SQL3/MySQL-03-01.sql;
~~~

用`show tables;`命令查看一下数据库列表
![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191023-1571843471962)

### 主键

主键 (PRIMARY KEY)是用于约束表中的一行，作为这一行的唯一标识符，在一张表中通过主键就能准确定位到一行。

> 应用情景：假如我们要存储一个学生的信息，信息包含姓名，身高，性别，年龄。
> 不幸的是有两个女孩都叫小梦，且她们的身高和年龄相同，数据库将无法区分这两个实体，这时就需要用到主键了。

### 默认值约束

默认值约束 (DEFAULT) 规定，当有 DEFAULT 约束的列，插入数据为空时，将使用默认值。

> 应用情景：默认值常用于一些可有可无的字段，比如用户的个性签名，如果用户没有设置，系统给他应该设定一个默认的文本，比如空文本或 ‘这个人太懒了，没有留下任何信息’。

查看本测试文件的代码
![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191023-1571844232914)
可以知道，department中people_num的默认值是10。因此分别输入
~~~
INSERT INTO department(dpt_name,people_num) VALUES('dpt1',11);
~~~
~~~
INSERT INTO department(dpt_name) VALUES('dpt2');
~~~

查看结果为
![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191023-1571844375411)

dpt2虽然没有输入数值，但被默认赋值10。

### 唯一约束

唯一约束 (UNIQUE) 比较简单，它规定一张表中指定的一列的值必须不能有重复值，即这一列每个值都是唯一的。

> 代码中对employee下的phone值进行了唯一约束限制。
> ![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191023-1571844583758)
> 可以看到当尝试插入具有相同phone值的两段数据时，第二段数据报错

### 外键约束

外键 (FOREIGN KEY) 既能确保数据完整性，也能表现表之间的关系。

> 应用情景：比如，现在有用户表和文章表，给文章表中添加一个指向用户 id 的外键，表示这篇文章所属的用户 id，外键将确保这个外键指向的记录是存在的，如果你尝试删除一个用户，而这个用户还有文章存在于数据库中，那么操作将无法完成并报错。因为你删除了该用户过后，他发布的文章都没有所属用户了，而这样的情况是不被允许的。同理，你在创建一篇文章的时候也不能为它指定一个不存在的用户 id。

本代码中将employee的键值in_dpt设定为需要参考department中的键值dpt_name，因此若输入
~~~
INSERT INTO employee VALUES(02,'Jack',30,3500,114114,'dpt3');
~~~
会由于department中没有对应的dpt3这个值，导致employee的该插入操作失败。
![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191023-1571845121539)

> 若再将dpt3改为dpt2，则可以操作成功

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191023-1571845212636)

### 非空约束

非空约束 (NOT NULL),听名字就能理解，被非空约束的列，在插入值时必须非空。

> 尝试输入两组数据，其中第二组数据由于没有输入salary键值，而salary又在代码中有非空约束，导致第二组数据插入报错

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191023-1571845453219)



##  挑战：搭建一个简易的成绩管理系统的数据库

该篇过程略长，还是直接去网站上自己尝试吧，[挑战1链接](https://www.shiyanlou.com/courses/9)

**稍微补充一下自己遇到的问题：**

### 更改列名

不小心打错了student表中的sid，输入成了id，网上查了下，输入

> alter table 表名 change column 旧列名 新列名  新列名格式；

也就是
~~~
alter table student change column id sid  int；
~~~
就改好了

### 输入多组数值

根据先前学的还以为每次都要先申明输入变量名

> (变量1，变量2) values(val1,val2)

结果发现可以直接 values(val1,val2) ，默认申明好了表里的几个变量。然后输入多组变量就直接
~~~
 values(val1,val2),values(val1,val2);
~~~
即可。

### 外键约束修改

偷懒着想跳过约束，结果检测失败了，没办法，只好查下怎么添加外键约束。
输入

> alter table 表名 add constraint 外键名 foreign key (变量名) references 某个表(变量名);

[参考文章](https://www.cnblogs.com/eyu1993/p/8902705.html)



## select语句详解

### select语句查询特定列

SELECT 语句的基本格式为：
~~~
SELECT 要查询的列名 FROM 表名字 WHERE 限制条件;
~~~

> 之前的实验中，我们用`*`表示所有的列

如果是查询特定的某列，比如要查看 employee 表的 name 和 age，就输入：
~~~
SELECT name,age FROM employee;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572186871263)

### select语句查询条件(一)数值范围查找

在上面查看特定列对象的基础上，如果还要对数据进行条件筛选，就要在语句末尾添加`where` 语句，比如要再筛选出age大于25的数据，则输入：

~~~
SELECT name,age FROM employee WHERE age>25;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572187461325)

`where`后的条件语句使用的数学符号可以是`=,<,>,>=,<=`

还可以使用`or`以及`and`来实现多条件筛选，例如输入：

~~~
#筛选出 age 大于 25，且 age 小于 30
SELECT name,age FROM employee WHERE age>25 AND age<30;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572187624786)

> 若要包含25和30两个数字，则可以改为age BETWEEN 25 AND 30

 ![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572187709072)

### select查询条件(二)包含项查找

`in`和`not in`用来筛选在或者不在某个范围内，例如我们查询employee表中，在dpt3或者dpt4的人的name,age,phone,in_dpt，则输入
~~~
SELECT name,age,phone,in_dpt FROM employee WHERE in_dpt IN ('dpt3','dpt4');
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572187991703)

### select查询条件(三)模糊查找

模糊查询可使用`like`来实现，和 LIKE 联用的通常还有通配符，代表未知字符。SQL中的通配符是 _ 和 % 。其中 `_` 代表一个未指定字符，`%` 代表不定个未指定字符

比如若是只记得电话号码前四位数为1101，而后两位忘记了，则可以用两个 _ 通配符代替：
~~~
SELECT name,age,phone FROM employee WHERE phone LIKE '1101__';
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572188245060)

同样的，模糊搜索name开头为J的数据，输入
~~~
SELECT name,age,phone FROM employee WHERE name LIKE 'J%';
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572188369454)

### select排序查询

使用`order by`可让查询结果升序或者降序排序。默认情况下是升序。

> 使用关键词 ASC 和 DESC 可指定升序或降序排序。

例如，按salary降序排序，输入：
~~~
SELECT name,age,salary,phone FROM employee ORDER BY salary DESC;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572188787708)

### 内置函数计算

| 函数名： | count | sum  | avg      | max    | min    |
| -------- | ----- | ---- | -------- | ------ | ------ |
| 作用:    | 计数  | 求和 | 求平均值 | 最大值 | 最小值 |

>其中 COUNT 函数可用于任何数据类型(因为它只是计数)，而 SUM 、AVG 函数都只能对数字类数据类型做计算，MAX 和 MIN 可用于数值、字符串或是日期时间数据类型。

比如计算出 salary 的最大、最小值，输入：
~~~
SELECT MAX(salary) AS max_salary,MIN(salary) FROM employee;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572189281405)

### 子查询

对于涉及多个表的查询需要运用子查询功能。

例如：想要知道名为 "Tom" 的员工所在部门做了几个工程。员工信息储存在 employee 表中，但工程信息储存在 project 表中。

输入：
~~~
SELECT of_dpt,COUNT(proj_name) AS count_project FROM project GROUP BY of_dpt
HAVING of_dpt IN
(SELECT in_dpt FROM employee WHERE name='Tom');
~~~

> 上面代码包含两个 SELECT 语句，第二个 SELECT 语句将返回一个集合的数据形式，然后被第一个 SELECT 语句用 in 进行判断。

HAVING 关键字可以的作用和 WHERE 是一样的，都是说明接下来要进行条件筛选操作。

区别在于 HAVING 用于对分组后的数据进行筛选

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572189447462)

> 子查询还可以扩展到 3 层、4 层或更多层。

### 连接查询

子查询虽然可以处理多个表，但返回的只是一个表中的数据，若要同时返回多个表的数据，则必须使用`join`操作。

例如，查询各员工所在部门的人数，其中员工的 id 和 name 来自 employee 表，people_num 来自 department 表，输入：
~~~
SELECT id,name,people_num
FROM employee,department
WHERE employee.in_dpt = department.dpt_name
ORDER BY id;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572189612143)

> 也可以使用`join on` 语法，之前的语句可改为：

~~~
SELECT id,name,people_num
FROM employee JOIN department
ON employee.in_dpt = department.dpt_name
ORDER BY id;
~~~

结果不变。

### 课后习题

> 使用连接查询的方式，查询出各员工所在部门的人数与工程数，工程数命名为 count_project。（连接 3 个表，并使用 COUNT 内置函数）

~~~
> select name,people_num, count(proj_name) as count_project from 
> employee,department,project 
> where in_dpt = dpt_name and of_dpt = dpt_name
> group by name, people_num;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191027-1572191387699)

> 一开始没加group by 语句，一直报错





##  数据库及表的修改和删除

### 删除数据库

使用 `drop database 数据库名`删除数据库

例如，输入：
~~~
DROP DATABASE test_01;
~~~

删除数据库test_01

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191028-1572272169192)

### 修改数据库

由于修改数据库名会带来安全问题，因此现在的mysql版本已经去除了修改命令。

> 若是需要更改数据库名，最好是新建一个数据库，然后把数据复制过去，同时最好保留旧数据库作为备份。

### 修改库中的表

#### 修改表名

有三种语句格式，效果一样

~~~
RENAME TABLE 原名 TO 新名字;

ALTER TABLE 原名 RENAME 新名;

ALTER TABLE 原名 RENAME TO 新名;
~~~

例如，输入:
~~~
RENAME TABLE table_1 TO table_2;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191028-1572272502479)

#### 删除表

~~~
DROP TABLE 表名字;
~~~

例如，输入:
~~~
DROP TABLE table_2;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191028-1572272591178)

### 修改表结构

若要在表中新增一列，使用

~~~
ALTER TABLE 表名字 ADD COLUMN 列名字 数据类型 约束;
或：
ALTER TABLE 表名字 ADD 列名字 数据类型 约束;
~~~

例如，在employee表中加入一个height列，输入:
~~~
ALTER TABLE employee ADD height INT(4) DEFAULT 170;
~~~

> 提醒：语句中的 INT(4) 不是表示整数的字节数，而是表示该值的显示宽度，如果设置填充字符为 0，则 170 显示为 0170

![](https://doc.shiyanlou.com/MySQL/sql-05-05.png/wm)

> 加入的列默认是放在最右边的，如果想指定插入位置，就再加上 `after 某列` 

![](https://doc.shiyanlou.com/MySQL/sql-05-06.png/wm)

> 如果想放在第一列就改为`first`

![](https://doc.shiyanlou.com/MySQL/sql-05-07.png/wm)

删除一列使用
~~~
ALTER TABLE 表名字 DROP COLUMN 列名字;

或： ALTER TABLE 表名字 DROP 列名字;
~~~

重命名一列
~~~
ALTER TABLE 表名字 CHANGE 原列名 新列名 数据类型 约束;
~~~

> 注意：这条重命名语句后面的 “数据类型” 不能省略，否则重命名失败。

更改数据类型

~~~
ALTER TABLE 表名字 MODIFY 列名字 新数据类型;
~~~

> 修改数据类型必须小心，因为这可能会导致数据丢失。在尝试修改数据类型之前，请慎重考虑。

### 表内容修改

修改某个值
~~~
UPDATE 表名字 SET 列1=值1,列2=值2 WHERE 条件;
~~~

![](https://doc.shiyanlou.com/MySQL/sql-05-10.png/wm)

> 一定要有`where`条件约束，否则会把整个表全都改了

删除一行记录

~~~
DELETE FROM 表名字 WHERE 条件;
~~~

![](https://doc.shiyanlou.com/MySQL/sql-05-11.png/wm)

> 删除操作也必须有`where`条件约束，否则会删除所有内容





##  其它基本操作

### 索引

索引是一种与表有关的结构，它的作用相当于书的目录，可以根据目录中的页码快速找到所需的内容。

对一张表中的某个列建立索引，有以下两种语句格式：
~~~
ALTER TABLE 表名字 ADD INDEX 索引名 (列名);

CREATE INDEX 索引名 ON 表名字 (列名);
~~~

例如输入
~~~
ALTER TABLE employee ADD INDEX idx_id (id);  #在employee表的id列上建立名为idx_id的索引

CREATE INDEX idx_name ON employee (name);   #在employee表的name列上建立名为idx_name的索引
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572616720326)

再输入
~~~
show index from employee;
~~~
![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572618288640)

可以看到刚才新建的索引

### 视图

视图是从一个或多个表中导出来的表，是一种虚拟存在的表。用户可以不用看到整个数据库中的数据，而只关心对自己有用的数据。

创建视图的语句格式为：
~~~
CREATE VIEW 视图名(列a,列b,列c) AS SELECT 列1,列2,列3 FROM 表名字;
~~~

> 可见创建视图的语句，后半句是一个 SELECT 查询语句，所以视图也可以建立在多张表上，只需在 SELECT 语句中使用子查询或连接查询，这些在之前的实验已经进行过。

现在创建一个简单的视图，输入
~~~
CREATE VIEW v_emp (v_name,v_age,v_phone) AS SELECT name,age,phone FROM employee;
~~~
查看，输入
~~~
select * from v_emp;
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572618902708)

### 导入

导入有两种形式：

- 数据导入：数据存储在类似xx.txt的文档中，导入规则由数据库完成。
- SQL文件导入：相当于执行该文件中包含的 SQL 语句，可以实现多种操作，包括删除，更新，新增，甚至对数据库的重建。

由于导入导出大量数据都属于敏感操作，根据 mysql 的安全策略，导入导出的文件都必须在指定的路径下进行，在 mysql 终端中查看路径变量，输入：

~~~
show variables like '%secure%';
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572619653336)

> 注意到 secure_file_priv 变量指定安全路径为 /var/lib/mysql-files/ ，要导入数据文件，需要将该文件移动到安全路径下。

这里，我们将测试文件复制过去
~~~
sudo cp -a /home/shiyanlou/Desktop/SQL6 /var/lib/mysql-files/ #复制
sudo vim /var/lib/mysql-files/SQL6/in.txt #查看内容
~~~

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572620139432)

数据文件导入语句格式为：
~~~
LOAD DATA INFILE '文件路径和文件名' INTO TABLE 表名字;
~~~

于是，输入：
~~~
load data infile '/var/lib/mysql-files/SQL6/in.txt' into table employee;
~~~

用`select *from employee;`查看一下

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572621222674)

### 导出

导出语句基本格式为：
~~~
SELECT 列1，列2 INTO OUTFILE '文件路径和文件名' FROM 表名字;
~~~

> 注意：语句中 “文件路径” 之下不能已经有同名文件。

将测试数据导出，输入：
~~~
SELECT * INTO OUTFILE '/var/lib/mysql-files/out.txt' FROM employee;
~~~

可输入`sudo gedit /var/lib/mysql-files/out.txt`查看内容

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572621476756)

### 备份

备份与导出的区别：导出的文件只是保存数据库中的数据；而备份，则是把数据库的结构，包括数据、约束、索引、视图等全部另存为一个文件。

我们将使用mysqldump来完成备份。

> mysqldump 是 MySQL 用于备份数据库的实用程序。它主要产生一个 SQL 脚本文件，其中包含从头重新创建数据库所必需的命令 CREATE TABLE INSERT 等。

使用 mysqldump 备份的语句：
~~~
mysqldump -u root 数据库名>备份文件名;   #备份整个数据库

mysqldump -u root 数据库名 表名字>备份文件名;  #备份整个表
~~~

先退出mysql控制台，在终端输入：
~~~
cd /home/shiyanlou/
mysqldump -u root mysql_shiyan > bak.sql;
~~~

输入`sudo gedit /home/shiyanlou/bak.sql`来查看内容

![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572621805749)

### 恢复

使用备份文件可以恢复数据库，或者用作导入。恢复数据库有两种方式：
- 第一种是之前经常使用的，类似`source /tmp/SQL6/MySQL-06.sql`这样的语句
- 第二种方式：

~~~
mysql -u root          #因为在上一步已经退出了MySQL，现在需要重新登录

CREATE DATABASE test;  #新建一个名为test的数据库
~~~
再次 Ctrl+D 退出 MySQL，然后输入语句进行恢复，把刚才备份的 bak.sql 恢复到 test 数据库：

~~~
mysql -u root test < bak.sql
~~~

查看一下test数据库的表
![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572622190476)
已经把数据从备份文件中导入到test里了

### 课后习题
> 建立员工名字 employee.name 和对应部门人数 department.people_num 的视图并展示。

~~~mysql
create view v_emp2 (employee_name,dptartment_people_num) as select name,people_num from employee,department WHERE in_dpt = dpt_name;
~~~

> 自己写出来时发现没加where限制，上面是已经补上了的

更正之后的结果：
![图片描述](https://dn-simplecloud.shiyanlou.com/courses/uid1134095-20191101-1572622827849)



**到此，门估计都还没入......**

(完)