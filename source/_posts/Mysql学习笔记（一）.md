title: Mysql学习笔记（一）
author: Zhiyuan
tags:
  - Mysql
categories:
  - 学习笔记
date: 2019-09-21 19:29:00
---
DDL语句  
1、建库  

```
CREAT DATABASE  名字
```


不区分大小写，推荐关键字大写；单词间加‘_’  
mysql数据结构：char，varchar  

```
CREATE TABLE tm_teammanager( 
		id INT(11),  
		'name'  VARCHAR(30),#通过飘号选中的字符会保留字符串本意，不会当成关键字，~  
		gender CHAR(2)
)
```

2、查表结构  

```
DESC tm_teammanager
```

3、从删库到跑路  
备份为.sql文件  

```
DROP TABLE 表名  
DROP DATABASE 
```

4、修改表名  

```
ALTER TABLE oldname RENAME newname;
```

5、给表添加一列  

```
ALTER TABLE 表名  ADD COLUMN 名字 INT(3) (加上类型)
```

6、删除表一列  

```
ALTER TABLE 表名 DROP COLUMN 列名
```

7、给一列重命名change  

```
ALTER TABLE 表名 CHANGE 列名 新列名 数据类型
```

8、修改表列的数据类型modify  

```
ALTER TABLE 表名 MODIFY 列名 新数据类型
```

DML语句  
1、插入数据   

```
INSERT INTO 表名 VALUES();  #必须给所有列指定值  
INSERT INTO 表名(id,列名……)  VALUES(对应值);
```

2、查询 select

3、修改数据 update  

```
UPDATE 表名 SET 列名=值 WHERE id=2；
```

4、删除 delete  

```
DELETE FROM 表名  
delete、drop、truncate区别
```

约束 constrains  
1、主键约束 primary key（字段值唯一非空）主键只能有一个  
1.1添加主键：创建表时添加约束CONSTRAINT PRIMARY KEY(列名)  
1.2 通过alter语句给表添加主键约束（表创建后添加）  

```
ALTER TABLE 表名 DROP PRIMARY KEY; #先删除原有主键约束  
ALTER TABLE 表名 ADD PRIMARY KEY(列名)  
或 ALTER TABLE 表名 MODIFY …… PRIMARY KEY
```

2、唯一键约束  
2.1 创建时添加  
2.2 修改表的指定列添加唯一键约束  

```
ALTER TABLE ^ MODIFY ^ ^ UNIQUE;  
```

删除唯一键约束  

```
ALTER TABLE 表名 DROP INDEX 列名；  
```

查看索引 SHOW INDEX FROM 表名

3、非空约束 not null  
3.1创建表时添加  

```
CREATE TABLE t_stu(  
&emsp;&emsp;id INT(11) NOT NULL;  
);  
```

删除非空约束  

```
alter table 表名 modify 列名 类型；
```

3.2修改列添加非空约束  

```
alter table 表名 modify 列名 类型 not null default “etc”；  
```

4、默认值约束 default  
5、外键约束  
外键关联时，两张表建立外键的列数据类型必须相同，长度尽量一致

5.1修改表的指定列，添加外键约束  

```
ALTER TABLE 表名 ADD CONSTRAINT 外键名 FOREIGN KEY （当前表的指定列） references 关联的表名（指定要关联的列名）;

ALTER TABLE t_stu ADD CONSTRAINT fk_cid FOREIGN KEY(cid) REFERENCES t_course(id);
```

删除外键约束：删除外键，然后才能删除外键对应的索引  

```
ALTER TEBLE 表名 DROP FOREIGN KEY(外键名)  
ALTER TABLE 表名 DROP INDEX 外键名；  
```

索引需要删除，否则影响效率

5.2创建表时添加  

```
CONSTRAINT 外键名 FOREIGN KEY(列名) REFERENCES关联的表名（列名）
```

5.3删除被其他表关联的数据：当没有其他表引用该行数据可以删除
5.4添加外键时指定级联删除或者级联置空





