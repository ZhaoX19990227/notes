### 执行顺序

```mysql
from--->where---->group by--->having--->select--->distinct---->union----->order by
```

#### group_concat

```sql
-- 将符合条件的一列数据转行成一行数据 SEPARATOR分隔符 默认是逗号
select GROUP_CONCAT(receiver_name ORDER BY DESC id desc SEPARATOR ';') from oms_order GROUP BY id
```

```sql
SELECT ABS(-123)  取绝对值
SELECT CEIL(1.5)  向上取整
select FLOOR(1.5)	向下取整
select GREATEST(1,2,3) 最大值
SELECT LEAST(1,2,3)  最小值  如有null结果就是null
和max和min的区别在于：max和min要求输入的是一个表达式或者是一个列名
SELECT MOD(5,2) 取余数

```

#### 递归查询

```sql
-- 递归的常见形式
WITH CTE AS (
SELECT column1,column2... FROM tablename WHERE conditions
UNION ALL
SELECT column1,column2... FROM tablename 
INNER JOIN CTE ON conditions 
)

-- 示例
with recursive cte as
(
select * from areas where id = 12
union all
select c.* from areas c, cte where c.id = cte.pid
)
select * from cte order by pid, id asc
id name  pid
1	 中国	  0
2	 北京市	1
12 海淀区	2

先通过select * from areas where id = 12找到海定区这条数据
然后结果作为条件找到id=2的北京市的 联合在一起作为cte返回

接着又进行递归查询到id=1的中国数据
```



### 窗口函数

聚合函数是将一组数据返回一条数据（即分组），窗口函数可将窗口范围内的数据输入到聚合函数中，并不改变行数。

```sql
-- 对每个部门的员工按照薪资排序，并给出排名   
-- ROW_NUMBER()和RANK()一样，但是RANK遇到一样的数据会显示一样的序号。1 1 1 4
-- PARTITION by 分组
-- 还有个dense_rank() 遇到重复的数据会显示一样的序号，但是后面不一样的数据的需要也是紧跟的 1 1 1 2 3
select dname,ename,salary, ROW_NUMBER() over(PARTITION by dname order by salary DESC) as rn from employee;

dname ename salary rn
研发部	章三	30000		1
研发部	赵翔	20000		2
研发部	余晓桅	10000  3
销售部	王五	50000		1
销售部	李四	40000		2
```

### 表加锁（MyISAM支持表锁，Innodb支持行锁）

```sql
MyISAM 在执行查询前会自动加读锁，更新前会自动加写锁。
加了读锁后只能读，不能修改 读锁是共享锁，可以多个人进行加读锁，都可以读，并且都不能修改
注意，加了读锁后，只能对当前表进行读取，不能再读其他表
unlock tables；  -- 释放锁  只对当前有效，其他人都需要进行解锁才可以读取其他表的数据
写锁是独享锁，只有自己释放锁后，其他人才能读取数据，否则一直阻塞
而且写锁只能加一个
```

#### 读锁

```sql
lock table table_name read;
```

#### 写锁

```sql
lock table table_name write;
```

#### 行锁

Innodb实现了两种类型的锁：

1. 共享锁	又称读锁  多个事务对于同一组数据可以共享同一把锁，都能访问到数据，只能读不能修改
2. 排他锁    又称写锁  如一个事务获取了一个数据行的写锁，其他事务就不能再获取当前行的任意锁，但是获取拍他锁的事务可以进行读取和修改。
3. 对于更新，Innodb自动加上排他锁   读取不会加锁。

```sql
# 共享锁
select ... lock in share mode
# 排他锁
select ... for update
```

### 日志

#### 错误日志

```sql
-- 查看错误日志目录
SHOW VARIABLES LIKE 'log_error%'
-- 查看是否开启binlog日志
show variables like 'log_bin'
-- 查看使用的日志格式
show variables like 'binlog_format'
```

#### 二进制日志BINLOG

记录了所有DDL（定义）和DML（操作），但是不包括查询语句，对于灾难时的数据恢复起着重要作用，MySQL的主从复制，就是通过binlog实现的。

```sql
# MySQL8.0默认开启，低版本需要在my.ini文件中追加配置
log_bin=mysqlbin
# 配置二进制日志的格式
-- STATEMENT：记录的都是sql语句，每一条对数据进行修改的sql都会记录，通过mysqlbinlog工具可以查看。主从复制的时候，从库会将日志解析为原文本，并在从库中执行一次。
-- ROW：记录的是每一行的变更，而不是sql语句
-- MIXED：混合以上两种模式
binlog_format=STATEMENT/ROW/MIXED
# 重启mysql服务
```

#### 慢查询日志

```sql
-- 查看是否开启慢查询日志
SHOW VARIABLES LIKE 'slow_query_log%' 
-- 开启慢查询日志
set GLOBAL slow_query_log = 1
-- 查看多久才会被记录慢查询
show VARIABLES LIKE 'long_query_time%'  -- 默认10s
```

###  索引

```markdown
- 主键索引 primary key
- 唯一索引 unique  一张表可以有多个唯一索引
- 普通索引 index 普通字段上的索引 normal
- 复合索引 一个索引包含多个列
```

```sql
# 查看索引
show index from table_name;
# 创建索引
① 普通索引
create table t_dept(
    no int not null primary key,
    name varchar(20) null,
    sex varchar(2) null,
    info varchar(20) null,
    index index_no(no)
  )
② 唯一索引
create table t_dept(
       no int not null primary key,
       name varchar(20) null,
       sex varchar(2) null,
       info varchar(20) null,
       unique index index_no(no)
     ）

③ 全文索引
create table t_dept(
       no int not null primary key,
       name varchar(20) null,
       sex varchar(2) null,
       info varchar(20) null,
       fulltext index index_no(no)

④ 多列索引
create table t_dept(
       no int not null primary key,
       name varchar(20) null,
       sex varchar(2) null,
       info varchar(20) null,
       key index_no_name(no,name)
     )
① 普通索引
create index index_name on t_dept(name);
② 唯一索引
create unique index index_name on t_dept(name);
③ 全文索引
create fulltext index index_name on t_dept(name);
④ 多列索引
create index index_name_no on t_dept(name,no)
#删除索引
drop index index_name on table_name;
#修改表创建索引
alter table table_name add index_type  index_name(colume_list)
```

```markdown
# 聚簇索引和非聚簇索引
```

在innoDB中，在聚簇索引之上建立的索引都叫辅助索引。

非聚簇索引都是辅助索引，辅助索引的叶子结点不再存储数据，而是主键值，辅助索引访问数据需要二次查找。

```markdown
# InnoDB和MyISAM区别《形容数据表的》
1.MyISAM用两个文件myi和myd分别存储索引和数据，查找到索引后还要去另一个文件查具体的数据
2.InnoDB只有一个文件,通过找到索引后就可以直接在树上找到对应的数据
3.MyISAM天然支持表锁，InnoDB支持行锁
```

 ```markdown
 # 什么是最左前缀原则？
 ```

最左优先，在创建多列索引时，要根据业务需求，where 子句中使用最频繁的一列放在最左边。

当我们创建一个组合索引的时候，如 (a1,a2,a3)，相当于创建了（a1）、(a1,a2)和(a1,a2,a3)三个索引，这就是最左匹配原则。

### 范围查询字段后面的索引也会失效，此时c将在索引树中无法被查询

```sql
select * where a=1 and b>1 and c=1
```

### or查询中，如果有一个字段没有建立索引，那么整个索引也会失效，直接走主索引

```sql
select * where x=1 or y=1
```

### 为什么order by 会导致索引失效？

因为MySQL认为即使 使用 索引树进行查询，不需要排序，但是最后仍然需要回表，这种代价比全表扫描然后排序的代价更大。

### 字段操作导致索引失效

mysql中'123'=123 返回1  'a'会被转化成0，所有非数字字符都会被转化成0。对索引字段进行运算会导致索引失效，因为需要进行运算或者类型转换，效率会降低，并且可能进行类型转换后不再是一颗B+树。

### 索引有哪些优缺点？

(1) 优点：

- 唯一索引可以保证数据库表中每一行的数据的唯一性
- 索引可以加快数据查询速度，减少查询时间

(2)缺点：

- 创建索引和维护索引要耗费时间
- 索引需要占物理空间，除了数据表占用数据空间之外，每一个索引还要占用一定的物理空间
- 以表中的数据进行增、删、改的时候，索引也要动态的维护。

### 走索引还是全表扫描

走索引就是从根节点开始扫描，如果存储的数据不是在非叶子结点，就只能对叶子结点进行从左往右全表扫描

如果查询where a>6，则会先找到a=6，然后将前面的叶子结点数据返回即可，因为都是排序好的。

### 索引覆盖

只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。

```markdown
# 数据库的ACID
- 原子性、一致性、隔离性、持久性
原子性（Atomicity）
原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
一致性（Consistency）
事务前后数据的完整性必须保持一致。
隔离性（Isolation）
事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。
持久性（Durability）
持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响
```

```markdown
- 脏读：一个事务读到了另一个事务修改过还没提交的数据

- 不可重复读：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。

- 幻读：是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行。

- 不可重复读的重点是修改： 

  同样的条件，你读取过的数据，再次读取出来发现值不一样了

- 幻读的重点在于新增或者删除： 

  同样的条件，第 1 次和第 2 次读出来的记录数不一样
```

## 隔离级别

- Read uncommitted 读未提交，可能出现脏读，不可重复读，幻读。

- Read committed 读提交，可能出现不可重复读，幻读。

- Repeatable read 可重复读，可能出现脏读。

- Serializable 可串行化，同一数据读写都加锁，避免脏读，性能不忍直视。

  InnoDB 默认隔离级别为可重复读级别

## MVCC

MVCC (Multiversion Concurrency Control)，即多版本并发控制技术。

- 在数据库管理系统中，实现对数据库的并发访问，主要是**为了提高数据库的并发性能。**InnoDB 引擎支持 MVCC，因为 myIsam 不支持事务所以也不支持 MVCC。
- 查询一些正在被另一个事务更新的行，并且可以看到它们被更新之前的值，这样在做查询的时候就不用等待另一个事务释放锁。
- 用更好的方式去处理 读-写冲突 ，做到即使有读写冲突时，也能做到 不加锁 ， 非阻塞并发读 ，而这个读指的就是 快照读 , 而非当前读 。当前读实际上是一种加锁的操作，是悲观锁的实现。而MVCC本质是采用乐观锁思想的一种方式。

### 🌞视图

#### 创建视图

```sql
create view view_name as sql语句
```

#### 查看视图

```sql
select * from view_name   [视图在mysql中也会作为一张表的存在]
```

⚠️：视图可以更新，并且会同步更新基表数据！

#### 查看创建视图的语句

```sql
show create view view_name
```

### 🎢存储过程

*存储过程和函数是事先经过编译并存储在数据库的一段sql语句的集合，调用存储过程和函数可以简化工作，减少数据库和服务器之间的传输，提高效率*

- 存储过程没有返回值
- 存储函数有返回值

#### 创建存储过程

```sql
delimiter $ -- 将sql的分隔符改成$

create procedure procedure_name (参数) 
begin
	-- sql语句;
end$

delimiter ;

# 测试
delimiter $
create PROCEDURE procedure_test()
begin 
select 'hello mysql';
END$  #这里不能换行
delimiter ;
call procedure_test();
```

#### 调用存储过程

```sql
call procedure_name();
```

#### 查看存储过程

```sql
select name from mysql.proc where db='db_name';
```

#### 删除存储过程

```sql
drop procedure procedure_name$ 
```

## 存储引擎

```sql
show ENGINES;
# 可以查询到MyISAM和InnoDB的信息
Engine  Support  			comment																							事务 XA  savepoints
--------------------------------------------------------------------------------------------
MyISAM	YES	 				MyISAM storage engine																				NO	NO	NO
InnoDB	DEFAULT			Supports transactions, row-level locking, and foreign keys	YES	YES	YES

# InnoDB支持事务，行级锁，外键
```

- InnoDB的表结构在frm文件中，每个表的数据和索引在ibd文件中
- MyISAM的数据和索引在两个不同的文件中myd和myi中

### B树和B+树的区别

- B树所有的键值对都存在树的节点上

- B+树只有叶子结点会存储值，并且所有的叶子结点之间存在双向指针，形成一个双向链表。

B+树非叶子节点不存储数据，在相同的数据量下，B+树更加矮壮。（数据都存储在叶子节点上，非叶子节点的存储能存储更多的索引，所以整棵树就更加矮壮）。B+树叶子节点之间组成一个链表，方便于遍历查询（遍历操作在MySQL中比较常见）

B+树是多路搜索树，只有叶子节点才存储数据，叶子节点之间链表进行关联。（树矮，易遍历）

在MySQL InnoDB引擎下，每创建一个索引，相当于生成了一颗B+树。

如果该索引是「聚集(聚簇)索引」，那当前B+树的叶子节点存储着「主键和当前行的数据」

如果该索引是「非聚簇索引」，那当前B+树的叶子节点存储着「主键和当前索引列值」

## 回表

*当前索引无法检索出完整的内容，需要通过主键索引进行二次检索。*

当我们使用索引查询数据时，检索出来的数据可能包含其他列，但走的索引树叶子节点只能查到当前列值以及主键ID，所以需要根据主键ID再去查一遍数据，得到SQL 所需的列

订单号ID建了索引，但SQL 是：select orderId,orderName from orderdetail where orderId = 123

在订单ID的索引树的叶子节点只有orderId和Id，还想检索出orderName，所以MySQL 会拿到ID再去查出orderName返回，这种操作就叫回表。

想要避免回表，也可以使用覆盖索引（能使用就使用，因为避免了回表操作）。所谓的覆盖索引，实际上就是你想要查出的列刚好在叶子节点上都存在，比如我建了orderId和orderName联合索引，刚好我需要查询也是orderId和orderName，这些数据都存在索引树的叶子节点上，就不需要回表操作了。

## 最左前缀原则

如有索引 (a,b,c,d)，查询条件 a=1 and b=2 and c>3 and d=4，则会在每个节点依次命中a、b、c，无法命中d

先匹配最左边的，索引只能用于查找key是否存在（相等），遇到范围查询 (>、<、between、like左匹配)等就不能进一步匹配了，后续退化为线性查找。

## 非主键自增有什么问题

需要考虑长度、唯一性和块移动的问题（块内的空间在当前时刻已经存储满了，但新生成的uuid需要插入已满的块内，就需要移动块的数据）

## order by 常见面试题

### 查询语句有 in 多个属性时，SQL 执行是否有排序过程？

假设现在有联合索引 (city,order_num,user_code)，执行以下 SQL 语句：

```sql
select city, order_num, user_code from `order` where city in ('广州') order by order_num limit 1000
```

in 单个条件，是不需要排序的。

但是，in 多个条件时；就会有排序过程，explain 一下，看到最后有 Using filesort 就说明有排序过程。这是为啥呢？

因为 order_num 本来就是组合索引，满足 "city=广州" 只有一个条件时，它是有序的。满足  "city=深圳" 时，它也是有序的。但是两者加到一起就不能保证 order_num 还是有序的了

## 三个范式是什么

第一范式：**字段是最小的的单元不可再分**

- 学生信息组成学生信息表，有年龄、性别、学号等信息组成。这些字段都不可再分，所以它是满足第一范式的

第二范式：满足第一范式,**表中的字段必须完全依赖于全部主键而非部分主键。**

- **其他字段组成的这行记录和主键表示的是同一个东西，而主键是唯一的，它们只需要依赖于主键，也就成了唯一的**
- 学号为1024的同学，姓名为Java3y，年龄是22岁。姓名和年龄字段都依赖着学号主键。

第三范式：满足第二范式，**非主键外的所有字段必须互不依赖**

- **就是数据只在一个地方存储，不重复出现在多张表中，可以认为就是消除传递依赖**
- 比如，我们大学分了很多系（中文系、英语系、计算机系……），这个系别管理表信息有以下字段组成：系编号，系主任，系简介，系架构。那我们能不能在学生信息表添加系编号，系主任，系简介，系架构字段呢？不行的，因为这样就冗余了，非主键外的字段形成了依赖关系(依赖到学生信息表了)！正确的做法是：学生表就只能增加一个系编号字段。

### drop、delete与truncate分别在什么场景之下使用？

我们来对比一下他们的区别：

drop table

- 1)属于DDL
- 2)不可回滚
- 3)不可带where
- 4)表内容和结构删除
- 5)删除速度快

truncate table

- 1)属于DDL
- 2)不可回滚
- 3)不可带where
- 4)表内容删除
- 5)删除速度快

delete from

- 1)属于DML
- 2)可回滚
- 3)可带where
- 4)表结构在，表内容要看where执行的情况
- 5)删除速度慢,需要逐行删除

## 使用场景

- **不再需要一张表的时候，用drop**
- **想删除部分数据行时候，用delete，并且带上where子句**
- **保留表而删除所有数据的时候用truncate**
- **truncate没有日志记录,delete有日志记录**

### varchar和char的区别

**Char是一种固定长度的类型，varchar是一种可变长度的类型**

### MySQL中InnoDB引擎的行锁是通过加在什么上完成

**InnoDB是基于索引来完成行锁**

```mysql
select * from tab_with_index where id = 1 for update;
```

`for update` 可以根据条件来完成行锁锁定,并且 id 是有索引键的列,如果 id 不是索引键那么InnoDB将完成表锁,,并发将无从谈起

## 数据库优化

```sql
show  global status like 'Com_______';# 查看全局使用增删改查的次数
SHOW SESSION STATUS LIKE 'Com_______' # 查看当前回话使用增删改查的次数
```

### 定位低效SQL

- 慢查询日志
- show processlist  查看实时的sql执行情况

## Explain执行计划

- id：表示查询中执行select子句或者是操作表的顺序，id有三种情况
  - id相同表示加载表的顺序是从上到下
  - id不同，id值越大，优先级越高，越先被执行
  - id有相同，也有不同。id值越大，优先级越高，越先被执行，id相同表示加载表的顺序是从上到下
- select_type：查询类型
  - simple：简单查询，不包含子查询或者union
  - primary：查询中若包含任何复杂的子查询，最外层查询标记为此标识
  - subquery：在select或where列表中包含了子查询
  - derived：在from列表中包含的子查询，被标记为derived（衍生）mysql会递归执行这些子查询，把结果放在临时表汇总
  - union：若第二个select出现在union后，则标记为union；若union包含在from子句的子查询中，外层select将被标记为derived
- table_type：访问类型
  - NULL：不访问任何表、索引、直接返回结果
  - system：表只返回一行记录
  - const：根据主键或者唯一索引，只匹配一行记录
  - eq_ref：类似ref，区别在于使用的是唯一索引，使用主键的关联查询， 只有一条记录
  - ref：非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，返回多个
  - range：范围，where之后出现< > between in等操作
  - index：index与all的区别为index类型只是遍历了索引树，通常比all快，all是遍历数据文件
  - all：遍历全表找到匹配的行

- possible Keys 可能用到的索引  key：实际用到的索引  key_len 索引长度（越短越好）
- rows：扫描的行
- extra：额外的一些执行计划信息（using filesort和using tempoary需要优化）
  - using filesort：说明mysql会使用一个外部的排序，而不是按照表内的索引顺序进行读取
  - using tempoary：使用了临时表保存中间结果，MYSQL在对查询结果排序时使用临时表，常见于order by和group by
  - using index：表示相应的select操作使用了覆盖索引，避免访问表的数据行，效率不错


## trace指令

```sql
打开trace，设置格式为json，并设置trace最大能够使用的内存大小，避免解析过程中因为内存过小而不能完整展示
 set optimizer_trace="enabled=on",end_markers_in_json=on;
 set optimizer_tarce_max_mem_size=1000000;
# 你的sql
SELECT * FROM pms_attr where attr_id < 10
# 分析优化器执行计划
SELECT * from information_schema.optimizer_trace
```

## 避免索引失效

- 全值匹配
- 最左前缀原则

  - 注意，是指包含复合索引的最左列且不跳过索引中的列

    - 例如，name,address,status三个字段组成复合索引
- 相当于创建了三个索引
        - name 
  - name+address
        - name+address+status
- name在前都是可以的
      - address status name也是可以的，包含了最左列 优化器进行了优化
- 但是name，status不可以 跳过了中间的索引字段 索引会失效
- 范围查询后面的字段索引会失效
- 字段运算包括字符的剪切拼接等操作会导致索引失效
- 字符串（varchar）不加单引号会导致索引失效

- 尽可能的使用 varchar 代替 char，变长字段存储空间小，可以节省存储空间。
- in走索引，not in不走索引

### 查看索引使用情况

```sql
<!-- 查看索引使用情况 -->
show status like 'Handler_read%';
<!-- 全局 -->
show status like 'Handler_read%';
```

## 优化

### 效率：

**null > system > const > eq_ref > ref > range > index > all**

- **null**: mysql能够在优化阶段分解查询语句。效率最高，例如：在索引列中选取最小值，可以单独查询索引来完成，不需要在执行时访问表。

![img](/Users/zhaoxiang/Desktop/java系统笔记/面试MySQL.assets/0d49a2bfa49d4e14a3edc1b96ccf6a51~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0-20221115201805092.png)

- **system**,**const**：mysql能对查询的某部分进行优化并将其转化成一个常量，用于 `primary key` 或 `unique key`的所有列与常数比较时，所以表最多有一个匹配行，读取一次，速度比较快。`system`是`const`的特例，表里只有一条元素匹配时为`system`

![img](/Users/zhaoxiang/Desktop/java系统笔记/面试MySQL.assets/420990c91d8f4b0892fd4af33c11d322~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.png)

- **eq_ref**: `primary key` 或 `unique key`索引的所有部分被连接使用，最多只会返回一条符合条件的记录。这可能是在`const`之外最好的连接类型了，简单的select查询不会出现这种type。

![img](/Users/zhaoxiang/Desktop/java系统笔记/面试MySQL.assets/7731fd021acd4bb68ef9197ff486d1aa~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.png)

- **ref**: 相比 `eq_ref` ，不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行。

![img](/Users/zhaoxiang/Desktop/java系统笔记/面试MySQL.assets/72941d1a93c64797961cbc465c5946bf~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.png)



![img](/Users/zhaoxiang/Desktop/java系统笔记/面试MySQL.assets/7b6df4dff9ac4305aa9f163453292d91~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.png)

- **range**: 范围扫描通常出现在 in(), between , > , < , >= 等操作中，使用一个索引来检索给定范围的行。

![img](/Users/zhaoxiang/Desktop/java系统笔记/面试MySQL.assets/abf627a69b58413a85ec01d529866855~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.png)

- **index**: 扫描全索引就能拿到结果，一般是扫描某个二级索引，这种扫描不会从索引树根节点开始快速查找，而是直接对二级索引的叶子结点遍历和扫描，速度还是比较慢的，这种查询一般为使用覆盖索引，二级索引一般比较小，所以这种通常比`all`快一些。

![img](/Users/zhaoxiang/Desktop/java系统笔记/面试MySQL.assets/72a06ee6303d41ce9c365bbd5b557b59~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.png)

> index和ref的区别，虽然两者都是走的索引，ref是带有where条件的，会查找出符合条件的数据，而index是查询所有的数据，走B+数的叶子结点的指针，一条条往下面遍历。

- **all**: 即全表扫描，扫描聚簇索引的所有叶子节点，通常情况下这需要增加索引来进行优化了。

> index和all的区别：虽然都是扫描全表，index是扫描 非聚簇索引，需要回表操作，而all是直接扫描的聚簇索引，一条条遍历。

### 优化order by

```markdown
mysql中排序的方式有两种，using fileSort和using index，其中降序排序和升序排序都是使用的fileSort，效率较低。
using index不需要额外排序，效率高  当查询的时候使用的是覆盖索引且字段都是索引，用的才是using index。
# 覆盖索引：
select id,name,address from stu;  
但是如果多个字段 有的是order by desc 有的是asc 就会同时出现using fileSort和using index
并且 排序字段的顺序必须和索引字段顺序一致
所以：优化order by就是
# 排序的字段要么都降序要么都升序
# 排序字段的顺序必须和索引字段顺序一致
```

### 优化group by

默认group by底层也是会排序的，并且会同时使用temporary临时表和using fileSort排序，可以通过使用order by null来取消group  by的排序，想要取消创建临时表，可以创建索引来走索引而不创建临时表

### 优化子查询

可以使用连接查询代替子查询  因为子查询使用的是index，而连接查询使用的是ref，效率比index更高一些

#### 用union all代替union

`Union`：对两个结果集进行`并集操作`，`不包括重复行`，`同时进行默认规则的排序`；
`Union All`：对两个结果集进行`并集操作`，`包括重复行`，`不进行排序`

(select * from user where id=1) 
union 
(select * from user where id=2);
 排重的过程需要遍历、排序和比较，它更耗时，更消耗cpu资源。

#### in和exists

`如果sql语句中包含了in关键字`，则它会`优先执行in里面的子查询语句`，然后再执行in外面的语句。如果in里面的数据量很少，作为条件查询速度更快。
而如果sql语句中包含了`exists关键字`，它`优先执行exists左边的语句（即主查询语句）`。然后把它作为条件，去跟右边的语句匹配。如果匹配上，则可以查询出数据。如果匹配不上，数据就被过滤掉了。
    总结一下：
    `in 适用于左边大表，右边小表。`
    `exists 适用于左边小表，右边大表。`

#### 如果明知道查询结果只有一个,SQL语句中使用LIMIT 1会提高查询效率。

只要找到了对应的一条记录,就不会继续向下扫描了,效率会大大提高。 LIMIT 1适用于查询结果为1条(也可能为0)会导致全表扫描的的SQL语句。 

#### 用连接查询代替子查询

```markdown
# mysql中如果需要从两张以上的表中查询出数据的话，一般有两种实现方式：子查询 和 连接查询。
```

#### 子查询的例子如下：

```sql
select * from order
where user_id in (select id from user where status=1)
```

子查询语句可以通过`in`关键字实现，一个查询语句的条件落在另一个`select`语句的查询结果中。程序先运行在嵌套在最内层的语句，再运行外层的语句。
子查询语句的优点是简单，结构化，如果涉及的表数量不多的话。
但缺点是mysql执行子查询时，`需要创建临时表`，`查询完毕后，需要再删除这些临时表`，有一些额外的`性能消耗`。

#### 这时可以改成连接查询。具体例子如下：

```sql
select o.* from order o
inner join user u on o.user_id = u.id
where u.status = 1
```

#### 男女交换

```sql
update student set sex =(case sex when 'm' then 'f' else 'm' end)
或者
update student set sex = if(sex = 'm','f','m')
```

#### 获取当前日期

```sql
- SELECT NOW()  返回当前系统日期+时间 
- SELECT CURDATE()  返回当前系统日期，不包含时间
- SELECT CURTIME()   返回当前时间，不包含日期
```

### 三大范式

数据库表中的所有字段值都是不可分解的原子值

第二范式在第一范式的基础之上确保数据库表中的每一列都和主键相关

第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。

#### JSON_UNQUOTE和JSON_EXTRACT函数

```sql
#JSON_UNQUOTE 函数作用是 去除json字符串的引号，将值转成string类型
# JSON_EXTRACT 函数作用是 提取json值
select name,
(JSON_EXTRACT(login_info, '$.phone')) phone,
JSON_UNQUOTE(JSON_EXTRACT(login_info, '$.wechat')) wechat
from t_user;

# 简洁写法
select name,
login_info ->> '$.phone' phone,
login_info ->> '$.wechat' wechat
from t_user;

等同于 JSON_UNQUOTE(JSON_EXTRACT(login_info, '$.wechat'))
```
