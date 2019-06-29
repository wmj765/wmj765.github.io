title: mysql 拆分以逗号隔开的字段并应用在in查询
catalog: true
author: wangmj
header-img: Demo.png
tags: []
categories: []
date: 2017-04-18 18:26:00
subtitle:
---
# mysql 拆分以逗号隔开的字段并应用在in查询

利用substring_index及笛卡尔积来循环拆分sql字段；

首先建立几条数据

| id      |    value |
| :-------- |  :--: |
| Computer  |  5,ddd,eee   |
| Phone     |    12,3333,11  |
| Pipe      |   234  |

假如我们需要查询的字段in phone字段的值(12,3333,11)，我们可以用一下方法

###具体方法
```python
准备示例数据
create table tbl_name (ID int ,mSize varchar(100));
insert into tbl_name values (1,'tiny,small,big');
insert into tbl_name values (2,'small,medium');
insert into tbl_name values (3,'tiny,big');

实现行列转换的SQL
select * from table where phone in (
	select  
		substring_index(substring_index(a.mSize,','
		,b.help_topic_id+1),',',-1) 
	from 
		tbl_name a
	join
		mysql.help_topic b
		on b.help_topic_id < (length(a.mSize) -     length(replace(a.mSize,',',''))+1)
	order by a.ID;
)
```


**原理分析**　
这个join最基本原理是笛卡尔积。通过这个方式来实现循环。 以下是具体问题分析： length(a.Size) - length(replace(a.mSize,',',''))+1 表示了，按照逗号分割后，改列拥有的数值数量，下面简称n join过程的伪代码：
```python
根据ID进行循环
{
    判断：i 是否 <= n
    {
        获取最靠近第 i 个逗号之前的数据， 即 substring_index(substring_index(a.mSize,',',b.ID),',',-1)
        i = i +1 
    }
    ID = ID +1 
}
```
以上及实现了字段按逗号分隔来用作in的子查询