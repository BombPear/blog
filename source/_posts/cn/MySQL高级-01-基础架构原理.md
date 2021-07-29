---
title: MySQL高级-01-基础架构原理
catalog: true
date: 2020-04-17 02:34:17
subtitle: 
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- Mysql
categories:
- 数据库
---

<!-- <center>
    <h1>
        MySQL高级-01-基础架构原理
    </h1>
</center> -->


# 1、再次认识MySQL

## 1、MySQL简介

- MySQL是一个**关系型数据库*****管理系统***，由瑞典MySQL AB公司开发，目前属于Oracle公司。 
  - SQL、NoSQL
  - Redis
- MySQL是一种关联数据库管理系统，将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。
- Mysql是**开源的**，所以你不需要支付额外的费用。
- Mysql是可以定制的，采用了**GPL协议**，你可以修改源码来开发自己的Mysql系统。 
- Mysql支持大型的数据库。可以处理拥有上千万条记录的大型数据库。
- MySQL使用**标准的SQL数据**语言形式。
  - SQL
- Mysql可以允许于多个系统上，并且支持多种语言。这些编程语言包括C、C++、Python、Java、Perl、PHP、Eiffel、Ruby和Tcl等。
- MySQL支持大型数据库，支持5000万条记录的数据仓库，32位系统表文件最大可支持4GB，64位系统支持最大的表文件为8TB。
  - MySQL（持久化）+ES（检索）

## 2、高手之路

DBA：
DEV：
系统开发：
- 数据库内部结构和原理
- 数据库建模优化:
  - int32  long varchar
- **数据库索引建立**
- **SQL语句优化**
- SQL编程(自定义函数、存储过程、触发器、定时任务)
- mysql服务器的安装配置
- 数据库的性能监控分析与系统优化
- 各种参数常量设定
- 集群配置-如：主从复制
- **分布式架构搭建、垂直切割和水平切割**: 理解思想
- 数据迁移
- 容灾备份和恢复
- shell或python等脚本语言开发
- 对开源数据库进行二次开发
  .....



> **SQL语句优化**

## 3、MySQL部署

> linux版本
- yum：
  - yum源（某个软件的下载地址）？linux自带
  - 配置yum源
  - 自动解决依赖
- rpm
  - 自己解决依赖
  - 麻烦。支持离线
- tar make & make install
  - 下载源码自己编译

### 1、下载

官网下载地址：http://dev.mysql.com/downloads/mysql/

总：https://dev.mysql.com/downloads/

![image-20210725153927780](MySQL高级.assets/image-20210725153927780.png)





### 2、安装与启动

> - rpm适用于所有环境，而yum要搭建本地yum源才可以使用。
> - yum是上层管理工具，自动解决依赖性，而rpm是底层管理工具。

#### 1、rpm方式

https://downloads.mysql.com/archives/community/  下载bundle

```sh
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.34-1.el7.x86_64.rpm-bundle.tar
```

```sh
mkdir mysql & tar -xvf mysql-5.7.34-1.el7.x86_64.rpm-bundle.tar -C mysql/
```

##### 1、安装

> 1、前置环境检查

```sh
#centos7 检查mariadb
rpm -qa|grep mariadb
#移除旧版本
rpm -e --nodeps  mariadb-libs
```

```sh
#检查前置依赖
rpm -qa|grep libaio
rpm -qa|grep net-tools

#如果没有
yum install -y libaio net-tools
```

```sh
#由于mysql安装过程中，会通过mysql用户在/tmp目录下新建tmp_db文件，所以请给/tmp较大的权限

chmod -R 777 /tmp
```



> 2、按照顺序安装

```sh
#在mysql的安装文件目录下执行：（必须按照顺序执行）
rpm -ivh mysql-community-common-5.7.16-1.el7.x86_64.rpm 
rpm -ivh mysql-community-libs-5.7.16-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.16-1.el7.x86_64.rpm 
rpm -ivh mysql-community-server-5.7.16-1.el7.x86_64.rpm
 
#如在检查工作时，没有检查mysql依赖环境再安装mysql-community-server会报错
```



> 查看结果

```sh
mysqladmin --version
 
#查看mysql用户和组
cat /etc/passwd
cat /etc/group
```





#### 2、yum方式

##### 1、安装

https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html

```sh
#准备MySQL5.7 yum源
vim  /etc/yum.repos.d/mysql-community.repo

#内容如下
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

```sh
# 安装完
yum install mysql-community-server
```



##### 2、启动

```sh
#启动服务
systemctl start mysqld
#开机启动【必须记住】【必须记住】【必须记住】
systemctl enable mysqld


# 等于上面两行
systemctl enable mysqld --now
```

> 1、默认完成的事情

- 服务器初始化创建好

- 生成数据目录，并初始化好数据[/var/lib/mysql/]

- 创建超级账户 `'root'@'localhost` . 默认密码保存在 error log file. 用下面命令查询

  - ```sh
    grep 'temporary password' /var/log/mysqld.log
    ```

  - ```sh
    #请先修改默认密码
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'Lfy123456!';
    ```

> 2、MySQL的几个核心位置
>
> ```sh
> ps -ef|grep mysql 也可以看到
> ```

| 目录         | 路径                                      |
| ------------ | ----------------------------------------- |
| **数据目录** | /var/lib/mysql                            |
| **配置文件** | /etc/my.cnf                               |
| **执行程序** | /usr/bin/mysql，/usr/bin/mysqladmin，xxxx |
| 脚本位置     | /usr/share/mysql                          |
| mysql进程    | /var/run/mysqld/mysqld.pid                |
| mysql插件    | /usr/lib64/mysql/plugin                   |



# 2、MySQL核心设置

## 1、my.cnf

### 1、修改字符集

> 查看默认字符集
>
> ```sql
> show variables like 'char%';
> 
> #character_set_database：就是数据库内部存储字符串用的编码
> #character_set_connection ：就是通过socket与mysql通信时的网络编码
> #character_set_client：mysql命令终端和navicat都属于客户端
> ```

```sh
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

最终效果：

![image-20210725171527474](MySQL高级.assets/image-20210725171527474.png)



## 2、用户与权限管理

### 1、用户管理

#### 1、创建用户

```sql
create user zhang3 identified by '123123';
#表示创建名称为zhang3的用户，密码设为123123；
```

#### 2、了解user表

```sql
select * from mysql.user;
select host,user,authentication_string,select_priv,insert_priv,drop_priv from mysql.user;
```

![image-20210725172330968](MySQL高级.assets/image-20210725172330968.png)

- host ： 表示连接类型
  - % 表示用户可以从任何地址连接到服务器
  - localhost 表示用户只能从本地连接到服务器
  - 指定一个ip表示用户只能从此ip连接到服务器
- User:表示用户名
         同一用户通过不同方式链接的权限是不一样的。
- password ： 密码
  - 所有密码串通过 password(明文字符串) 生成的密文字符串。加密算法为MYSQLSHA1 ，不可逆 
  - mysql 5.7 的密码保存到 authentication_string 字段中不再使用password 字段。
  - root 和 张三  密码都是Lfy23456！  ，在底层加密后都是一样的
  
- select_priv , insert_priv等
  - 为该用户所拥有的权限。

```sql
create user "li4"@"%" identified by "Lfy123456!";
create user "lfy"@"localhost" identified by "Lfy123456!";
```



#### 3、修改密码

```sql
#修改当前用户的密码:
set password =password('Lfy123456!');
 
#修改某个用户的密码:
update mysql.user set authentication_string=password('Lfy123456!') where user='wang5';


# 
flush privileges;  #新版本的mysql无需这个操作 5.7.35
```

#### 4、修改用户

```sql
update mysql.user set user='li4' where user='zhang3';

```

#### 5、删除用户

```sql
drop user li4;
```



### 2、权限管理

#### 1、授予权限

```sql
#授权命令： 
grant 权限1,权限2,…权限n on 数据库名称.表名称 to 用户名@用户地址 identified by ‘连接口令’;

grant select,insert,delete,drop on hello.* to wang5@localhost identified by 'Lfy123456!';

grant all privileges on *.* to wang5@localhost identified by 'Lfy123456!';

#该权限如果发现没有该用户，则会直接新建一个用户。比如  
grant select,insert,delete,drop on atguigudb.* to li4@localhost identified by 'Lfy123456!';
 #给li4用户用本地命令行方式下，授予atguigudb这个库下的所有表的插删改查的权限。
 
grant all privileges on *.* to joe@'%'  identified by 'Lfy123456!'; 
#授予通过网络方式登录的的joe用户 ，对所有库所有表的全部权限，密码设为123.


flush privileges;  
#重新加载权限信息，默认修改都在cache中。必须做完才能生效，新版本的mysql无需这个操作
```



#### 2、收回权限

```sql
#查看当前用户权限
show grants;
#查看指定用户权限
show grants for "li4"@"host";
 
#收回权限命令： 
revoke  权限1,权限2,…权限n on 数据库名称.表名称  from  用户名@用户地址 ;
 
REVOKE ALL PRIVILEGES ON *.* FROM wang5@localhost;
#收回全库全表的所有权限
 
REVOKE select,insert,update,delete ON mysql.* FROM joe@localhost;
#收回mysql库下的所有表的插删改查权限
 
#必须用户重新登录后才能生效
```





### 3、远程访问

> 参照  《二.2.2.1》 章节

```sql
grant all privileges on *.* to root@'%'  identified by 'Lfy123123!';
```



# 3、MySQL基本原理

## 1、架构图

![image-20210725201912541](MySQL高级.assets/image-20210725201912541.png)

- 1.连接层
  - 最上层是一些客户端和连接服务，包含本地sock通信和大多数基于客户端/服务端工具实现的类似于tcp/ip的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。
- 2.服务层
  - 2.1  Management Serveices & Utilities： 系统管理和控制工具  
  - 2.2  SQL Interface: SQL接口
    - 接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface
  - 2.3 Parser: 解析器
    - SQL命令传递到解析器的时候会被解析器验证和解析。 
  - 2.4 Optimizer: 查询优化器。
    - SQL语句在查询之前会使用查询优化器对查询进行优化。 
           用一个例子就可以理解： select uid,name from user where  gender= 1;
           优化器来决定先投影还是先过滤。
  - 2.5 Cache和Buffer： 查询缓存。
    - 如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。
            这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等
- 3.引擎层
  - 存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取。后面介绍MyISAM和InnoDB
  - ![image-20210725202357155](MySQL高级.assets/image-20210725202357155.png)
- 4.存储层
    数据存储层，主要是将数据存储在运行于裸设备的文件系统之上，并完成与存储引擎的交互。

## 2、存储引擎

```sql
  #看你的mysql现在已提供什么存储引擎:
  mysql> show engines;
  
  #看你的mysql当前默认的存储引擎:
  mysql> show variables like '%storage_engine%';
```

### 1、各存储引擎

1、InnoDB存储引擎
InnoDB是MySQL的默认事务型引擎，它被设计用来处理大量的短期(short-lived)事务。除非有非常特别的原因需要使用其他的存储引擎，否则应该优先考虑InnoDB引擎。8.0+ 全文索引

2、MyISAM存储引擎
MyISAM提供了大量的特性，包括全文索引、压缩、空间函数(GIS)等，但MyISAM不支持事务和行级锁，有一个毫无疑问的缺陷就是崩溃后无法安全恢复。

```sh
Innodb存储文件有frm、ibd，而Myisam是frm、MYD、MYI

Innodb：frm是表定义文件，ibd是数据文件+索引
Myisam：frm是表定义文件，myd是数据文件，myi是索引文件
```



3、Archive引擎
Archive档案存储引擎只支持INSERT和SELECT操作，在MySQL5.1之前不支持索引。
Archive表适合日志和数据采集类应用。
根据英文的测试结论来看，Archive表比MyISAM表要小大约75%，比支持事务处理的InnoDB表小大约83%。

4、Blackhole引擎
Blackhole引擎没有实现任何存储机制，它会丢弃所有插入的数据，不做任何保存。但服务器会记录Blackhole表的日志，所以可以用于复制数据到备库，或者简单地记录到日志。但这种应用方式会碰到很多问题，因此并不推荐。 

5、CSV引擎 
CSV引擎可以将普通的CSV文件作为MySQL的表来处理，但不支持索引。
CSV引擎可以作为一种数据交换的机制，非常有用。
CSV存储的数据直接可以在操作系统里，用文本编辑器，或者excel读取。

6、Memory引擎
如果需要快速地访问数据，并且这些数据不会被修改，重启以后丢失也没有关系，那么使用Memory表是非常有用。Memory表至少比MyISAM表要快一个数量级。

7、Federated引擎
Federated引擎是访问其他MySQL服务器的一个代理，尽管该引擎看起来提供了一种很好的跨服务器的灵活性，但也经常带来问题，因此默认是禁用的。





### 2、大厂怎么用

  ![image-20210725210724087](MySQL高级.assets/image-20210725210724087.png)

 Percona 为 MySQL 数据库服务器进行了改进，在功能和性能上较 MySQL 有着很显著的提升。该版本提升了在高负载情况下的 InnoDB 的性能、为 DBA 提供一些非常有用的性能诊断工具；另外有更多的参数和命令来控制服务器行为。

该公司新建了一款存储引擎叫xtradb完全可以替代innodb,并且在性能和并发上做得更好,

阿里巴巴大部分mysql数据库其实使用的percona的原型加以修改。
AliSql+AliRedis



> 特别推荐：https://open.oceanbase.com/



## 3、查询流程

### 1、核心流程

![img](MySQL高级.assets/1383365-20190201094534920-1451367430.png)

1. mysql 客户端/服务端通信阶段。
2. 查询缓存阶段。（MySQL出于性能考虑，默认是关闭的）
3. **查询优化处理阶段。（我们的关注点也在此）**
4. 查询执行引擎阶段。
5. 返回客户端阶段。

![image-20210725214641266](MySQL高级-01-基础架构原理.assets/image-20210725214641266.png)

### 2、show profile

> 利用show profile 查看sql的**执行资源消耗信息**

```sh
#开启缓存功能（就可以看到查询时看缓存的过程）
vim /etc/my.cnf
#[mysqld]新增一行：
query_cache_type=1
#重启mysql 
```

```sql
#先开启 
show variables  like '%profiling%';
set profiling=1;
```



```sql
#测试
select * from xxx ;

#显示最近几次查询
show profiles;


 #查看程序的执行步骤
show profile cpu,block io for query 编号 
```





# 4、JOIN查询

## 1、SQL执行

> 手动编写的sql

![image-20210725204142467](MySQL高级.assets/image-20210725204142467.png)





> 优化器优化后的
>
> > 随着Mysql版本的更新换代，其优化器也在不断的升级，优化器会分析不同执行顺序产生的性能消耗不同而动态调整执行顺序。
> >
> > 下面是经常出现的查询顺序：
> >
> > FO JW GHSDOL
> >
> > 佛叫我干活，速度上线
> >
> > FOJWGHSDOL

![image-20210725204248158](MySQL高级.assets/image-20210725204248158.png)

## 2、JOIN图

![image-20210725214753314](MySQL高级-01-基础架构原理.assets/image-20210725214753314.png)





## 3、测试

### 1、建表

```sql
CREATE TABLE `tbl_dept` (
 `id` INT(11) NOT NULL AUTO_INCREMENT,
 `deptName` VARCHAR(30) DEFAULT NULL,
 `locAdd` VARCHAR(40) DEFAULT NULL,
 PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
 
CREATE TABLE `tbl_emp` (
 `id` INT(11) NOT NULL AUTO_INCREMENT,
 `name` VARCHAR(20) DEFAULT NULL,
 `deptId` INT(11) DEFAULT NULL,
 PRIMARY KEY (`id`),
 KEY `fk_dept_id` (`deptId`)
 #CONSTRAINT `fk_dept_id` FOREIGN KEY (`deptId`) REFERENCES `tbl_dept` (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
 
 
 
INSERT INTO tbl_dept(deptName,locAdd) VALUES('RD',11);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('HR',12);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('MK',13);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('MIS',14);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('FD',15);
 
 
INSERT INTO tbl_emp(NAME,deptId) VALUES('z3',1);
INSERT INTO tbl_emp(NAME,deptId) VALUES('z4',1);
INSERT INTO tbl_emp(NAME,deptId) VALUES('z5',1);
 
INSERT INTO tbl_emp(NAME,deptId) VALUES('w5',2);
INSERT INTO tbl_emp(NAME,deptId) VALUES('w6',2);
 
INSERT INTO tbl_emp(NAME,deptId) VALUES('s7',3);
 
INSERT INTO tbl_emp(NAME,deptId) VALUES('s8',4);
 
INSERT INTO tbl_emp(NAME,deptId) VALUES('s9',51);
```

![image-20210725215840897](MySQL高级-01-基础架构原理.assets/image-20210725215840897.png)

### 2、测试

```sql
 
1 A、B两表共有
 select * from tbl_emp a inner join tbl_dept b on a.deptId = b.id;
 # 子查询
select * from tbl_emp a  where a.deptId IN (select id from tbl_dept)
#多表查
select a.*,b.* from tbl_emp a,tbl_dept b where  a.deptId = b.id;
 
2 A、B两表共有+A的独有
 select * from tbl_emp a left join tbl_dept b on a.deptId = b.id;
 
3 A、B两表共有+B的独有
 select * from tbl_emp a right join tbl_dept b on a.deptId = b.id;
 
4 A的独有 
select * from tbl_emp a left join tbl_dept b on a.deptId = b.id where b.id is null; 
 
5 B的独有
 select * from tbl_emp a right join tbl_dept b on a.deptId = b.id where a.deptId is null; #B的独有
 
6 AB全有
#MySQL Full Join的实现 因为MySQL不支持FULL JOIN,下面是替代方法
 #left join + union(可去除重复数据)+ right join
SELECT * FROM tbl_emp A LEFT JOIN tbl_dept B ON A.deptId = B.id
UNION
SELECT * FROM tbl_emp A RIGHT JOIN tbl_dept B ON A.deptId = B.id
 
7 A的独有+B的独有
SELECT * FROM tbl_emp A LEFT JOIN tbl_dept B ON A.deptId = B.id WHERE B.`id` IS NULL
UNION
SELECT * FROM tbl_emp A RIGHT JOIN tbl_dept B ON A.deptId = B.id WHERE A.`deptId` IS NULL;
```



# 5、函数与存储过程

> 如何实现批量导入100w测试数据

## 1、建表

```sql
 
# 新建库
create database bigdata;
use bigdata;
 
 
#1 建表dept
CREATE TABLE dept(  
id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,  
deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,   
dname VARCHAR(20) NOT NULL DEFAULT "",  
loc VARCHAR(13) NOT NULL DEFAULT ""  
) ENGINE=INNODB DEFAULT CHARSET=utf8;  
 
 
#2 建表emp
CREATE TABLE emp  
(  
id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,  
empno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0, /*编号*/  
ename VARCHAR(20) NOT NULL DEFAULT "", /*名字*/  
job VARCHAR(9) NOT NULL DEFAULT "",/*工作*/  
mgr MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,/*上级编号*/  
hiredate DATE NOT NULL,/*入职时间*/  
sal DECIMAL(7,2) NOT NULL,/*薪水*/  
comm DECIMAL(7,2) NOT NULL,/*红利*/  
deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 /*部门编号*/  
)ENGINE=INNODB DEFAULT CHARSET=utf8;
```



> **设置参数log_bin_trust_function_creators【重要】**
>
> 当开启二进制日志后(可以执行show variables like 'log_bin'查看是否开启)，
>
> 如果变量log_bin_trust_function_creators为*OFF*，那么**创建或修改存储函数就会报错**
> “ERROR 1418 (HY000): This function has none of DETERMINISTIC, NO SQL, 
> or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)”这样的错误



![image-20210725221245095](MySQL高级-01-基础架构原理.assets/image-20210725221245095.png)



解决：

```sql
show variables like 'log_bin_trust_function_creators'; 

#临时修改，mysql重启丢失
set global log_bin_trust_function_creators=1;  

#永久修改
vim /etc/my.cnf 

[mysqld]
log_bin_trust_function_creators=1
```

## 2、创建函数

### 1、产生随机字符串函数

```java
public String aaaa(int n){
    int i = 0;
    String chars_str,return_str;
    while(i<n){
        return_str = return_str+"随机单字符"
    }
}
```



```sql
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
 DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';
 DECLARE return_str VARCHAR(255) DEFAULT '';
 DECLARE i INT DEFAULT 0;
 WHILE i < n DO
 SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
 SET i = i + 1;
 END WHILE;
 RETURN return_str;
END $$
DELIMITER ;
```

```sql
drop function rand_string;
```



> 试试调用： 
>
> ```sql
> select rand_string(5);
> ```

### 2、产生随机部门id函数

```sql
 
#用于随机产生部门编号
DELIMITER $$
CREATE FUNCTION rand_num( ) 
RETURNS INT(5)  
BEGIN   
 DECLARE i INT DEFAULT 0;  
 SET i = FLOOR(100+RAND()*10);  
RETURN i;  
END $$
DELIMITER ;
 
#假如要删除
drop function rand_num;
```



## 3、创建存储过程



### 1、emp表中插入数据

```sql
DELIMITER $$
CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))  
BEGIN  
DECLARE i INT DEFAULT 0;   
#set autocommit =0 把autocommit设置成0  
SET autocommit = 0;    
 REPEAT  
 SET i = i + 1;  
 INSERT INTO emp (empno, ename ,job ,mgr ,hiredate ,sal ,comm ,deptno ) VALUES ((START+i) ,rand_string(6),'SALESMAN',0001,CURDATE(),2000,400,rand_num());
 UNTIL i = max_num  
 END REPEAT;  
 COMMIT;  
 END $$
DELIMITER ;


#删除
# drop PROCEDURE insert_emp;
```



### 2、dept表中插入数据

```sql
 
#执行存储过程，往dept表添加随机数据
DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))  
BEGIN  
DECLARE i INT DEFAULT 0;   
 SET autocommit = 0;    
 REPEAT  
 SET i = i + 1;  
 INSERT INTO dept (deptno ,dname,loc ) VALUES ((START+i) ,rand_string(10),rand_string(8));  
 UNTIL i = max_num  
 END REPEAT;  
 COMMIT;  
 END $$ 
DELIMITER ; 
#删除
# 
# drop PROCEDURE insert_dept;

```



### 3、调用

```sql
CALL insert_dept(100,10);
CALL insert_emp(100001,1000000);
```

