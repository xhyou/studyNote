# MySql的架构介绍

## 安装

> 在linux下安装

## 设置开启自启动

> chkconfig mysql on
>
> ntsysv 查看自启动的服务有哪些 【*】代表开机自启动

## 安装位置

| 路径              | 解释                      | 备注                       |
| ----------------- | ------------------------- | -------------------------- |
| /var/lib/mysql/   | mysql数据库文件的存放路径 |                            |
| /usr/share/mysql  | 配置文件目录              | mysql.server命令及配置文件 |
| /etc/init.d/mysql | 启停相关的脚本            |                            |

## 修改配置文件

> 5.5版本  /usr/share/mysql/my-huge/cnf  
>
> 拷贝 cp  /usr/share/mysql/my-huge/cnf   /etc/my.cnf
>
> 高版本  /usr/share/mysql/my-defalt.cnf   /etc/my.cnf

## 修改默认字符集编码

```shell
1.查看字符集

show variables like ‘%char%’

2.在 /etc/my.cnf中[client]设置

   default-charater-set=utf8

   [mysqld]中添加

   charater_set_server=utf8
   charater_set_client=utf8
   charater_server=utf8_general_ci
   
   [mysql]
   
   default-charater-set=utf8
```

## MySql配置文件

二进制日志文件：log-bin(主从复制)  window在==my.ini==  linux在 ==/etc/my.cnf==

错误日志log-error:默认是关闭的（mysqlerr.err文件）

查询日志：默认关闭

数据文件：1.frm文件 -> 存放表结构

​				  2.myd文件 -> 存放表数据

​			      3.myi文件  -> 存放索引

## MySql逻辑校验

 连接层

 服务层:执行SQL服务

 存储引擎:MyISAM(不支持主外键,不支持事务、表锁、只缓存索引)、innoDB(支持行锁和事务)

 储存层

## 存储引擎的介绍

  查看存储引擎:

```shell
方式一：show engines;
方式二：show variables like '%storage_engine%';
```

## SQL查询速率慢的原因

```sql
1.性能下降sql慢

2.执行时间长

3.等待时间长

原因：

1.查询语句写的烂

2.索引失效->{单值，复合}
  
   例子：创建单值索引 create index idx_index_name from user(name);
   	     创建复合索引 create index idx_index_name_email from user(name,eamil);
   	     
   单值索引: select * from user where name =''  查询一个条件

   复合索引: select * from user where name ='' and email=''
   
3.查询关联太多join(设计缺陷或者不得已的需求)  

4.服务器调优及各个参数设置(缓冲、线程等)
```

## SQL的执行顺序

```sql
 FROM <left table>        	  //先是一个笛卡尔积
 ON  <join condition>   
 <join type> JOIN <right table>
 WHERE <where condition>
 SELECT
 DISTINCT <select_list>
 ORDER BY <order_by_condition>
 LIMIT <limit_number>
```

```sql
案例：A表,B表
1.A表和B表的交集
 select * from A inner join B on A.key =B.key
2.A表和B表的共有加A表独有
 select * from A left join B on A.key=B.key
3.A表和B表的共有加B表的独有加
 select * from A right join B on A.key =B.key
4.A表独有
 select * from A left join B on A.key = B.key where B.key is null
5.B表独有
 select * from A right join B on A.key = B.key where A.key is null
6.A表和B表的全部
 select * from A full outer join B on A.key = B.key
7.A表独有和B表独有的
 select * from A full outer join B on A.key =B.key where A.key is null and B.key is null 
 //注意 mysql不支持 6 7的写法,使用union
```

## 索引（重要）

* 索引是帮助MySQL高效获取数据的==数据结构==

* 索引：**==排好序的快速查找的数据结构==**

* 影响的范围：查找和排序(where 和 order by)

优势：

   1.提高了索引的小效率,降低了数据库的IO成本

   2.通过索引对数据进行排序,降低数据排序的成本,降低cpu的消耗

劣势：

   1.索引实际也是一张表,需要占用空间的。

   2.索引提高了查找速度,但是会降低更新表的速度。因为更新表的时候,不仅要保存数据还要添加索引的字段

### 索引的分类

* 单值索引：一个索引值包含单个列,一个表可以有多个单列索引
* 唯一索引：索引列的值必须唯一,但允许有空值
* 复合索引：一个索引包含多个列

```sql
--创建索引
create [unique] index indexName on mytable(columnName(length))
alter mytable ADD [unique] index [indexName] on(columnName(length))
--删除索引
drop index[indexName] on mytable
--查看索引
show index from table_name 

--使用Alter来创建索引的四种方式
ALTER TABLE tab_name ADD PRIMARY KEY (column_list);添加一个主键,意味着索引值是唯一,不为NUlL
ALTER TABLE ADD UNIQUE INDEX_NAME(column_list);这条语句创建的索引值唯一(除NUll外)
ALTER TABLE tab_name ADD INDEX index_name(column_list);添加普通索引,索引值可出现多次
ALTER TABLE tab_name ADD FULLTEXT index_name(column_list);添加全文索引
```

### MySql索引结构

* Btree索引

### 哪些情况创建索引

* 主键自动建立唯一索引
* 频繁查找的条件
* 查询中与其他边建立关联的字段,外键关系建立索引
* Where条件用不到的字段不要创建索引
* 单值/组合索引的选择问题？(在高并发下建议创建组合索引)
* 查询字段中排序字段,排序字段若通过索引去访问将大大提高排序效率
* 查询中统计或则分组字段（疑问）

### 哪些情况不适合创建索引

* 表记录太少
* 频繁更新的字段不适合创建索引
* 数据重复并且分布平均的字段（假设一个字段只有True和false,分布概率大概都为50%）

### 性能分析

1.Mysql Query Optimizer SQL的服务

2.MySQL常见的瓶颈：

   2-1：CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据的时候

   2-2：磁盘I/O瓶颈发生在装入数据远大于内存容量的时候

   2-3：服务器硬件的性能瓶颈

### Explain

执行计划：使用Explain关键字可以模拟优化器执行SQL查询语句,从而知道Mysql是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈。

Explain能做什么？

* 表的读取顺序
* 数据读取操作的操作类型
* 哪些索引可以使用
* 哪些索引被实际使用
* 表之间的引用
* 每张表有多少行被优化器查询

使用Explain包含的信息

| id   | select_type | table       | type | possible_keys | key  | key_len | ref  | rows | Extra |
| ---- | ----------- | ----------- | ---- | ------------- | ---- | ------- | ---- | ---- | ----- |
|      |             | DERIVED衍生 |      |               |      |         |      |      |       |

* id

```sql
id select查询的序列号,包含一组数字,表示查询中执行select子句或操作表的顺序
三种情况
1.id相同,执行顺序由上至下
2.如果是子查询,id的序号会递增,id值越大优先越高
3.id如果相同,可以认为是一组,从上往下顺序执行；在所有组中,id值越大,优先级越高,越先执行
```

* select_type

```sql
1.simple
  简单的select查询,查询中不包含子查询或者union
2.primary
  查找中若包含任何复杂的子部分,最外层被标记为primary 
3.subquery
  在select或者where中包含了子查询
4.derived
  在from列表中包含的子查询被标记为DERIVED(衍生)Mysql会递归执行这些子查询,把结果放在临时表里
5.union
  若第二个select出现在union之后,则被标记为union。若union包含在from子句的子查询中，外层的select被标   记为derived  
6.union result
  从union获得的select(两个union合并的结果)
```

* type

```sql
1.All
2.index
  index、和all都是读全表,但是index是从索引中读取的,all是从硬盘读取的
  （select id from A id为主键索引）
3.range
  检索给定范围的行,使用一个索引来选择行(比如between、in)
4.ref
  非唯一性的所有扫描，返回匹配某个单独值的所有行。
  (一个公司的所有程序员)
5.eq_ref
  唯一性索引扫描，对于每个索引键，表中只有一条记录来与之匹配。常见与主键或者唯一索引扫描
  (一个公司的CEO)
6.const
  表示通过索引一次就找到了,const用于比较primary或者unique索引。
7.system
  表只有一条记录（等于系统表）,特例一般不会出现
8.null
访问类型的排序,从最好到最差(一般达到range或者ref就可以)
system>const>eq_ref>ref>range>index>all
```

* possible_keys 和 key

```sql
possible_keys  显示可能应用到这张表的索引,可能一个或者多个
			   查询涉及到的字段上若存在索引,则该索引将被列出，但不一定被实际使用
key:实际使用的索引,如果为null，则没有使用索引
    查询中若使用了覆盖索引,则该索引仅出现在key列表中
    覆盖索引：select查询的字段和我创建的复合索引的个数和顺序一一一致
```

* key_len

```
标识索引中使用的字节数，可通过该列计算查询中使用的索引的长度。越短越好
```

* ref

```
显示一个索引的哪一列被使用了,如果可能得话是一个常数
```

* rows

```
根据表统计的信息及索引选用情况,大致估算出找到所需的记录需要读取的行数(越少越好)
```

* Extra

```
包含不适合在其他列中显示但十分重要的额外信息

Using_filesort:说明mysql会对数据使用一个外部的索引排序,而不是按照表内的索引顺序进行读取.MySql无法利用索引完成排序操作称为"文件排序"(最好不要出现)
Using_temporary(新建了一个临时表):使用了临时表保存中间结果,Mysql在对查询结果排序时使用临时表。常见于order by和分组查询group by  (不能有)
Using_index:表示相应的select操作中使用了覆盖索引(Covering index)，避免访问表的数据行,效率不错
			如果同时出现using where,标识索引被用来执行索引键值的查找
			如果没有同时出现using where，表明索引引用来读取数据而非执行查找动作
Using where:使用到了where
Using join buffer:使用了连接缓存;
imposible where:where子句中的值总是false，不能用来执行任何元素
例子：select * from a where a.name='A' and a.name ='B' name不可能同时是'A'和'B'
select tables optimized away:(了解)
dinstinct:(了解)
```

```sql
例子：
select * from tbl_emp;
使用执行计划
explain select * from tbl_emp;
```

### 索引优化





# 查询的截取分析