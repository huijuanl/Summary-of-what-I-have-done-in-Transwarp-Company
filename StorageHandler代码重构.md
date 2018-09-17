# StorageHandler代码重构

标签（空格分隔）： elasticsearch

---
[TOC]

 
目的：重构之后，需精简原有的create/drop/alter/truncate方法实现，统一主表和分区表的ddl逻辑

语义(SQL语句->Es操作一一对应)
------------------------

* SQL DDL基本操作(create,truncate,alter,drop)
* 分区表/非分区表  内表/外表
分区表:
(1)主表(非分区表)：保存源信息，不保存插入的数据
(2)分区表：保存插入的数据
一个分区对应一个es index
* 建分区表
```
create table test001 (key string,sv1 string,sv2 string,sv3 double,sv4 int,sv5 boolean) partitioned by range (sv1)stored as es;

alter table test001 add partition a values less than ('20121223') location 'star:///esdrive/esdrive_partition.hd_range_part1';

alter table test001 add partition b values less than ('20121226') location 'star:///esdrive/esdrive_partition.hd_range_part2';
```
![image.png-39.7kB][4]

* 建外表
```
CREATE EXTERNAL TABLE esdrive_external_table(
        key1 string,
        ex0 int,
        ex1 bigint,
        ex2 double,
        ex3 string
)STORED BY 'io.transwarp.esdrive.ElasticSearchStorageHandler'
WITH SERDEPROPERTIES ('elasticsearch.columns.mapping'='_id,sv0,sv2,sv5,sv7')
TBLPROPERTIES ('elasticsearch.tablename'='default.esdrive_inner_table');
```
* DDL
![image.png-69.3kB][1]
新架构：抽象类DefaultStargateStorageHandler
------------------------
![image.png-76.4kB][2]

ElasticSearchStorageHandler旧架构
-----------
**commitCreateTable():**
实现功能：
(1)创建非分区表（主表）
**createOrTruncateTable():**
实现功能：
(1)create分区表
(2)truncate非分区表(主表)/分区表

缺点：可读性不够强，杂糅，代码冗余，不符合新架构的要求
ElasticSearchStorageHandler重构
----------------------
* Create 重构逻辑
  (1) check
  (2) 真正建表
![image.png-69.3kB][1]

![image.png-69.4kB][3]
非分区表:preCreateTable()-->CreateFactTable()
分区表:CreateFactTable()
```
CreateFactTable():
(1)如果不是分区表：如果为内表，
    a.从table中取出建表所需的settings,mappings等源信息
(2)如果是分区表：
    a.检查合法性；
    如果是内表:
    b.与Es通信取出主表的settings,mappings源信息
(3) 根据源信息建相应的Es表(createIndicesOnEs())
```


* Truncate 重构
truncateFactTable()
非分区表/分区表的truncate是统一的
**truncate步骤:**
```
truncateFactTable()
表已经存在的前提下:
(1)与Es通信取出主表的源信息保存
(2) 将对应的Es表删掉(deleteIndicesOnEs())
(3)根据源信息建相应的Es表(createIndicesOnEs())
```

**create分区表的逻辑和truncate主表或分区表的逻辑是类似的**

* drop
对于主表和分区表的逻辑是一样的
```
将对应的Es表删掉deleteIndicesOnEs（）
```
create/truncate/drop重构总结
-------------
* check逻辑
  (1)非分区表preCreateTable()
     分区表createFactTable()
  (2)抛异常：规范
  create分区外表，抛异常(新增)
  truncate分区外表是否需要抛异常？
* 弃用createOrTruncateTable()
* 根据create和truncate共性的部分抽出内联函数createIndicesOnEs:（根据mapping,setting,indexname等创建Es表），（原来只是建主表调用，现在可被create/truncate共享）
* truncate/drop共用deleteIndicesOnEs（）函数

alter
----------------
alter的用法：

* alter tablename add partition（create命令）
* 增加新列
(对于alter之前已经insert的数据，相应的add_col1列值为null。修改test001的主表，分区表的mapping，增加add_col1的信息)

![image.png-25.7kB][5]

![image.png-27.4kB][6]
```
alter table test001 add columns (add_col1 int);
```
![image.png-32kB][7]

![image.png-29.6kB][8]

* 修改Es表的shard信息

![image.png-39.7kB][9]
```
alter table  test001 set tblproperties ('number_of_replicas' = '2');
```
![image.png-36.6kB][10]

* alter重构
alterFactTable():
alterFactTable()步骤：
```
(1)check:如果alter外表，就抛出异常
(2)alter具体的功能
```
alter暂时没有需要重构的地方





  [1]: http://static.zybuluo.com/lihuijuan114/gu6n5ttvv8r1dk9ncqf9pmem/image.png
  [2]: http://static.zybuluo.com/lihuijuan114/tgofg93l78s4lzb74hhn134a/image.png
  [3]: http://static.zybuluo.com/lihuijuan114/8w5luslwwa6h1y0j9xvlm074/image.png
  [4]: http://static.zybuluo.com/lihuijuan114/rnjzarggbls7glo4ou1ttj09/image.png
  [5]: http://static.zybuluo.com/lihuijuan114/j3f66rybz2q1p5p2f9g0zxhm/image.png
  [6]: http://static.zybuluo.com/lihuijuan114/zorq0rehlwggjgiccbs4gvni/image.png
  [7]: http://static.zybuluo.com/lihuijuan114/n40udpr73r2oldcsa8gev1v9/image.png
  [8]: http://static.zybuluo.com/lihuijuan114/d5vxv438dhaqj4c1nhsdtxbf/image.png
  [9]: http://static.zybuluo.com/lihuijuan114/rnjzarggbls7glo4ou1ttj09/image.png
  [10]: http://static.zybuluo.com/lihuijuan114/un21lc4kjavklmph231kyc7x/image.png
