---
title: MySQL高级-02-索引分析与优化
catalog: true
date: 2020-04-17 02:34:17
subtitle: 
sticky: 999
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- Mysql
categories:
- 数据库
---

# 1、索引简介

## 1、是什么

- MySQL官方对索引的定义为：**索引（Index）是帮助MySQL高效获取数据的数据结构。**
- 索引的本质：索引是数据结构。

> ***数据库原理***：索引是采用 **B 树结构存储**的，所以**对应的索引项并不会被删除**，经过一段时间的增删改操作后，数据库中就会出现大量的存储碎片，这和磁盘碎片、内存碎片产生原理是类似的，这些存储碎片不仅占用了存储空间，而且降低了数据库运行的速度。如果发现索引中存在过多的存储碎片的话就要进行“碎片整理”了，最方便的**“碎片整理” 手段就是重建索引**， 重建索引会将先前创建的索引删除然后重新创建索引，**主流数据库管理系统都提供了重建索引的功能**，比如 REINDEX、REBUILD 等，如果使用的数据库管理系统没有提供重建索引的功能，可以首先用DROP INDEX语句删除索引，然后用ALTER TABLE 语句重新创建索引。
>
> MySQL：索引可能因为删除，或者页分裂等原因，导致数据页有空洞，重建索引的过程会创建一个新的索引，把数据按顺序插入，这样页面的利用率高，也就是索引更紧凑、更省空间。
>
> **MySQL底层使用B+树**
>
> 线性结构：（找一个东西，最坏情况需要遍历完）
>
> - 数组、链表
>
> 非线性结构：（找一东西，最坏的情况也比较节省时间）
>
> - 图、树

- BST：Binary Search Tree(二叉搜索树)

- B+树与B树中的B代表平衡（balance）而不是二叉（Binary）
- B+是B树的改进版

- 没有B-树；B-Tree



2^3=8

2^x=n

设 有n个元素，2的x次方是n，求x的值

如果 n=64 则x=6

log2  64=6

二叉树的最优复杂度：

![image-20210727100848130](MySQL高级-02-索引分析与优化.assets/image-20210727100848130.png)





## 2、加快检索

在数据之外，**数据库系统还维护着满足特定查找算法的数据结构**，这些数据结构以某种方式引用（指向）数据，
这样就可以在这些数据结构上实现高级查找算法。**这种数据结构，就是索引。**

下图就是一种可能的索引方式示例：

![image-20210726222132292](MySQL高级-02-索引分析与优化.assets/image-20210726222132292.png)

- 左边是数据表，一共有两列七条记录，最左边的是数据记录的物理地址
- 为了加快Col2的查找，可以维护一个右边所示的二叉查找树，每个节点分别包含索引键值和一个指向对应数据记录物理地址的指针，这样就可以运用二叉查找在一定的复杂度内获取到相应数据，从而快速的检索出符合条件的记录。



## 3、几种索引

### 1、单列索引

> 即一个索引只包含单个列，一个表可以有多个单列索引



### 2、唯一索引

> 索引列的值必须唯一，但允许有空值



### 3、复合索引

> 即一个索引包含多个列





> - 常规索引，也叫普通索引(index或key)，它可以常规地提高查询效率。一张数据表中可以有多个常规索引。常规索引是使用最普遍的索引类型，如果没有明确指明索引的类型，我们所说的索引都是指常规索引。
>
> - 主键索引(Primary Key)，也简称主键。它可以提高查询效率，并提供唯一性约束。一张表中只能有一个主键。被标志为自动增长的字段一定是主键，但主键不一定是自动增长。一般把主键定义在无意义的字段上(如：编号)，主键的数据类型最好是数值。
>
> - 唯一索引（Unique Key），可以提高查询效率，并提供唯一性约束。一张表中可以有多个唯一索引。
>
> - 全文索引(Full Text)，可以提高全文搜索的查询效率，一般使用Sphinx替代。但Sphinx不支持中文检索，Coreseek是支持中文的全文检索引擎，也称作具有中文分词功能的Sphinx。实际项目中，我们用到的是Coreseek。
>
> - 外键索引(Foreign Key)，简称外键，它可以提高查询效率，外键会自动和对应的其他表的主键关联。外键的主要作用是保证记录的一致性和完整性。



### 4、索引操作

#### 1、创建

```sql
CREATE  [UNIQUE ] INDEX indexName ON mytable(columnname(length)); 

CREATE INDEX idx_deptno ON dept(deptno)
#或者

ALTER TABLE 表名 ADD  [UNIQUE ]  INDEX [indexName] ON (columnname(length)) 
```

#### 2、删除

```sql
DROP INDEX [indexName] ON mytable; 
```



#### 3、查看

```sql
SHOW INDEX FROM table_name;
```



#### 4、alter使用

```sql
 
有3种方式来添加数据表的索引：
ALTER TABLE tbl_name ADD PRIMARY KEY (column_list): 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
 
ALTER TABLE tbl_name ADD UNIQUE index_name (column_list): 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。
 
ALTER TABLE tbl_name ADD INDEX index_name (column_list): 添加普通索引，索引值可出现多次。
```



## 4、MySQL索引结构

### 1、B-Tree：B树

> B树(Balance Tree多路平衡查找树)
>
> B+树(加强版多路平衡查找树)
>
> MySQL索引使用B+树，也就是B树进阶版



### 2、为什么是B+树

> 思考：有16个数，在里面找一个数怎么最快？

#### 1、全部遍历

> 知道就行

#### 2、Hash

hash算法。把任意东西转成一串数字

aaa = bbb

111 = ccc



加速查找速度的数据结构，常见的有两类：
(1)哈希，例如HashMap，查询/插入/修改/删除的平均时间复杂度都是O(1)；
(2)树，例如平衡二叉搜索树，查询/插入/修改/删除的平均时间复杂度都是O(log2N)；

可以看到，不管是读请求，还是写请求，哈希类型的索引，都要比树型的索引更快一些，那为什么，索引结构要设计成树型呢？



> 想想范围/排序等其它SQL条件：
> 哈希型的索引，时间复杂度会退化为O(n)而树型的“有序”特性，依然能够保持O(log2(n)) 的高效率。



**备注：InnoDB并不支持哈希索引。**



> 时间复杂度
>
> Ο(1)＜Ο(log2N)＜Ο(n)＜Ο(nlog2N)＜Ο(n^2)＜Ο(n^3)＜ Ο(n^k) ＜Ο(2^n)\****

![image-20210726223610823](MySQL高级-02-索引分析与优化.assets/image-20210726223610823.png)



#### 3、BST：二叉树

> ***基础概念：（适用任何树）***
>
> 节点：每个点
>
> 根节点：树根起始
>
> 叶子节点：最后一层，没有子节点的节点
>
> 非叶子节点（分支节点）：有子节点的节点
>
> 节点的度：一个节点最大包含多少个子节点。叶子节点度为0
>
> 树的度：节点度的最大值
>
> 深度：节点从根往下数，所在层次，最深的节点所在层次也就是根的深度
>
> 高度：节点往下数最长路径深度，根的高度=树的高度





> 规则：小左大右

![image-20210726223337159](MySQL高级-02-索引分析与优化.assets/image-20210726223337159.png)



https://www.cs.usfca.edu/~galles/visualization/BST.html

> 思考？BST问题？
>
> 极限就是一个链表

```java
//编写树的遍历，增加
public class Node{
    private Node left;
    private Integer val;
    private Node right
    
}

Node root = new Node(2);

addNode(Node node,Node root){
    
    Node temp = searchNode(root);
    if(node.val > temp.val){
        temp.right(node);
    }else{
        temp.left(node);
    }
}


Node searchParentNode(Node node,Node root){
    //找办法
   
}
```



#### 4、平衡二叉树(AVL)

https://www.cs.usfca.edu/~galles/visualization/AVLtree.html

> 核心： 
>
> 1、右旋
>
> ![图片](MySQL高级-02-索引分析与优化.assets/641)
>
> 2、左旋
>
> ![图片](MySQL高级-02-索引分析与优化.assets/640)

问题：高度太高





#### 5、B树

https://www.cs.usfca.edu/~galles/visualization/BTree.html



##### 1、底层原理

- 数据库索引是存储在磁盘上的，如果数据很大，必然导致索引的大小也会很大，超过几个G（好比新华字典字数多必然导致目录厚）-
- 当我们利用索引查询时候，是不可能将全部几个G的索引都加载进内存的，我们能做的只能是：
  - 逐一加载每一个磁盘页，因为磁盘页对应着索引树的节点。

![image-20210726231310267](MySQL高级-02-索引分析与优化.assets/image-20210726231310267.png)



##### 2、磁盘页&磁盘块

```sql
SHOW GLOBAL STATUS LIKE 'Innodb_page_size';

#Innodb_page_size
#INNODB page size (DEFAULT 16KB). 
#Many VALUES are counted IN pages; the page size enables them TO be easily converted TO bytes
```

##### 3、块页关系

- 系统从磁盘读取数据到内存时是以磁盘块（block）为基本单位的，位于同一个磁盘块中的数据会被一次性读取出来，而不是需要什么取什么。
- InnoDB存储引擎中有页（Page）的概念，页是其磁盘管理的最小单位。 
- 系统一个磁盘块的存储空间往往没有这么大，因此InnoDB每次申请磁盘空间时都会是若干地址连续磁盘块来达到页的大小16KB。
  InnoDB在把磁盘数据读入到磁盘时会以页为基本单位，在查询数据时如果一个页中的每条数据都能有助于定位数据记录的位置，
  这将会减少磁盘I/O次数，提高查询效率。



##### 4、B树检索原理

![image-20210726231602535](MySQL高级-02-索引分析与优化.assets/image-20210726231602535.png)



每个节点占用一个盘块的磁盘空间，一个节点上有两个升序排序的关键字和三个指向子树根节点的指针，指针存储的是子节点所在磁盘块的地址。

模拟查找关键字29的过程：
根据根节点找到磁盘块1，读入内存。【磁盘I/O操作第1次】
比较关键字29在区间（17,35），找到磁盘块1的指针P2。
根据P2指针找到磁盘块3，读入内存。【磁盘I/O操作第2次】
比较关键字29在区间（26,30），找到磁盘块3的指针P2。
根据P2指针找到磁盘块8，读入内存。【磁盘I/O操作第3次】
在磁盘块8中的关键字列表中找到关键字29。

分析上面过程，发现需要3次磁盘I/O操作，和3次内存查找操作。由于内存中的关键字是一个有序表结构，可以利用二分法查找提高效率。而3次磁盘I/O操作是影响整个BTree查找效率的决定因素。BTree相对于AVLTree缩减了节点个数，使每次磁盘I/O取到内存的数据都发挥了作用，从而提高了查询效率。

> 结论：B树比平衡二叉树减少了一次IO操作





#### 6、B+树

##### 1、结构图

![image-20210726231720859](MySQL高级-02-索引分析与优化.assets/image-20210726231720859.png)

![image-20210726231754285](MySQL高级-02-索引分析与优化.assets/image-20210726231754285.png)



##### 2、检索原理

> 由于B+树的非叶子节点只存储键值信息，假设每个磁盘块能存储4个键值及指针信息，则变成B+树后其结构如下图所示：

![image-20210726231849184](MySQL高级-02-索引分析与优化.assets/image-20210726231849184.png) 

![image-20210726231929176](MySQL高级-02-索引分析与优化.assets/image-20210726231929176.png)



##### 3、结论

- 1、B+树是在B树基础上的一种优化使其更适合实现外存储索引结构，InnoDB存储引擎就是用B+树实现其索引结构。
- 2、B+树相对于B树有几点不同
  
  - 非叶子节点只存储键值信息。
  - 所有叶子节点之间都有一个链指针。
  - 数据记录都存放在叶子节点中。



##### 4、时间复杂度

![image-20210726232110521](MySQL高级-02-索引分析与优化.assets/image-20210726232110521.png)





![image-20210726232131148](MySQL高级-02-索引分析与优化.assets/image-20210726232131148.png)



> 递归？



## 5、索引优势劣势

优势：

- 加快检索
- 通过索引列对数据进行排序，降低数据排序的成本，降低了CPU的消耗

劣势：

- 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。
  因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，
  都会调整因为更新所带来的键值变化后的索引信息
- 索引也是要占用额外空间







# 2、EXPLAIN-索引分析

## 1、是什么

查看查询计划

> 使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈



http://dev.mysql.com/doc/refman/5.7/en/explain-output.html

![image-20210727121714567](MySQL高级-02-索引分析与优化.assets/image-20210727121714567.png)



## 2、怎么玩

Explain + SQL语句： 在优化器部分执行

![image-20210727121812140](MySQL高级-02-索引分析与优化.assets/image-20210727121812140.png)



```sql
EXPLAIN SELECT * FROM emp;
```

![image-20210727121908889](MySQL高级-02-索引分析与优化.assets/image-20210727121908889.png)





## 3、能看出什么

- 表的读取顺序
- 数据读取操作的操作类型
- 哪些索引可以使用
- 哪些索引被实际使用
- 表之间的引用
- 每张表有多少行被优化器查询





## 4、各字段解释

```sql
# 测试脚本
 
 CREATE TABLE t1(id INT(10) AUTO_INCREMENT,content  VARCHAR(100) NULL ,  PRIMARY KEY (id));
 CREATE TABLE t2(id INT(10) AUTO_INCREMENT,content  VARCHAR(100) NULL ,  PRIMARY KEY (id));
 CREATE TABLE t3(id INT(10) AUTO_INCREMENT,content  VARCHAR(100) NULL ,  PRIMARY KEY (id));
 CREATE TABLE t4(id INT(10) AUTO_INCREMENT,content  VARCHAR(100) NULL ,  PRIMARY KEY (id));
 
 
 INSERT INTO t1(content) VALUES(CONCAT('t1_',FLOOR(1+RAND()*1000)));
 
  INSERT INTO t2(content) VALUES(CONCAT('t2_',FLOOR(1+RAND()*1000)));
  
  INSERT INTO t3(content) VALUES(CONCAT('t3_',FLOOR(1+RAND()*1000)));
    
  INSERT INTO t4(content) VALUES(CONCAT('t4_',FLOOR(1+RAND()*1000)));
```

### 1、id

> select查询的序列号,包含一组数字，表示查询中执行select子句或操作表的顺序
>
> 三种情况

#### 1、id相同

> id相同，执行顺序由上至下

![image-20210727122359732](MySQL高级-02-索引分析与优化.assets/image-20210727122359732.png)

```sql
EXPLAIN SELECT t2.id FROM t2,t1,t3 WHERE t1.id = t2.id AND t2.id=t3.id
```



#### 2、id不同

> id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

![image-20210727122505624](MySQL高级-02-索引分析与优化.assets/image-20210727122505624.png)



 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

```sql
EXPLAIN 
SELECT t2.* FROM 
t2 WHERE id = (
  SELECT id FROM t1 WHERE id = (
	   SELECT id FROM t3 WHERE t3.id=1
	)
)
```





#### 3、同又不同

> - id如果相同，可以认为是一组，从上往下顺序执行；
> - 在所有组中，id值越大，优先级越高，越先执行
>
> 衍生 = DERIVED
>
> 这个抓图，是5.5的

![image-20210727122554348](MySQL高级-02-索引分析与优化.assets/image-20210727122554348.png)

```sql
EXPLAIN select t2.* from (select t3.id from t3 ) s1,t2 where s1.id=t2.id


EXPLAIN select t2.* from (select t3.id from t3 where t3.content = 't3_169') s1,t2 where s1.id=t2.id
```



> 新版抓图

![image-20210727153608626](MySQL高级-02-索引分析与优化.assets/image-20210727153608626.png)



重点总结：

- **id号每个号码，表示一趟独立的查询。**
- **一个sql 的查询趟数越少越好。**



### 2、select_type

**查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询**



> 常用的

![image-20210727122749200](MySQL高级-02-索引分析与优化.assets/image-20210727122749200.png)

> 官网的

![image-20210727122820116](MySQL高级-02-索引分析与优化.assets/image-20210727122820116.png)



> 我们的

![image-20210727122949264](MySQL高级-02-索引分析与优化.assets/image-20210727122949264.png)



#### 1、SIMPLE

>  简单的 select 查询,查询中不包含子查询或者UNION



#### 2、PRIMARY

> 查询中若包含任何复杂的子部分，最外层查询则被标记为

```sql
EXPLAIN SELECT t2.* FROM t2 WHERE t2.content = (SELECT t3.content FROM t3 WHERE t3.content='t3_169')

PRIMARY 不叫主键

这是一个主查询，一定有子查询(说明会有复杂表连接等操作)
```



#### 3、SUBQUERY

> 在SELECT或WHERE列表中包含了子查询

![image-20210727123108018](MySQL高级-02-索引分析与优化.assets/image-20210727123108018.png)



#### 4、DEPENDENT SUBQUERY

![image-20210727123155077](MySQL高级-02-索引分析与优化.assets/image-20210727123155077.png)



#### 5、UNCACHEABLE SUBQUREY

> 不能缓存的子查询

![image-20210727123221376](MySQL高级-02-索引分析与优化.assets/image-20210727123221376.png)

![image-20210727123320950](MySQL高级-02-索引分析与优化.assets/image-20210727123320950.png)



```sql
explain select * from t1 where id = ( select id from t2 where id = (select FLOOR(1+RAND()*100)) );
```



#### 6、DERIVED

>  在FROM列表中包含的子查询被标记为DERIVED(衍生)，MySQL会递归执行这些子查询, 把结果放在临时表里。

![image-20210727122554348](MySQL高级-02-索引分析与优化.assets/image-20210727122554348.png)



#### 7、UNION

> 若第二个SELECT出现在UNION之后，则被标记为UNION；
> 若UNION包含在FROM子句的子查询中,外层SELECT将被标记为：DERIVED

```sql
explain
select * from(
(select emp_no from employees el limit 10)
union all
(select emp_no from employees e2 limit 10)
union all
(select emp_no from employees e3 limit 10)
) tb;

#以上查询的执行计划如下图所示。3个联合（union）的select 查询中，只有第一个（el数据表）不是union，其余两个的select_type均为union。union的第一个查询设置为代表整个union结果的select_type类型。此外要将3个子查询的结果用union all进行联合，并创建临时表进行使用，所以union all的第一个查询的select_type为DERIVED。
```





#### 8、UNION RESULT

>从UNION表获取结果的SELECT
>
>```html
>union result为包含union结果的数据表。
>MariaDB中，union all或union（DISTINCT）查询会将所有union结果创建为临时表。
>执行计划中，该临时表所在行为select_type为union result。
>由于union result在实际查询中不是单位查询，所以没有单独的id值。
>```

```sql
explain
select emp_no from salaries where salary>100000
union all
select emp_no from dept_emp where from_date>'2001-01-01'
```







### 3、table

> 显示这一行的数据是关于哪张表的



### *4、type

> type显示的是**访问类型**，是较为重要的一个指标，结果值从最好到最坏依次是： 

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > **ALL** 



> 我们需要记住的：最好到最坏

**system>const>eq_ref>ref>range>index>ALL**



> SYSTEM > **CONST** > **EQ_REF** > REF > **RANGE** > INDEX > ALL
>
> - 百万级数据，需要优化到**RANGE**级别；最好达到**eq_ref**





#### 1、system

> 表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计

**system：**

- 表中只有一行记录，等于系统表
- 是const类型的特例，平时不会出现

![image-20210727130910308](MySQL高级-02-索引分析与优化.assets/image-20210727130910308.png)

```sql
explain select * from (select * from t1 where id = 1) d1
```



#### 2、const

> 表示通过索引一次就找到了,const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快
> 如将主键置于where列表中，MySQL就能将该查询转换为一个常量;
>
> xxxx   where id=xxx   主键精确查就是常量

**CONST：**

- 将主键或唯一索引的所有部分与常量值比较，如将**主键置于where列表中**，就能转换成一个常量const
- 表最多有一个匹配行，由于只有一行，则可以被优化器的其余部分视为常量；
- CONST很快，因为只读一次；用于***将主键或唯一索引的所有部分与常量值比较***
- CONST的意义：只有一个值能够对应（PRIMARY或UNIQUE），该行的值可以被优化器的其余部分视为常量

![image-20210727131311242](MySQL高级-02-索引分析与优化.assets/image-20210727131311242.png)



#### 3、eq_ref

> 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
>
> 连表+主键匹配  eq_ref

**eq_ref：**

- 用于多表查询，唯一性索引（主键/UNIQUE）
- 常见于主键和唯一索引扫描（即单列索引）
- 与const的区别：JOIN的最好的type；比较值是常量或使用此表之前读取的列的表达式



案例一：

![image-20210727131711781](MySQL高级-02-索引分析与优化.assets/image-20210727131711781.png)

```sql
explain select * from t1,t2 where t1.id=t2.id
```



案例二：

> t1,t2,t3三张表，特例情况，每张表有且仅有一条记录。
>  t1里面的记录，t2/t3只有一条与之对应

![image-20210727131644133](MySQL高级-02-索引分析与优化.assets/image-20210727131644133.png)

```sql
select * from t1,t2,t3 where t1.id = t2.id and t2.id = t3.id

eq_ref: 等于一个位置
```

![image-20210727162906728](MySQL高级-02-索引分析与优化.assets/image-20210727162906728.png)



```sql

explain select t1.id from t1,t2,t3 where t1.id = t2.id and t2.id = t3.id
```

![image-20210727163137772](MySQL高级-02-索引分析与优化.assets/image-20210727163137772.png)

案例三：

> 多条记录

![image-20210727131810960](MySQL高级-02-索引分析与优化.assets/image-20210727131810960.png)



![image-20210727163241336](MySQL高级-02-索引分析与优化.assets/image-20210727163241336.png)

#### 4、ref

> 非唯一性索引扫描，返回匹配某个单独值的所有行. 本质上也是一种索引访问，
>
> 它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体

**ref：**

- 非唯一性索引扫描/多列索引，非主键/UNIQUE，可能是外键或其他非唯一性索引，可能返回多行





案例一：

![image-20210727131941197](MySQL高级-02-索引分析与优化.assets/image-20210727131941197.png)

> 虽然是复合索引，但是用 一个索引查（就可能产生多条记录）





案例二：

![image-20210727132026278](MySQL高级-02-索引分析与优化.assets/image-20210727132026278.png)



> 用非唯一索引去查，也是这个效果



#### 5、range

**range：**

- 只检索给定范围的行，用一个索引来选择行
- 一般是WHERE语句中出现了Between、<、>、IN等查询
- **比全表扫描好**，只需要开始于索引的一点，结束于另一点



案例一：

![image-20210727132133557](MySQL高级-02-索引分析与优化.assets/image-20210727132133557.png)



案例二：

![image-20210727132146253](MySQL高级-02-索引分析与优化.assets/image-20210727132146253.png)

#### 6、index

> Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。
> （也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）

**index：**

- 使用索引（覆盖索引/利用索引排序分组），但是没有使用索引过滤
- 全索引扫描：SELECT id FROM t1;

![image-20210727132311033](MySQL高级-02-索引分析与优化.assets/image-20210727132311033.png)









#### 7、all

> Full Table Scan，将遍历全表以找到匹配的行

**all：**

- 全表扫描

![image-20210727132509810](MySQL高级-02-索引分析与优化.assets/image-20210727132509810.png)









### 5、possible_keys

> 显示可能应用在这张表中的索引，一个或多个。
> 查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用



### *6、key

- 实际使用的索引。如果为NULL，则没有使用索引
- 查询中若使用了覆盖索引，则该索引和查询的select字段重叠

![image-20210727132707766](MySQL高级-02-索引分析与优化.assets/image-20210727132707766.png)







### 7、key_len

> 表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好



### 8、ref

> 显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值

案例一：

![image-20210727132839999](MySQL高级-02-索引分析与优化.assets/image-20210727132839999.png)

由key_len可知t1表的idx_col1_col2被充分使用，col1匹配t2表的col1，col2匹配了一个常量，即 'ac'





案例二：

查询中与其它表关联的字段，外键关系建立索引

![image-20210727132901474](MySQL高级-02-索引分析与优化.assets/image-20210727132901474.png)







### 9、rows

> 根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数



![image-20210727133014021](MySQL高级-02-索引分析与优化.assets/image-20210727133014021.png)





> 建立外键，进行测试

![image-20210727133147482](MySQL高级-02-索引分析与优化.assets/image-20210727133147482.png)





### *10、Extra

> 包含不适合在其他列中显示但十分重要的额外信息

#### 1、Using Where

> 表示优化器需要通过索引**回表查询数据；**
>
> Using where
>
> 





#### 2、Using filesort 

![image-20210727133300341](MySQL高级-02-索引分析与优化.assets/image-20210727133300341.png)

```sql
explain select id,deptno from emp where empno=100020 order by ename;
#Using index condition; Using filesort

explain select id,deptno from emp where empno=100020 AND deptno=101 order by ename;
#Using index condition; Using where; Using filesort



# 用到 Using filesort，说明是无索引字段的排序
explain select id from emp where empno=100020 AND sal = 2000 order by id;
#Using where; Using index
```



 **查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度**



![image-20210727133320976](MySQL高级-02-索引分析与优化.assets/image-20210727133320976.png)









#### 3、Using temporary 

> 使了用临时表保存中间结果,MySQL在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by。

![image-20210727133433446](MySQL高级-02-索引分析与优化.assets/image-20210727133433446.png)

```sql
EXPLAIN SELECT deptno FROM emp WHERE emp.id > 100000 GROUP BY deptno
#Using where; Using temporary; Using filesort

#给deptno 建单列索引后：
EXPLAIN SELECT deptno FROM emp WHERE id > 100000 GROUP BY deptno
#Using where; Using index
```





#### 4、Using index

> 表示相应的select操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错！
> 如果同时出现using where，表明索引被用来执行索引键值的查找;
> 如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。
>
> 表示直接访问索引就足够获取到所需要的数据，不需要通过**索引回表**；



覆盖索引：

- 覆盖索引（Covering Index）,也说为索引覆盖。
- 理解: 就是select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件,换句话说查询列要被所建的索引覆盖。
- 如 select id from emp；

 

注意：

- 如果要使用覆盖索引，一定要注意select列表中只取出需要的列，不可select *，
- 因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降。 



#### 5、Using index condition 

> 会先条件过滤索引，过滤完索引后找到所有符合索引条件的数据行，随后用 WHERE 子句中的其他条件去过滤这些数据行；

# 3、索引优化

> 避免索引失效的情况、
>
> 先讨论单表

```sql
SHOW INDEX FROM emp;
# 小技巧：完全释放索引空洞

# alter table emp engine=InnoDB;
```



## 1、建表

```sql
# 测试sql
CREATE TABLE staffs (
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR (24) NOT NULL DEFAULT '' COMMENT '姓名',
  age INT NOT NULL DEFAULT 0 COMMENT '年龄',
  pos VARCHAR (20) NOT NULL DEFAULT '' COMMENT '职位',
  add_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间'
) CHARSET utf8 COMMENT '员工记录表' ;
 
 
INSERT INTO staffs(NAME,age,pos,add_time) VALUES('z3',22,'manager',NOW());
INSERT INTO staffs(NAME,age,pos,add_time) VALUES('July',23,'dev',NOW());
INSERT INTO staffs(NAME,age,pos,add_time) VALUES('2000',24,'dev',NOW());
INSERT INTO staffs(NAME,age,pos,add_time) VALUES('2000',25,'dev',NOW());
 
SELECT * FROM staffs;
 
ALTER TABLE staffs ADD INDEX idx_staffs_nameAgePos(name, age, pos);
```

> 请注意，创建复合索引应当包含少数几个列，并且这些列经常在select查询里使用。在复合索引里包含太多的列不仅不会给带来太多好处。而且由于使用相当多的内存来存储复合索引的列的值，其后果是内存溢出和性能降低。





## 2、索引失效案例

### 1、全值匹配我最爱

```sql
EXPLAIN SELECT * FROM staffs WHERE NAME = 'July';
EXPLAIN SELECT * FROM staffs WHERE NAME = 'July' AND age = 25;
EXPLAIN SELECT * FROM staffs WHERE NAME = 'July' AND age = 25 AND pos = 'dev';
```

![image-20210727134308400](MySQL高级-02-索引分析与优化.assets/image-20210727134308400.png)



### 2、最佳左前缀法则

> 如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。

```sql
EXPLAIN SELECT * FROM staffs WHERE age = 25 AND pos = 'dev';
 
EXPLAIN SELECT * FROM staffs WHERE pos = 'dev';
```

![image-20210727134409188](MySQL高级-02-索引分析与优化.assets/image-20210727134409188.png)



### 3、索引列不另操作

>  不在索引列上做任何操作（计算、函数、(自动or手动)类型转换），会导致索引失效而转向全表扫描

```sql
EXPLAIN SELECT * FROM staffs WHERE left(NAME,4) = 'July';
#或者
EXPLAIN SELECT * FROM staffs WHERE id + 5 =10;
```



![image-20210727134502212](MySQL高级-02-索引分析与优化.assets/image-20210727134502212.png)





### 4、存储引擎不能使用索引中范围条件右边的列

> 范围右边索引失效

```sql
EXPLAIN SELECT * FROM staffs WHERE NAME = 'z4';

EXPLAIN SELECT * FROM staffs WHERE NAME = 'z4' and age=22;

EXPLAIN SELECT * FROM staffs WHERE NAME = 'z4' and age=22 and pos='manager';

EXPLAIN SELECT * FROM staffs WHERE NAME = 'z4' and age>22 and pos='manager';
```



![image-20210727134535128](MySQL高级-02-索引分析与优化.assets/image-20210727134535128.png)



### 5、尽量使用覆盖索引(只访问索引的查询(索引列和查询列一致))，减少select *

```sql
EXPLAIN SELECT * FROM staffs WHERE NAME = 'z3' and age=22 and pos='manager';

EXPLAIN SELECT name,age,pos FROM staffs WHERE NAME = 'z3' and age=22 and pos='manager';

########

EXPLAIN SELECT * FROM staffs WHERE NAME = 'z3' and age>22 and pos='manager';

EXPLAIN SELECT name,age,pos FROM staffs WHERE NAME = 'z3' and age>22 and pos='manager';

#### Using Index
```



![image-20210727134759905](MySQL高级-02-索引分析与优化.assets/image-20210727134759905.png)



### 6、mysql 在使用不等于(!= 或者<>)的时候有时候无法使用索引会导致全表扫描

```sql
EXPLAIN SELECT * FROM staffs WHERE NAME = 'July';

EXPLAIN SELECT * FROM staffs WHERE NAME != 'July';

```



![image-20210727134829637](MySQL高级-02-索引分析与优化.assets/image-20210727134829637.png)



### 7、注意null/not null对索引的可能影响

#### 1、情况一

```sql
#建议新建一个库，试试测试脚本
CREATE TABLE staffs (
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR (24) NOT NULL DEFAULT '' COMMENT '姓名',
  age INT NOT NULL DEFAULT 0 COMMENT '年龄',
  pos VARCHAR (20) NOT NULL DEFAULT '' COMMENT '职位',
  add_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间'
) CHARSET utf8 COMMENT '员工记录表' ;
INSERT INTO staffs(NAME,age,pos,add_time) VALUES('z3',22,'manager',NOW());
ALTER TABLE staffs ADD INDEX idx_staffs_nameAgePos(NAME, age, pos);

ALTER TABLE staffs ADD INDEX idx_staffs_name(NAME);


EXPLAIN SELECT * FROM staffs WHERE NAME IS NULL; 


EXPLAIN SELECT * FROM staffs WHERE NAME IS NOT NULL;
```

![image-20210727134920573](MySQL高级-02-索引分析与优化.assets/image-20210727134920573.png)

#### 2、情况二

```sql
#测试脚本如下：
 
CREATE TABLE staffs2 (
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR (24),
  age INT NOT NULL DEFAULT 0 COMMENT '年龄',
  pos VARCHAR (20) NOT NULL DEFAULT '' COMMENT '职位',
  add_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间'
) CHARSET utf8 COMMENT '员工记录表' ;
INSERT INTO staffs2(NAME,age,pos,add_time) VALUES('z3',22,'manager',NOW());
ALTER TABLE staffs2 ADD INDEX idx_staffs2_nameAgePos(NAME, age, pos);
ALTER TABLE staffs2 ADD INDEX idx_staffs2_name(NAME);
 
EXPLAIN SELECT * FROM staffs2 WHERE NAME IS NULL;
EXPLAIN SELECT * FROM staffs2 WHERE NAME IS NOT NULL;
```

![image-20210727135010852](MySQL高级-02-索引分析与优化.assets/image-20210727135010852.png)



#### 3、官网总结

![image-20210727135027108](MySQL高级-02-索引分析与优化.assets/image-20210727135027108.png)



### 8、like以通配符开头('%abc...')mysql索引失效会变成全表扫描的操作

![image-20210727135053789](MySQL高级-02-索引分析与优化.assets/image-20210727135053789.png)

```sql
```







### 9、字符串不加单引号索引失效

![image-20210727135121077](MySQL高级-02-索引分析与优化.assets/image-20210727135121077.png)

```sql
#5.7 的新版效果
#SQL传的索引列的值如果要MySQL进行类型转换，再查，MySQL会优先使用 精确的单列索引
#SQL传的索引列的值如果可以直接使用，去查，MySQL会优先使用 复合索引
EXPLAIN SELECT NAME FROM staffs WHERE NAME = 2000
```

![image-20210728105746276](MySQL高级-02-索引分析与优化.assets/image-20210728105746276.png)



### 10、少用or,用它来连接时会索引失效

![image-20210727135141105](MySQL高级-02-索引分析与优化.assets/image-20210727135141105.png)

```sql
#5.7
# or的不至于全表了. 优化器会处理为范围查询
```



```sql
# 5.7的小优化


# 全值匹配都还行，只要大哥一直有
# 中间兄弟不能断，索引列上少计算
# 范围之后全失效，LIKE百分不最左
# 覆盖索引不写*,  不等空值有影响
# OR和IN已被优化。 VAR引号可以丢，
# 但是注意这四句 ：索引类型若转换
# 两种条件请判断，转换要找单索引
# 不转优先复合列。 SQL优化小诀窍
```







## 3、周阳老师的口诀

先来个小测验

![image-20210727135217796](MySQL高级-02-索引分析与优化.assets/image-20210727135217796.png)

![image-20210727135320387](MySQL高级-02-索引分析与优化.assets/image-20210727135320387.png)



```sql
#大口诀
全职匹配我最爱，最左前缀要遵守；
带头大哥不能死，中间兄弟不能断；
索引列上少计算，范围之后全失效；
LIKE百分写最右，覆盖索引不写*；
不等空值还有OR，索引影响要注意；
VAR引号不可丢， SQL优化有诀窍。
```



# 4、复杂情况

## 1、关联查询

### 1、建表

```sql
 
CREATE TABLE IF NOT EXISTS `class` (
`id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`id`)
);
CREATE TABLE IF NOT EXISTS `book` (
`bookid` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`bookid`)
);
 
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
 
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));


## 
```



### 2、实战

```sql
 # 1、下面开始explain分析
EXPLAIN SELECT SQL_NO_CACHE * FROM class LEFT JOIN book ON class.card = book.card;
#结论：type 有All
#### 1、如果连表查询，连接条件无索引，导致笛卡尔积，全表扫描  40*40=1600

#### 2、左连接给左表连接列建索引，无意义, 驱动表都需要全表扫描。
#### 3、左连接给右表连接列建索引, 意义大，被驱动表要查询



 
# 2、添加索引优化。请问下面两个添哪个
ALTER TABLE book ADD INDEX idx_card(card)
ALTER TABLE class ADD INDEX idx_card(card)

# 驱动表无法避免全表扫描


DROP INDEX idx_card ON class;
DROP INDEX idx_card ON book;



# 4、换成inner join,智能的，只要连表的字段有索引，就会最后被用到


# MySQL自动决定谁作为驱动表（有索引的一般是被驱动表）


```







### 3、建议

- 1、保证被驱动表的join字段已经被索引
- 2、left join 时，选择小表作为驱动表，大表作为被驱动表。
- 3、inner join 时，mysql会自动帮你把小结果集的表选为驱动表。
- 4、子查询尽量不要放在被驱动表，有可能使用不到索引。
- 5、能直接写多表关联就不写子查询





## 2、排序优化

### 1、基础优化

```sql
#以下是否能使用到索引，能否去掉using filesort
explain  select SQL_NO_CACHE * from emp order by empno,deptno; 

#解决上面问题？用索引？
create index idx_empno_deptno_ename on emp (empno,deptno,ename)

#再试试？

explain  select SQL_NO_CACHE * from emp order by empno,deptno limit 10; 
 
 #如果需要排序，就先过滤，缩小范围
 
 #无过滤 不索引  【没有过滤条件也就是where，直接排序用不上索引，可以加limit使他用上（分页也是过滤）】 
 
 
 
 
###################################################################################

 
explain  select SQL_NO_CACHE * from emp where empno=100021 order by deptno;
 
explain  select SQL_NO_CACHE * from emp where empno=100021 order by   deptno,ename; 
explain  select SQL_NO_CACHE * from emp where empno=100021 order by  ename,deptno;
##上面反映，顺序错，必排序（filesort）


#这条很重要，mysql在不影响结果的情况下会优化，也就是等于 order by empno,deptno
explain  select SQL_NO_CACHE * from emp where empno=100021 order by  deptno,empno;


explain  select SQL_NO_CACHE * from emp where empno=100021 order by  deptno,mgr;
# 混入非索引字段，必排序 （filesort）

 
explain  select SQL_NO_CACHE * from emp where deptno=100021 order by empno;
# 带头大哥没有，无索引

#顺序错，必排序 Using filesort
 
 
################################################################################
 
 
 
explain select * from emp where empno=100021 order by  deptno desc, ename desc ;
 
  
explain select * from emp where empno=100021 order by  deptno asc, ename desc ;
 
#方向反 必排序
```

- 以下必排序代表： Using filesort
  - 不过滤，必排序
  - 顺序错，必排序
  - 方向反，必排序

> ORDER BY子句，尽量使用 **Using Index**方式排序,避免使用**Using fileSort**方式排序



### 2、索引选择

> 实验准备

```sql
# 先删除emp表所有索引，保留 id字段即可

#查询 deptno为 105 的，且员工编号小于101000的用户，按 ename 排序
SELECT SQL_NO_CACHE * FROM emp WHERE deptno =105 AND empno <101000 ORDER BY ename ;
 

#如何优化上面？
#开始优化：
#思路：  尽量让where的过滤条件和排序使用上索引
#         但是一共两个字段(deptno,empno)上有过滤条件，一个字段(ename)有索引 
## 1、第一种？我们建一个三个字段的组合索引可否？
 CREATE INDEX idx_deptno_empno_ename ON emp(deptno,empno,ename);
 
 
 
## 2、第二种？只关注Using filesort???
deptno_empno  #
deptno_ename  #

### 你可以永远相信MySQL
SELECT SQL_NO_CACHE * FROM emp WHERE deptno =105 AND empno <101000 ORDER BY ename ;
最终: 给 deptno 和 empno 建立复合索引

rows：判定金标准
```









## 3、另外

- group by 使用索引的原则几乎跟order by一致 
- 唯一区别是groupby 即使没有过滤条件用到索引，也可以直接使用索引。（也就是不用where也可以大胆的group by 索引列）



- 别忘了覆盖索引规则。也就是不要写select *











