title: Mysql学习笔记（三）
author: Zhiyuan
tags:
  - Mysql
categories:
  - 学习笔记
date: 2019-09-21 20:17:00
---
select 字段列表
form 表名
where 查询筛选条件
group by 指定分组的列列表
having 指定分组后的筛选条件
order by 排序列的列表
limit 起始索引值，要查询的记录条数

1、分组查询：group by
分组函数（聚合函数）：将多行的值进行统计返回一行结果
sum() ; avg();  min();  max(); count().
注：分组函数不能和与分组无关的列一起使用；
分组查询时，查询的列可以是分组的条件

```
SELECT SUM(salary)，COUNT(eid)，did 
FROM t_employ
GROUP BY did;

SELECT did,COUNT(eid)
FROM t_employ
WHERE salary>10000
AND did IS NOT NULL
GROUP BY did;   
```

2、条件中需要使用分组后的结果:having作用和where一样，但是执行顺序在分组之后

```
SELECT AVG(salary)
FROM t_employ
GROUP BY did
HAVING AVG(salary)>9000;
```

3、order by：降序排列desc，升序排列asc
`SELECT * FROM t_employ ORDER BY salary DESC, eid ASC;`

4、分页查询：limit index，size

5、子查询：当前查询依赖另一个查询的结果

```
SELECT * FROM t_employ WHERE did=(
        SELECT did FROM t_employ WHERE ename='罗宾'
)；
```

三种类型：1、将子查询结果当做where查询的条件；2、当做临时表再次查询；3、作为主查询的判断条件，决定数据是否查询。
exists型子查询

6、复制表
 复制表结构：`CREATE TABLE t_emp AS(SELECT * FROM t_employ WHERE 1=2);`
复制表结构+指定数据：where 条件 

7、单行函数：处理一行返回一行结果
`SELECT UPPER(email) FROM t_employ;`
password():mysql数据库在将数据库管理员的信息进行保存时，会将密码加密保存。
`SELECT PASSWORD('123456') FROM DUAL;   (dual练习表)`

