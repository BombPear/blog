---
title: MySQL高级-03-集群原理与搭建
catalog: true
date: 2020-04-17 02:34:17
subtitle: 
lang: cn
sticky: 100
header-img: /img/header_img/lml_bg.jpg
tags:
- Mysql
categories:
- 数据库
---

<!-- <center>
    <h1>
        MySQL高级-03-集群原理与搭建
    </h1>
    <h2 style="color:red">
        主从复制
    </h2>
</center> -->

## 1、基本原理

- Master - RW（Read Write）
  - bin-log
- Slave  - RO （Read Only）

- slave会从master读取binlog来进行数据同步

![image-20210727224000298](MySQL高级-03-集群原理与搭建.assets/image-20210727224000298.png)

MySQL复制过程分成三步：

- 1、master将改变记录到二进制日志（binary log）。这些记录过程叫做二进制日志事件，binary log events；
- 2、slave将master的binary log events拷贝到它的中继日志（relay log）；
- 3、slave重做中继日志中的事件，将改变应用到自己的数据库中。 MySQL复制是异步的且串行化的



## 2、基本原则

- 每个slave只有一个master
- 每个节点只能有一个唯一的服务器ID
- 每个master可以有多个salve



## 3、最大问题

延迟

> 网络： 
>
> - 无盘化
>
> **磁盘**：  100mb/s
>
> CPU：
>
> MEMEROY：





## 4、其他集群方式

![img](MySQL高级-03-集群原理与搭建.assets/wps1.jpg)

以上可以作为企业中常用的数据库解决方案；

双主：

- 主备模式： 备机要同步主机数据
- 备机可以不提供服务
- VIP：IP飘移
  - 客户端负载均衡
    - 优点：服务端压力小
    - 缺点：无法实时感知对方的集群状态
  - 服务端负载均衡
    - 优点：集群状态更新实时
    - 缺点：消耗一定资源





### 1、MySQL-MMM

 ![img](MySQL高级-03-集群原理与搭建.assets/wps2.jpg)

MySQL-MMM是Master-Master Replication Manager for MySQL（mysql主主复制管理器）的简称，是Google的开源项目（Perl脚本）。MMM基于MySQL Replication做的扩展架构，主要用来监控mysql主主复制并做失败转移。其原理是将真实数据库节点的IP（RIP）映射为虚拟IP（VIP）集。mysql-mmm的监管端会提供多个虚拟IP（VIP），包括一个可写VIP，多个可读VIP，通过监管的管理，这些IP会绑定在可用mysql之上，当某一台mysql宕机时，监管会将VIP迁移至其他mysql。在整个监管过程中，需要在mysql中添加相关授权用户，以便让mysql可以支持监理机的维护。授权的用户包括一个mmm_monitor用户和一个mmm_agent用户，如果想使用mmm的备份工具则还要添加一个mmm_tools用户。



### 2、MHA

MHA（Master High Availability）目前在MySQL高可用方面是一个相对成熟的解决方案,由**日本**DeNA公司youshimaton（现就职于Facebook公司）开发，是一套优秀的作为MySQL高可用性环境下故障切换和主从提升的高可用软件。在MySQL故障切换过程中，MHA能做到在0~30秒之内自动完成数据库的故障切换操作（以2019年的眼光来说太慢了），并且在进行故障切换的过程中，MHA能在最大程度上保证数据的一致性，以达到真正意义上的高可用。



### 3、InnoDB Cluster

InnoDB Cluster支持自动Failover、强一致性、读写分离、读库高可用、读请求负载均衡，横向扩展的特性，是比较完备的一套方案。但是部署起来复杂，想要解决router单点问题好需要新增组件，**如没有其他更好的方案可考虑该方案。** InnoDB Cluster主要由MySQL Shell、MySQL Router和MySQL服务器集群组成，三者协同工作，共同为MySQL提供完整的高可用性解决方案。MySQL Shell 对管理人员提供管理接口，可以很方便的对集群进行配置和管理,MySQL Router 可以根据部署的集群状况自动的初始化，是客户端连接实例。如果有节点down机，集群会自动更新配置。集群包含单点写入和多点写入两种模式。在单主模式下，如果主节点down掉，从节点自动替换上来，MySQL Router会自动探测，并将客户端连接到新节点。

![image-20210727225601591](MySQL高级-03-集群原理与搭建.assets/image-20210727225601591.png)





## 5、搭建主从

> 以Docker方式为例

> Docker启动容器的数据部分一定挂载出来

### 1、创建Master

```sh
docker run -p 3307:3306 --name mysql-master \
-v /mydata/mysql/master/log:/var/log/mysql \
-v /mydata/mysql/master/data:/var/lib/mysql \
-v /mydata/mysql/master/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root \
--restart=always \
-d mysql:5.7 
#参数说明
	-p 3307:3306：将容器的3306端口映射到主机的3307端口
	-v /mydata/mysql/master/conf:/etc/mysql：将配置文件夹挂在到主机
	-v /mydata/mysql/master/log:/var/log/mysql：将日志文件夹挂载到主机
	-v /mydata/mysql/master/data:/var/lib/mysql/：将配置文件夹挂载到主机
	-e MYSQL_ROOT_PASSWORD=root：初始化root用户的密码
```



```sh
#修改master基本配置
vim /mydata/mysql/master/conf/my.cnf

[client]
default-character-set=utf8
 
[mysql]
default-character-set=utf8
 
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
#注意：skip-name-resolve一定要加，不然连接mysql会超级慢

```

```sh
#添加master主从复制部分配置
server_id=1
log-bin=mysql-bin
read-only=0

#可以同步的库
binlog-do-db=hello
binlog-do-db=world

#不用同步的库
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
```

重启



### 2、创建Slaver

```sh
docker run -p 3308:3306 --name mysql-slaver-01 \
-v /mydata/mysql/slaver/log:/var/log/mysql \
-v /mydata/mysql/slaver/data:/var/lib/mysql \
-v /mydata/mysql/slaver/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root \
--restart=always \
-d mysql:5.7 
```

```sh
#修改slave基本配置
vim /mydata/mysql/slaver/conf/my.cnf

[client]
default-character-set=utf8
 
[mysql]
default-character-set=utf8
 
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

```sh
#添加master主从复制部分配置
server_id=2
log-bin=mysql-bin
read-only=1

binlog-do-db=hello
binlog-do-db=world


replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
```



### 3、master授权slaver链接

```sql
#1、进入master容器
docker exec -it mysql /bin/bash

#2、进入mysql内部 （mysql –uroot -p）
	#1）、授权root可以远程访问（ 主从无关，为了方便我们远程连接mysql）
grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
flush privileges;
    #2）、添加用来同步的用户
       GRANT REPLICATION SLAVE ON *.* to 'backup'@'%' identified by '123456';
       
       
#3、查看master状态
   show master status\G;



```



### 4、slaver同步master数据

```sql
#1、进入slaver容器
docker exec -it mysql-slaver-01 /bin/bash
#2、进入mysql内部（mysql –uroot -p）
   #1）、授权root可以远程访问（ 主从无关，为了方便我们远程连接mysql）
grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
flush privileges;

   #2）、设置主库连接
change master to master_host='172.17.0.2',master_user='backup',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=0,master_port=3306;
  #3）、启动从库同步
start slave;
  #4）、查看从库状态
     show slave status\G;
```

![image-20210727230328840](MySQL高级-03-集群原理与搭建.assets/image-20210727230328840.png)

至此主从配置完成； 

总结：

​	1）、主从数据库在自己配置文件中声明需要同步哪个数据库，忽略哪个数据库等信息。并且server-id不能一样

​	2）、主库授权某个账号密码来同步自己的数据

​	3）、从库使用这个账号密码连接主库来同步数据