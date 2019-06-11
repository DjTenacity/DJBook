### 数据库小demo

#### 设计班级表，与学生表关联，并进行查询

create table T_classes(
id  int auto_increment primary key not null,
name varchar(10));

create table T_students(
id int auto_increment primary key not null,
name varchar(10),
classid int,
FOREIGN key(classid) REFERENCES T_classes(id));

create view V_stu_cls as
select T_students.id,T_studtents.name,T_classes.name as title
from T_sudents
inner join T_classes on T_classes.id=T_students.classid;

#### 设计分类表，自关联，并进行查询
create table T_type(
id int auto_increment primary key not null,]
name varchar(10),
pid int,
FPREIGN key(pid) references T_type(id));

#### 创建视图存储上面的两个查询
create view V_type as
select sun. * from T_type as father
inner join T_type as son on father.id=son.id


#### 教室id作为学生的 外键
create table rooms(id int primary key not null,
title varchar(10));

create table stu(id int primary key auto_increment not null,
name varchar(10),roomid int );

//添加外键
alter table stu add constraint stu_room foreign key(roomid) references
rooms(id);

insert into rooms values('314','教室铭');
//放在添加314 之前就会报错
insert int0 stu values('1','小名',314);