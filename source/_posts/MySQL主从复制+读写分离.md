---
title: MySQL主从复制+读写分离
date: 2024-08-13 10:34:26
tags:
- MySQL
categories:
- MySQL
---

# 1、**MySQL主从复制**

MySQL主从复制是一个异步的复制过程，底层是基于Mysql数据库自带的 **二进制日志** 功能。就是一台或多台MySQL数据库（slave，即**从库**）从另一台MySQL数据库（master，即**主库**）进行日志的复制，然后再解析日志并应用到自身，最终实现 **从库** 的数据和 **主库** 的数据保持一致。MySQL主从复制是MySQL数据库自带功能，无需借助第三方工具。

二进制日志（BINLOG）记录了所有的 DDL（数据定义语言）语句和 DML（数据操纵语言）语句，但是不包括数据查询语句。此日志对于灾难时的数据恢复起着极其重要的作用，MySQL的主从复制， 就是通过该binlog实现的。默认MySQL是未开启该日志的。

**MySQL的主从复制原理如下：**

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-08-13_14-44-15.jpg)

**MySQL复制过程分成三步：**

1). MySQL master 将数据变更写入二进制日志( binary log)

2). slave将master的binary log拷贝到它的中继日志（relay log）

3). slave重做中继日志中的事件，将数据变更反映它自己的数据



## 1、**主库配置**

**1、MySQL 默认加载 my.cnf**

```
[mysqld]
log-bin=mysql-bin   #[必须]启用二进制日志
server-id=128      #[必须]服务器唯一ID(唯一即可)
expire_logs_days=7
max_binlog_size=100M
...
```

```java
docker run --privileged -d -p 3306:3306 \
  -v /data/dockerData/mysql/conf/my.cnf:/etc/mysql/conf.d/my.cnf \
  -v /data/dockerData/mysql/logs:/logs \
  -v /data/dockerData/mysql/data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=1234 \
  --name mysql5.7 \
  mysql:5.7 \
  --log-bin=/var/lib/mysql/mysql-bin  # 强制启用binlog（确保生效）
```

2、**验证**

```
# 进入容器
docker exec -it mysql5.7 mysql -uroot -p1234
# 查看binlog是否启用
show variables like 'log_bin';
# 查看主库状态
show master status;
```

**3、创建数据同步的用户并授权**

```
GRANT REPLICATION SLAVE ON *.* to 'xiaoming'@'%' identified by 'Root@251314';
```

注：上面SQL的作用是创建一个用户 xiaoming ，密码为 Root@123456  ，并且给xiaoming用户授予REPLICATION SLAVE权限。常用于建立复制时所需要用到的用户权限，也就是slave必须被master授权具有该权限的用户，才能通过该用户复制。 

**4、登录Mysql数据库，查看master同步状态**

执行下面SQL，记录下结果中**File**和**Position**的值

```
show master status;
```

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/1755072416433.jpg)

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/1755072451506.jpg)

注：上面SQL的作用是查看Master的状态，执行完此SQL后不要再执行任何操作

## 2、**从库配置**

**修改Mysql数据库的配置文件/etc/my.cnf**

```
server-id=201 	#[必须]服务器唯一ID
```

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/1755072533392.jpg)

```
docker run --privileged -d -p 3306:3306 \
  -v /data/dockerData/mysql/conf/my.cnf:/etc/mysql/conf.d/my.cnf \
  -v /data/dockerData/mysql/logs:/logs \
  -v /data/dockerData/mysql/data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=1234 \
  --name mysql5.7 \
  mysql:5.7
```

**登录Mysql数据库，设置主库地址及同步位置**

```
# 进入容器
docker exec -it mysql5.7 mysql -uroot -p1234

stop slave;

change master to master_host='192.168.60.128',master_user='xiaoming',master_password='Root@251314',master_log_file='mysql-bin.000002',master_log_pos=154;

start slave;
```

参数说明：

​	A. master_host : 主库的IP地址

​	B. master_user : 访问主库进行主从复制的用户名(上面在主库创建的)

​	C. master_password : 访问主库进行主从复制的用户名对应的密码

​	D. master_log_file : 从哪个日志文件开始同步(上述查询master状态中展示的有)

​	E. master_log_pos : 从指定日志文件的哪个位置开始同步(上述查询master状态中展示的有)

**查看从数据库的状态**

```
show slave status\G
```

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-08-13_16-12-02.jpg)

MySQL命令行技巧：

​	\G : 在MySQL的sql语句后加上\G，表示将查询结果进行按列打印，可以使每个字段打印到单独的行。即将查到的结构旋转90度变成纵向；

# 2、**读写分离案例**

对日益增加的系统访问量，数据库的吞吐量面临着巨大瓶颈。 对于同一时刻有大量并发读操作和较少写操作类型的应用系统来说，将数据库拆分为**主库**和**从库**，主库负责处理事务性的增删改操作，从库负责处理查询操作，能够有效的避免由数据更新导致的行锁，使得整个系统的查询性能得到极大的改善。

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/1755072836452.jpg)

Sharding-JDBC定位为轻量级Java框架，在Java的JDBC层提供的额外服务。 它使用客户端直连数据库，以jar包形式提供服务，无需额外部署和依赖，可理解为增强版的JDBC驱动，完全兼容JDBC和各种ORM框架。

使用Sharding-JDBC可以在程序中轻松的实现数据库读写分离。

用法：

1、在pom.xml中增加shardingJdbc的maven坐标

```
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>4.0.0-RC1</version>
</dependency>
```

2、在application.yml中增加数据源的配置

```
spring:
  shardingsphere:
    datasource:
      names:
        master,slave
      # 主数据源
      master:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.138.100:3306/rw?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowPublicKeyRetrieval=true
        username: root
        password: root
      # 从数据源
      slave:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.138.101:3306/rw?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowPublicKeyRetrieval=true
        username: root
        password: root
    masterslave:
      # 读写分离配置
      load-balance-algorithm-type: round_robin #轮询
      # 最终的数据源名称
      name: dataSource
      # 主库数据源名称
      master-data-source-name: master
      # 从库数据源名称列表，多个逗号分隔
      slave-data-source-names: slave
    props:
      sql:
        show: true #开启SQL显示，默认false
```

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/1755072960206.jpg)
