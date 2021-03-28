---
layout: post
title: "Basics of database operation"
date: "2021-03-28 10：42：00"
tag: MySQL SQL Database
---
- [MySQL 数据库介绍](#org2544109)
  - [分类](#orgd06fed8)
  - [MySQL 安装](#org10840b6)
  - [MySQL 使用](#org9bce169)
    - [基本操作](#org5d60d4b)
    - [创建一个数据表](#org6ba45fb)
    - [增删改查](#org6689744)
    - [常用数据类型](#org1ac21f7)
- [mysql 约束](#org629918b)
  - [主键约束 / 联合主键](#org0f42bd5)
  - [唯一约束](#org7eb4386)
  - [非空约束](#org16d01e9)
  - [默认约束](#orgbd4e34f)
  - [外键约束](#org7319863)
- [数据库的设计范式](#orgbc2f1ad)
  - [第一范式,1NF](#orgf09110b)
  - [第二范式,2NF](#orgaea4fd7)
  - [第三范式,3NF](#orgf1b6629)
- [mysql 查询练习](#org6bd86e3)
  - [先建立所需的表](#org8b86e69)
  - [建表代码](#org8a1904e)
  - [向表中添加记录](#orgf31581a)
  - [查询表中所有记录](#org7323436)
- [Sql 的四种链接查询](#org26e5cdf)
  - [先建立所需的表](#org6d551ee)
  - [内链接 inner join 或者 join](#org324f9aa)
  - [(外链接)左链接 left join 或者 left outer join](#org9cb1db5)
  - [(外链接)右链接 right join 或者 right outer join](#orge64cb1e)
  - [完全外链接 full join 或者 full outer join](#orgf3837f0)
- [MySQL 事务](#orgdae1aa0)
  - [事务的四大特征，ACID。](#org70420e2)
  - [隔离性:](#org3d7714d)
  - [mysql 如何开启事务？](#org8ca8f0e)
  - [1. read-uncommitted; 读未提交的](#org8b0aa23)
  - [2. read committed; 读已提交的](#org8379851)
  - [3, repeatable read; 可以重复读的](#org77f2939)
  - [4. serializable; 串行化](#org4a8ed12)
  - [隔离级别](#org8bbf9b6)
- [注](#orgc383441)


<a id="org2544109"></a>

# MySQL 数据库介绍


<a id="orgd06fed8"></a>

## 分类

-   关系型数据库
-   非关系型数据库


<a id="org10840b6"></a>

## MySQL 安装

-   系统：ubuntu

```sh
sudo apt-get install mysql-server 
sudo mysql_secure_installation    # 推荐进行安全配置
systemctl enable mysql            # 设置开机启动服务
mysql -u [用户名] -p [密码]        # 登入MySQL服务器
```


<a id="org9bce169"></a>

## MySQL 使用


<a id="org5d60d4b"></a>

### 基本操作

```sql
show databases;        -- 查询数据库。
select * from [table]; -- 查询一个表里的内容
use test;              -- 选中一个数据库进行操作。  
exit;                  -- 退出
create database test;  -- 创建一个数据库 
show tables;           -- 查看该数据库中的所有表
```


<a id="org6ba45fb"></a>

### 创建一个数据表

```sql
create table pet(
name varchar(20),
owner varchar(20),
species varchar(20),
sex char(1),
birth date,
death date
);
```


<a id="org6689744"></a>

### 增删改查

```sql
describe pet;         -- 查看创建好的数据表的结构
desc pet;
select * from pet;    -- 查看表中的所有记录

-- 向表中添加记录
insert into pet values('puffball','Diane','hamster','f','1999-12-9',null);

-- 删除表中的数据
delete from pet where name='puffball';

-- 修改表中的数据
insert into pet values('puffball','Diane','hamster','f','1999-12-9',null);
update pet set name='大波浪' where owner='Diane';

-- 再来查看所有数据
select * from pet;

drop database <数据库名>;
drop table teacher;
```


<a id="org1ac21f7"></a>

### 常用数据类型

-   日期的选择按照格式，数值和字符串选择按照大小。


<a id="org629918b"></a>

# mysql 约束


<a id="org0f42bd5"></a>

## 主键约束 / 联合主键

-   它能够唯一确定一张表中的一条记录，通过添加主键约束，可以使该字段不重复且不为空。

```sql
create table user(
id int primary key,
name varchar(20)
);

insert into user values(1,'张三');
insert into user values(1,'张三');  -- error
insert into user values(2,'张三');
```

-   联合主键

```sql
create table user2(
id int,
name varchar(20),
password varchar(20),
primary key(id,name)
);

insert into user2 values(1,'张三','123');
insert into user2 values(2,'张三','123');
```

-   自增约束

```sql
create table user3(
id int primary key auto_increment,
name varchar(20)
);

insert into user3(name) values('张三');
insert into user3(name) values('张三');
select * from user3;
```

-   创建表时，忘记创建主键约束。

```sql
create table user4(
id int,
name varchar(20)
);

desc user4;
alter table user4 add primary key(id);
desc user4;
```

-   不想要主键后，如何删除主键

```sql
alter table user4 drop primary key;
desc user4;
```

-   修改主键约束

```sql
alter table user4 modify id int primary key;
desc user4;
```


<a id="org7eb4386"></a>

## 唯一约束

-   约束修饰的字段的值不可重复,可为空。

```sql
create table user5(
id int,
name varchar(20)
);

alter table user5 add unique(name);
desc user5;
```

-   也可以这样

```sql
create table user6(
id int,
name varchar(20),
unique(name)
);

create table user7(
id int,
name varchar(20) unique
);
```

-   两个键在一起不重复就行。

```sql
create table user8(
id int,
name varchar(20),
unique(id,name)
);
```

-   删除唯一约束

```sql
alter table user7 drop index name;
desc user7;
```

-   modify 添加

```sql
alter table user7 modify name varchar(20) unique;
```


<a id="org16d01e9"></a>

## 非空约束

```sql
create table user9(
id int,
name varchar(20) not null
);

desc user9;
```


<a id="orgbd4e34f"></a>

## 默认约束

-   未传值时，就会使用默认值。

```sql
create table user10(
id int,
name varchar(20),
age int default 10
);

desc user10;

insert into user10(id,name) values(3,'name');
desc user10;
select * from user10;
```


<a id="org7319863"></a>

## 外键约束

-   涉及到两个表，父表，子表（主表，父表）;

```sql
create table classes(
id int primary key, 
name varchar(20)
);

create table students(
id int primary key,
name varchar(20),
class_id int,
foreign key(class_id) references classes(id)
);

desc classes;
desc students;

insert into classes values(1,'一班');
insert into classes values(2,'二班');
insert into classes values(3,'三班');
insert into classes values(4,'四班');

select * from classes;

insert into students values(101,'张三',1);
insert into students values(102,'张三',2);
insert into students values(103,'张三',3);
insert into students values(104,'张三',4);

insert into students values(105,'张三',5);
select * form classes;
```

-   主表中没有的数据，在父表中是不可以添加的。
-   主表中的记录被子表中引用，是不可以被删除的。

```sql
delete from classes where id=4;
```


<a id="orgbc2f1ad"></a>

# 数据库的设计范式


<a id="orgf09110b"></a>

## 第一范式,1NF

-   数据表中的所有字段都是不可分割的原子值

```sql
create table student2(
id int primary key,
name varchar(20),
address varchar(30)
);

insert into student2 values(1,'张四','北京昌平区1001号');
insert into student2 values(2,'李五','北京昌平区1002号');
insert into student2 values(3,'王六','北京昌平区1003号');

select * from student2;
```

-   字段或值还可以继续拆分的，就不满足第一范式。

```sql
create table student3(
id int primary key,
name varchar(20),
country varchar(30),
privence varchar(30),
city varchar(30),
detail varchar(30)
);

insert into student3 values(1,'张四','中国','北京','北京','昌平区1001号');
insert into student3 values(2,'李五','中国','北京','北京','昌平区1002号');
insert into student3 values(3,'王六','中国','北京','北京','昌平区1003号');

select * from student3;
```

-   范式，设计的越详细，对于某些实际的操作可能更好，但不一定都有好处。


<a id="orgaea4fd7"></a>

## 第二范式,2NF

-   前提，必须是满足第一范式，第二范式要求，除主键之外的每一列都必须完全依赖与主键。
-   如果要出现不完全依赖，只可能发生在联合主键的情况下。

```sql
-- 订单表
create table myorder{
product_id int,
customer_id int,
product_name varchar(20),
customer_name varchar(20),
primary key(product_id,customer_id)
);
```

-   问题？ 除主键之外的其他列，只依赖与主键的部分字段。
-   解决办法：拆表

```sql
create table myorder(
order_id int primary key,
product_id int,
customer_id int
);

create table product(
id int primary key,
name varchar(20)
);

create table customer(
id int primary key,
name varchar(20)
);
```

-   分成三个表，就满足第二范式。


<a id="orgf1b6629"></a>

## 第三范式,3NF

-   必须满足第二范式，除去主键外，其他列之间不能有传递依赖关系。

```sql
create table myorder(
order_id int primary key,
product_id int,
customer_id int
);

create table customer(
id int primary key,
name varchar(20),
phone varchar(15)
);
```


<a id="org6bd86e3"></a>

# mysql 查询练习


<a id="org8b86e69"></a>

## 先建立所需的表

**学号和课程号是联合主键，不用的话，每个人只能有一门成绩**

-   student 学号，姓名，性別，出生年月，所在班级
-   Course 课程号，课程名称，教师编号
-   Score 学号，课程号，成绩
-   Teacher 教师编号,教师名字,教师性别,出生年月,职称,所在部门


<a id="org8a1904e"></a>

## 建表代码

```sql
create database selectTest;
show databases;
use selectTest;
create table student(
sno varchar(20) primary key,
sname varchar(20) not null,
ssex varchar(10) not null,
sbirthday datetime,
class varchar(20) comment '学生所在的班级'
);

create table teacher( 
tno varchar(20) primary key,
tname varchar(20) not null,
tsex varchar(10) not null,
tbirthday varchar(20),
prof varchar(20) not null,
depart varchar(20) not null
);

create table course(
cno varchar(20) primary key,
cname varchar(20) not null,
tno varchar(20) not null,
foreign key(tno) references teacher(tno)
);


create table score(
sno varchar(20) not null,
cno varchar(20) not null,
degree decimal,
foreign key(sno) references student(sno),
foreign key(cno) references course(cno),
primary key(sno,cno)
);

show tables;
select * from teacher;
select * from student;
select * from score;
select * from course;
```


<a id="orgf31581a"></a>

## 向表中添加记录

```sql
--学生表数据
INSERT INTO student VALUES('101','曾华','男','1977-09-01','95033');
INSERT INTO student VALUES('102','匡明','男','1975-10-02','95031');
INSERT INTO student VALUES('103','王丽','女','1976-01-23','95033');
INSERT INTO student VALUES('104','李军','男','1976-02-20','95033');
INSERT INTO student VALUES('105','王芳','女','1975-02-10','95031');
INSERT INTO student VALUES('106','陆军','男','1974-06-03','95031');
INSERT INTO student VALUES('107','王尼玛','男','1976-02-20','95033');
INSERT INTO student VALUES('108','张全蛋','男','1975-02-10','95031');
INSERT INTO student VALUES('109','赵铁柱','男','1974-06-03','95031');

--教师表数据
INSERT INTO teacher VALUES('804','李诚','男','1958-12-02','副教授','计算机系');
INSERT INTO teacher VALUES('856','张旭','男','1969-03-12','讲师','电子工程系');
INSERT INTO teacher VALUES('825','王萍','女','1972-05-05','助教','计算机系');
INSERT INTO teacher VALUES('831','刘冰','女','1977-08-14','助教','电子工程系');

--添加课程表
INSERT INTO course VALUES('3-105','计算机导论','825');
INSERT INTO course VALUES('3-245','操作系统','804');
INSERT INTO course VALUES('6-166','数字电路','856');
INSERT INTO course VALUES('9-888','高等数学','831');

--添加成绩表
INSERT INTO score VALUES('103','3-245','86');
INSERT INTO score VALUES('103','3-105','92');
INSERT INTO score VALUES('103','6-166','85');

INSERT INTO score VALUES('105','3-245','75');
INSERT INTO score VALUES('105','3-105','88');
INSERT INTO score VALUES('105','6-166','79');

INSERT INTO score VALUES('109','3-245','68');
INSERT INTO score VALUES('109','3-105','76');
INSERT INTO score VALUES('109','6-166','81');

show tables;
select * from teacher;
select * from student;
select * from score;
select * from course;
```


<a id="org7323436"></a>

## 查询表中所有记录

-   查询student 表中所有的sname, ssex, class 列。

```sql
select sname, ssex, class from student;
```

-   查询教师所有的单位即不重复的depart 列。
-   distinct 排除重复

```sql
select distinct depart from teacher;
```

-   查询score 表中的成绩在60-90之间的所有记录。

```sql
select * from score where degree between 60 and 90;
select * from score where degree > 60 and degree < 90;
```

-   查询 score 表中成绩为85, 86, 或88 的记录。

```sql
select * from score where degree in(85,86,88); 
```

-   查询 student 表中“95031” 班或性別为女的同学记录。

```sql
select * from student where class='95031' or ssex='女';
```

-   以 class 降序查询 student 表的所有记录。

```sql
select * from student order by class desc;
select * from student order by class asc;  -- default
```

-   以cno升序，degree 降序查询 score 表中的所有记录。

```sql
select * from score order by cno asc, degree desc;
```

-   查询‘95031’ 班的学生人数。

```sql
select count(*) from student where class='95031';
```

-   查询 score 表中的最高分的学生学号和课程号。子查询或者排序。

```sql
select sno,cno from score where degree=(select max(degree) from score);
-- 1, 找到最高分，
select max(degree) from score;
-- 2, 找到v最高分的 sno 和 cno;
select sno,cno from score where degree=(select max(degree) from score);

-- 或者：排序的做法，limit 第一个数表示从多少开始，第二个数表示查多少条。
select sno,cno from score order by degree;
select sno,cno,degree from score order by degree desc limit 0,1;
```

-   查询每门课的平均成绩
-   avg()

```sql
select * from course;
select avg(degree) from score where cno='3-105';
select degree from score where cno='3-105';

-- 写多个后->
select avg(degree) from score where cno='3-105';
select avg(degree) from score where cno='3-245';
select avg(degree) from score where cno='6-166';
select avg(degree) from score where cno='9-888';     

-- 如果在一个sql 语句中写？ group by 分组。
select cno,avg(degree) from score group by cno;
```

-   查询 score 表中至少有 2 名学生选修的并以3开头的课程。

```sql
select cno from score group by cno having count(cno)>=2 and cno like '3%';
select cno,avg(degree) from score group by cno having count(cno)>=2 and like '3%';      -- error
select cno,avg(degree),count(*) from score group by cno having count(cno)>=2 and cno like '3%';
```

-   查询分数大于70,小于90 的 sno列。

```sql
select sno,degree from score where degree between 60 and 90;
```

-   查询所有学生的 sname, cno 和 degree 列。
-   多表查寻

```sql
select sno,sname from student;
select sno,cno,degree from score;
select sname,cno,degree from student,score where student.sno=score.sno;
```

-   查询所有学生的 sno, cname 和 degree 列。

```sql
select sno,cname,degree from course,score where course.cno = score.cno;
```

-   查寻所有学生的 sname, cname 和 degree 列。

```sql
sname -> student
cname -> course
degree -> score
select sname,cname,degree from student,course,score where student.sno = score.sno and course.cno = score.cno;
select sname,cname,degree,student.sno,course.cno from student,course,score where student.sno = score.sno and course.cno = score.cno;
select sname,cname,degree,student.sno as stu_sno,course.cno as cou_cno from student,course,score where student.sno = score.sno and course.cno = score.cno;
select sname,cname,degree,student.sno as stu_sno,score.sno,course.cno as cou_cno,score.cno from student,course,score where student.sno = score.sno and course.cno = score.cno;
```

-   查询 '95031' 班学生每们课的平均分。

```sql
select * from student where class='95031';
select sno from student where class='95031';
select * from score where sno in (select sno from student where class='95031');
select cno,avg(degree) from score where sno in (select sno from student where class='95031') group by cno;
```

-   查询选修 '3-105' 课程的成绩高于 109 号同学‘3-105'成绩的所有同学的记录。

```sql
select degree from score where sno='109' and cno='3-105';
select * from score where degree > ();
select * from score where cno='3-105' and  degree > (select degree from score where sno='109' and cno='3-105');
```

-   查询成绩高于学号为 109 课程号为 '3-105' 的成绩的所有记录。

```sql
select degree from score where sno='109' and cno='3-105';
select * from score where degree > (select degree from score where sno='109' and cno='3-105');
```

-   查询和学号为108，101 的同学同年出生的所有学生的 sno, sname 和 sbirthday列。

```sql
select * from student where sno in (101,108);
select year(sbirthday) from student where sno in (101,108);
select * from student where year(sbirthday)=(select year(sbirthday) from student where sno in (101,108));      -- error more than 1 row.
select * from student where year(sbirthday) in (select year(sbirthday) from student where sno in (101,108));

select * from student where sno in (101,108);
select year(sbirthday) from student where sno in (101,108);
select * from student where year(sbirthday) in (select year(sbirthday) from student where sno in (101,108));
```

-   查询’张旭'教师任课的学生成绩。

```sql
select * from teacher;
select * from teacher where tname='张旭';
select tno from teacher where tname='张旭';
select cno from course where tno=(select tno from teacher where tname='张旭');
select * from score where cno=(select cno from course where tno=(select tno from teacher where tname='张旭'));
```

-   查询选修某个课程的同学人数多于5人的教师姓名。

```sql
select cno from score group by cno;
select count(cno) from score group by cno;
select count(sno) from score group by cno;

-- 最多为3，添加点数据。
insert into score values('101','3-105','90');
insert into score values('102','3-105','69');
insert into score values('104','3-105','70');
select * from score;

select count(sno) from score group by cno;
select cno from score group by cno having count(*) > 5;
select tno from course where cno=(select cno from score group by cno having count(*) > 5);
select tname from teacher where tno=(select tno from course where cno=(select cno from score group by cno having count(*) > 5));
```

-   查询 95031 和 95033 班全体学生的记录。 **1个用=,多个用in。**

```sql
select * from student where class='95031' or class='95033';
insert into student values('110','刘海','男','1974-06-05','95038');
select * from student where class in ('95031','95033');
```

-   查询存在有85分以上成绩的课程cno。

```sql
select * from score where degree>85;
select cno,degree from score where degree>85;
```

-   查询出'计算机系'老师所有课程的成绩。

```sql
select * from teacher where depart='计算机系';
select * from course where tno in (select tno from teacher where depart='计算机系');
select * from score where cno in (select cno from course where tno in (select tno from teacher where depart='计算机系'));
```

-   查询'计算机系'与'电子工程系'不同职称的教师的tname 和 prof。

```sql
select * from teacher;
select prof from teacher where depart='电子工程系';
select * from teacher where depart='计算机系' and prof not in (select prof from teacher where depart='电子工程系');
select * from teacher where depart='电子工程系' and prof not in (select prof from teacher where depart='计算机系');
select * from teacher where depart='计算机系' and prof not in (select prof from teacher where depart='电子工程系') union
select * from teacher where depart='电子工程系' and prof not in (select prof from teacher where depart='计算机系');
```

-   查询选修编号为'3-105'的课程且成绩至少高于编号为'3-245'的同学的cno, sno 和degree, 并按degree 出高到低依次排列。

```sql
select * from score where cno='3-245';
select * from score where cno='3-105';
select * from score where cno='3-105' and degree>any(select degree from score where cno='3-245');
select * from score where cno='3-105' and degree>any(select degree from score where cno='3-245') order by degree desc;
```

-   查询选修编号为'3-105' **且** 成绩高于选修编号为'3-245'课程的同学的 cno, sno 和 degree。
-   all()

```sql
select * from score where cno='3-105';
select * from score where cno='3-245';
select * from score where cno='3-105' and degree>all(select degree from score where cno='3-245');
```

-   查询所有教师和同学的name, sex 和 birthday。
-   as

```sql
select sname,ssex,sbirthday from student;
select tname,tsex,tbirthday from teacher;
select sname as name,ssex as sex,sbirthday as birthday from student union select tname as name,tsex as sex,tbirthday as birthday from teacher;
select sname as name,ssex as sex,sbirthday as birthday from student union select tname,tsex,tbirthday from teacher;
```

-   查询所有女教师和所有女同学的 name, sex 和 birthday。

```sql
select sname,ssex,sbirthday from student where ssex='女';
select tname,tsex,tbirthday from teacher where tsex='女';
select sname as name,ssex as sex,sbirthday as birthday from student where ssex='女' union select tname,tsex,tbirthday from teacher where tsex='女';
```

-   查询成绩比该课程平均成绩低的同学的成绩。

```sql
select * from score;
select avg(degree) from score group by cno;
select cno,avg(degree) from score group by cno;
-- ??? 这个我不太懂
select * from score a where degree < ( select avg(degree) from score b where a.cno=b.cno);
```

-   查询所有任课教师的tname 和 depart。

```sql
select * from teacher;
select tno from course;
select * from teacher where tno in (select tno from course);
select tname,depart from teacher where tno in (select tno from course);
```

-   查询至少有2明男生的班号。

```sql
select * from student where ssex='男';
select class,count(sname) from student where ssex='男' group by class;
select count(sname) from student where ssex='男' group by class;
select class from student where ssex='男' group by class having count(*) >1;
```

-   查询student 表中不姓王的同学。
-   not like

```sql
select * from student;
select * from student where sname not like '王%';
```

-   查询 student 表中每个学生的姓名和年龄。
-   年龄 = 当前年份 - 出生年份

```sql
select * from student;
select sname,sbirthday from student;
select year(now());
select sname,year(now()) - year(sbirthday) as '年龄' from student;
```

-   查询 student 表中最大的和最小的 sbirthday 日期值。
-   max() min()

```sql
select * from student order by sbirthday desc;
select max(sbirthday) as '最大年龄',min(birthday) as '最小年龄' from student order by sbirthday desc;
```

-   以班号和年龄从大到小的顺序查询 student 表中的全部记录。

```sql
select * from student order by class desc,sbirthday desc;
```

-   查询男教师及其所上的课程。

```sql
select * from teacher where tsex='男';
select * from course where tno in (select tno from teacher where tsex='男');
```

-   查询最高分同学的 sno, cno,和 degree 列。

```sql
select max(degree) from score;
select sno,cno,degree from score where degree=(select max(degree) from score);
```

-   查询和'李军'同性别的所有同学的 sname。

```sql
select * from student where sname='李军';
select * from student where ssex=(select ssex from student where sname='李军');
select sname from student where ssex=(select ssex from student where sname='李军');
```

-   查询和'李军'同性别并同班的所有同学的 sname。

```sql
select * from student where sname='李军';
select sname,class,ssex from student where ssex=(select ssex from student where sname='李军') and class=(select class from student where sname='李军');
```

-   查询所有选修'计算机导论'课程的'男'同学。

```sql
select * from course where cname='计算机导论';
select * from score where cno=(select cno from course where cname='计算机导论');
select * from student where ssex='男' and sno in (select sno from score where cno=(select cno from course where cname='计算机导论'));
```

-   假设使用下面的命令建立列一个grade 表：

```sql
create table grade(
low int(3),
upp int(3),
grade char(1)
);
insert into grade values(90,100,'A');
insert into grade values(80,89,'B');
insert into grade values(70,79,'C');
insert into grade values(60,69,'D');
insert into grade values(0,59,'E');

select * from grade;
```

-   查询所有学生的 sno cno 和 grade 列。

```sql
select sno,cno,grade from score,grade where degree between low and upp;
```


<a id="org26e5cdf"></a>

# Sql 的四种链接查询


<a id="org6d551ee"></a>

## 先建立所需的表

```sql
create database testJoin;
use testJoin;

create table person(
id int,
name varchar(20),
cardId int
);

create table card(
id int,
name varchar(20)
);

show tables;

insert into card values(1,'饭卡');
insert into card values(2,'农行卡');
insert into card values(3,'工商卡');
insert into card values(4,'建行卡');
insert into card values(5,'邮政卡');

insert into person values(1,'张三',1);
insert into person values(1,'李四',3);
insert into person values(1,'王五',6);

select * from person;
select * from card;
```


<a id="org324f9aa"></a>

## 内链接 inner join 或者 join

-   并没有创建外键
-   inner join 查询,通过两张表中的数据，通过某个字段相对查询出相关记录。

```sql
select * from person inner join card on person.cardId=card.id;
select * from person join card on person.cardId=card.id;
```


<a id="org9cb1db5"></a>

## (外链接)左链接 left join 或者 left outer join

-   left join 左外链接, 取出左边表中的数据，而右边表中的数据，如果有相等的，就显示出来，没有就显示null。

```sql
select * from person left join card on person.cardId=card.id;
select * from card left join person on card.id=person.cardId;
```


<a id="orge64cb1e"></a>

## (外链接)右链接 right join 或者 right outer join

-   right join 右外链接, 取出右边表中的数据，而左边表中的数据，如果有相等的，就显示出来，没有就显示null。


<a id="orgf3837f0"></a>

## 完全外链接 full join 或者 full outer join

-   full join 全外链接(mysql 不支持）

```sql
select * from person full join card on person.cardId=card.id; -- error. not support.
-- 等价于
select * from person left join card on person.cardId=card.id union select * from person right outer join card on person.cardId=card.id;
```


<a id="orgdae1aa0"></a>

# MySQL 事务

-   事务，其实是一个最小的不可分割的单元。事务能够保证一个业务的完整性。

-   比如银行转账

a-> -100

```sql
update user set money=money-100 where name='a';
```

b-> +100

```sql
update user set money=money+100 where name='b';
```

-   实际程序中，如果只有一条语句执行成功，而另外一条没有成功，会出现数据不一致的情况。
-   多条 SQL 语句，可能会有同时成功的要求，要么同时失败。


<a id="org70420e2"></a>

## 事务的四大特征，ACID。

-   a 原子性，事务是最小的单位，不可再分割。
-   c 一致性，事务要求，同一事务中的 sql 语句，必须保证同时成功或同时失败。
-   i 隔离性，事务1 和 事务2 之间具有隔离性。
-   d 持久性，事务一但结束，(commit,rollback),不可返回。

事务开启:

-   1，修改默认提交 set autocommit=0;
-   2, begin
-   3, start transaction;
    
    事务提交： commit。 事务手动回滚： rollback。


<a id="org3d7714d"></a>

## 隔离性:

-   如何查看数据库的隔离级别？

```sql
-- mysql 8.0: 系统级别，会话级别。
select @@global.transaction_isolation;
select @@transaction_isolation;

-- mysql 5.x:
select @@global.tx_isolation;
select @@tx_isolation;
```

-   如何修改隔离级别？

```sql
set transaction isolation level read uncommitted;
select @@transaction_isolation;

-- 全局设置
set global transaction isolation level read uncommitted;
select @@global.transaction_isolation;
```


<a id="org8ca8f0e"></a>

## mysql 如何开启事务？

-   mysql 默认开启事务的(自动提交)。

```sql
select @@autocommit;
```

-   先建立所需的表

```sql
create database bank;
use bank;
create table user(
id int primary key,
name varchar(20),
money int
);

insert into user values(1,'a',1000);
select * from user;
```

-   默认事务开启的作用是什么？ 当我们去执行一条 sql 语句时，效果会立即体现出来，且不能回滚。
-   事务回滚（rollback），撤销 sql 语句执行的效果。

```sql
rollback;
select * from user;
```

-   设置默认事务为0的方式。自动提交为 false。

```sql
set autocommit=0;
select @@autocommit;
```

```sql
insert into user values(2,'b',1000);
select * from user;
rollback;
select * from user;
```

-   再插入一次数据, 并且手动提交，不可回滚。

```sql
insert into user values(2,'b',1000);
commit;
rollback;
select * from user;

insert into user values(1,'a',1000);
insert into user values(2,'b',1000);

update user set money=money-100 where name='a';
update user set money=money+100 where name='b';
select * from user;
rollback; 
select * from user;
```

-   事务给我们提供了一个反悔的机会。
-   begin 或者 start transaction; 都可以帮我们手动开启一个事务。

```sql
-- begin 
select * from user;
rollback;
select * from user;
begin;
update user set money=money-100 where name='a';
update user set money=money+100 where name='b';
select * from user;
rollback; 
select * from user;

-- start transaction;
select * from user;
rollback;
select * from user;
start transaction;
update user set money=money-100 where name='a';
update user set money=money+100 where name='b';
select * from user;
rollback; 
select * from user;
commit;
```

-   事务开启后，一旦 commit 提交，就不可回滚（也就是当前的这个事务在提交的时候就结束了）;


<a id="org8b0aa23"></a>

## 1. read-uncommitted; 读未提交的

-   例子：如有事务 a 和 事务 b, 在操作的过程中， 事务没有被提交，但是 b 可以看见 a 操作的结果。

```sql
insert into user values(3,'小明',1000);
insert into user values(4,'店铺',1000);
```

-   转账：小明在店铺买列鞋子：800;

```sql
start transaction;
update user set money=money-800 where name='小明';
update user set money=money+800 where name='店铺';
select * from user;
```

-   给店铺打电话，让查帐。

```sql
select * from user;
```

-   发货，晚上请女友吃返，1800, 发现钱不够。
-   小明 rollback;
-   钱回来了

-   如果两个不同的地方，都在进行操作，如果事务a 开启之后，他的数据可以被其他事务读取到，这样就会出现（脏读）。
-   一个事务读到了另一个事务没有提交的数据，实际开发是不允许脏读的。


<a id="org8379851"></a>

## 2. read committed; 读已提交的

-   小张： 银行的会计

```sql
start transaction;
select * from user;
```

-   小张上厕所去了

-   小王

```sql
start transaction;
insert into user values(5,'c',100);
commit;
select * from user;
```

-   小张上厕所回来

```sql
select avg(money) from user;
```

-   money 的平均值，变少了。
-   虽然我只能读到你提交的数据，但还是会出现问题，就是不可重复读现象。 read committed;


<a id="org77f2939"></a>

## 3, repeatable read; 可以重复读的

-   银行职员A，成都

```sql
start transaction;
```

-   银行职员B，北京

```sql
start transaction;
```

-   银行职员A，成都

```sql
insert into user values(6,'d',1000);
```

-   银行职员B，北京

```sql
select * from user;
insert into user values(6,'d',1000);
```

-   这种现象就叫做幻读。事务a 和 事务b 同时操作一张表，事务a 提交的数据，也不能被事务b 读到，就可以造成幻读。


<a id="org4a8ed12"></a>

## 4. serializable; 串行化

-   银行职员A，成都

```sql
start transaction;
```

-   银行职员B，北京

```sql
start transaction;
```

-   银行职员A，成都

```sql
insert into user values(7,'诸葛门',1000);
commit;
select * from user;
```

-   银行职员B，北京

```sql
select * from user;
```

-   银行职员A，成都

```sql
start transaction;
insert into user values(8,'魏大勋',1000);
-- *会卡在这个地方*
```

-   银行职员B，北京

```sql
select * from user;
commit;
```

-   银行职员A，成都
-   提交成功

-   当 user 表中被另外一个事务操作的时候，其它事务里面的写操作，是不可以进行的。进入排队状态（串行化），直到另一边的事务结束，这个写操作才会进行，张没有等待超时的情况下。
-   带来的问题：性能特差！！！


<a id="org8bbf9b6"></a>

## 隔离级别

-   read-uncommitted > read-committed > repeatable-read > serializable;
-   隔离级别越高，性能越差。
-   mysql 默认隔离级别是，repeatable-read;


<a id="orgc383441"></a>

# 注

-   本文是观看 **一天学会 MySQL 数据库** 是的学习笔记。如有错误，欢迎批评指正。[视频链接](https://www.bilibili.com/video/BV1Vt411z7wy?p=1)

