---
title: Redis集群搭建
date: 2025-03-20 14:36:00
tags:
- redis
categories:
- Linux
---

# Redis集群搭建

## 1、Redis主从集群

### 1.1、集群结构

搭建的主从集群结构如图：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_14-40-01.jpg)

共包含三个节点，一个主节点，两个从节点。

在同一台虚拟机中开启3个redis实例，模拟主从集群，信息如下：

| IP              | PORT | 角色   |
| --------------- | ---- | ------ |
| 192.168.150.101 | 7001 | master |
| 192.168.150.101 | 7002 | slave  |
| 192.168.150.101 | 7003 | slave  |



### 1.2、**准备实例和配置**

要在同一台虚拟机开启3个实例，必须准备三份不同的配置文件和目录，配置文件所在目录也就是工作目录。

1）创建目录

创建三个文件夹，名字分别叫7001、7002、7003：

```
# 进入/tmp目录
cd /tmp
# 创建目录
mkdir 7001 7002 7003
```

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-11-14.jpg)

2）恢复原始配置

修改redis-6.2.4/redis.conf文件，将其中的持久化模式改为默认的RDB模式，AOF保持关闭状态。

```
# 开启RDB
# save ""
save 3600 1
save 300 100
save 60 10000

# 关闭AOF
appendonly no
```

3）拷贝配置文件到每个实例目录

然后将redis-6.2.4/redis.conf文件拷贝到三个目录中（在/tmp目录执行下列命令）：

```
# 方式一：逐个拷贝
cp redis-6.2.4/redis.conf 7001
cp redis-6.2.4/redis.conf 7002
cp redis-6.2.4/redis.conf 7003
# 方式二：管道组合命令，一键拷贝
echo 7001 7002 7003 | xargs -t -n 1 cp redis-6.2.4/redis.conf
```

4）修改每个实例的端口、工作目录

修改每个文件夹内的配置文件，将端口分别修改为7001、7002、7003，将rdb文件保存位置都修改为自己所在目录（在/tmp目录执行下列命令）：

```
sed -i -e 's/6379/7001/g' -e 's/dir .\//dir \/tmp\/7001\//g' 7001/redis.conf
sed -i -e 's/6379/7002/g' -e 's/dir .\//dir \/tmp\/7002\//g' 7002/redis.conf
sed -i -e 's/6379/7003/g' -e 's/dir .\//dir \/tmp\/7003\//g' 7003/redis.conf
```

5）修改每个实例的声明IP

虚拟机本身有多个IP，为了避免将来混乱，需要在redis.conf文件中指定每一个实例的绑定ip信息，格式如下：

```
# redis实例的声明 IP
replica-announce-ip 192.168.150.101
```

每个目录都要改，我们一键完成修改（在/tmp目录执行下列命令）：

```
# 逐一执行
sed -i '1a replica-announce-ip 192.168.150.101' 7001/redis.conf
sed -i '1a replica-announce-ip 192.168.150.101' 7002/redis.conf
sed -i '1a replica-announce-ip 192.168.150.101' 7003/redis.conf

# 或者一键修改
printf '%s\n' 7001 7002 7003 | xargs -I{} -t sed -i '1a replica-announce-ip 192.168.150.101' {}/redis.conf
```



### 1.3、启动

```
# 第1个
redis-server 7001/redis.conf
# 第2个
redis-server 7002/redis.conf
# 第3个
redis-server 7003/redis.conf
或
./redis-6.2.4/src/redis-server ./7001/redis.conf
./redis-6.2.4/src/redis-server ./7002/redis.conf
./redis-6.2.4/src/redis-server ./7003/redis.conf
```

启动后：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-12-26.jpg)

```
# 如果要一键停止，可以运行下面命令：
printf '%s\n' 7001 7002 7003 | xargs -I{} -t redis-cli -p {} shutdown
```



### 1.4、开启主从关系

现在三个实例还没有任何关系，要配置主从可以使用replicaof 或者slaveof（5.0以前）命令。

有临时和永久两种模式：

-  修改配置文件（永久生效） 

slaveof  

-  使用redis-cli客户端连接到redis服务，执行slaveof命令（重启后失效）： 

```
slaveof <masterip> <masterport>
```

**注意**：在5.0以后新增命令replicaof，与salveof效果一致。

这里我们为了演示方便，使用方式二。

通过redis-cli命令连接7002，执行下面命令：

```
# 连接 7002
redis-cli -p 7002
# 执行slaveof
slaveof 124.183.178.142 7001
```

通过redis-cli命令连接7003，执行下面命令：

```
# 连接 7003
redis-cli -p 7003
# 执行slaveof
slaveof 192.168.150.101 7001
```

然后连接 7001节点，查看集群状态：

```
# 连接 7001
redis-cli -p 7001
# 查看状态
info replication
```

结果：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-22-47.jpg)

### 1.5、测试

执行下列操作以测试：

-  利用redis-cli连接7001，执行

```
set num 123 
```

-  利用redis-cli连接7002，执行

```
get num，再执行
set num 666 
```

-  利用redis-cli连接7003，执行

```
get num，再执行
set num 888 
```

可以发现，只有在7001这个master节点上可以执行写操作，7002和7003这两个slave节点只能执行读操作。



## 2、**搭建哨兵集群**

### 2.1、**集群结构**

这里我们搭建一个三节点形成的Sentinel集群，来监管之前的Redis主从集群。如图：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-36-19.jpg)

三个sentinel实例信息如下：

| 节点 | IP              | PORT  |
| ---- | --------------- | ----- |
| s1   | 192.168.150.101 | 27001 |
| s2   | 192.168.150.101 | 27002 |
| s3   | 192.168.150.101 | 27003 |

### **3.2.准备实例和配置**

要在同一台虚拟机开启3个实例，必须准备三份不同的配置文件和目录，配置文件所在目录也就是工作目录。

我们创建三个文件夹，名字分别叫s1、s2、s3：

```java
# 进入/tmp目录
cd /tmp
# 创建目录
mkdir s1 s2 s3
```

然后我们在s1目录创建一个sentinel.conf文件，添加下面的内容：

```java
port 27001
sentinel announce-ip 192.168.150.101
sentinel monitor mymaster 192.168.150.101 7001 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
dir "/tmp/s1"
```

解读：

- port 27001  ：是当前sentinel实例的端口

- sentinel monitor mymaster 192.168.150.101 7001 2   ：指定主节点信息 

- mymaster  ：主节点名称，自定义，任意写

- 192.168.150.101 7001  ：主节点的ip和端口

- 2  ：选举master时的quorum值

然后将s1/sentinel.conf文件拷贝到s2、s3两个目录中（在/tmp目录执行下列命令）：

```java
# 方式一：逐个拷贝
cp s1/sentinel.conf s2
cp s1/sentinel.conf s3
# 方式二：管道组合命令，一键拷贝
echo s2 s3 | xargs -t -n 1 cp s1/sentinel.conf
```

修改s2、s3两个文件夹内的配置文件，将端口分别修改为27002、27003：

```java
sed -i -e 's/27001/27002/g' -e 's/s1/s2/g' s2/sentinel.conf
sed -i -e 's/27001/27003/g' -e 's/s1/s3/g' s3/sentinel.conf
```

### 2.3、启动

为了方便查看日志，我们打开3个ssh窗口，分别启动3个redis实例，启动命令：

```java
# 第1个
redis-sentinel s1/sentinel.conf
./redis-6.2.4/src/redis-sentinel ./s1/sentinel.conf
# 第2个
redis-sentinel s2/sentinel.conf
./redis-6.2.4/src/redis-sentinel ./s2/sentinel.conf
# 第3个
redis-sentinel s3/sentinel.conf
./redis-6.2.4/src/redis-sentinel ./s3/sentinel.conf
```

启动后：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-39-20.jpg)

### 2.3、测试

尝试让master节点7001宕机，查看sentinel日志：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-42-35.jpg)

查看7003的日志：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-43-20.jpg)

查看7002的日志：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-43-38.jpg)

## 3、搭建分片集群

### 3.1、**集群结构**

分片集群需要的节点数量较多，这里我们搭建一个最小的分片集群，包含3个master节点，每个master包含一个slave节点，结构如下：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-45-49.jpg)

这里我们会在同一台虚拟机中开启6个redis实例，模拟分片集群，信息如下：

| IP              | PORT | 角色   |
| --------------- | ---- | ------ |
| 192.168.150.101 | 7001 | master |
| 192.168.150.101 | 7002 | master |
| 192.168.150.101 | 7003 | master |
| 192.168.150.101 | 8001 | slave  |
| 192.168.150.101 | 8002 | slave  |
| 192.168.150.101 | 8003 | slave  |

### 3.2、**准备实例和配置**

删除之前的7001、7002、7003这几个目录，重新创建出7001、7002、7003、8001、8002、8003目录：

```java
# 进入/tmp目录
cd /tmp
# 删除旧的，避免配置干扰
rm -rf 7001 7002 7003
# 创建目录
mkdir 7001 7002 7003 8001 8002 8003
```

在/tmp下准备一个新的redis.conf文件，内容如下：

```java
port 6379
# 开启集群功能
cluster-enabled yes
# 集群的配置文件名称，不需要我们创建，由redis自己维护
cluster-config-file /tmp/6379/nodes.conf
# 节点心跳失败的超时时间
cluster-node-timeout 5000
# 持久化文件存放目录
dir /tmp/6379
# 绑定地址
bind 0.0.0.0
# 让redis后台运行,守护进程
daemonize yes
# 注册的实例ip
replica-announce-ip 192.168.150.101
# 保护模式
protected-mode no
# 数据库数量
databases 1
# 日志
logfile /tmp/6379/run.log
```

将这个文件拷贝到每个目录下：

```java
# 进入/tmp目录
cd /tmp
# 执行拷贝
echo 7001 7002 7003 8001 8002 8003 | xargs -t -n 1 cp redis.conf
```

修改每个目录下的redis.conf，将其中的6379修改为与所在目录一致：

```java
# 进入/tmp目录
cd /tmp
# 修改配置文件
printf '%s\n' 7001 7002 7003 8001 8002 8003 | xargs -I{} -t sed -i 's/6379/{}/g' {}/redis.conf
```

### 3.3、**启动**

因为已经配置了后台启动模式，所以可以直接启动服务：

```java
# 进入/tmp目录
cd /tmp
# 一键启动所有服务
printf '%s\n' 7001 7002 7003 8001 8002 8003 | xargs -I{} -t redis-server {}/redis.conf
```

通过ps查看状态：

```java
ps -ef | grep redis
```

发现服务都已经正常启动：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-48-58.jpg)

如果要关闭所有进程，可以执行命令：

```java
ps -ef | grep redis | awk '{print $2}' | xargs kill
```

或者（推荐这种方式）：

```java
printf '%s\n' 7001 7002 7003 8001 8002 8003 | xargs -I{} -t redis-cli -p {} shutdown
```

### 3.4、**创建集群**

虽然服务启动了，但是目前每个服务之间都是独立的，没有任何关联。

我们需要执行命令来创建集群，在Redis5.0之前创建集群比较麻烦，5.0之后集群管理命令都集成到了redis-cli中。

1）Redis5.0之前

Redis5.0之前集群命令都是用redis安装包下的src/redis-trib.rb来实现的。因为redis-trib.rb是有ruby语言编写的所以需要安装ruby环境。

```java
# 安装依赖
yum -y install zlib ruby rubygems
gem install redis
```

然后通过命令来管理集群：

```java
# 进入redis的src目录
cd /tmp/redis-6.2.4/src
# 创建集群
./redis-trib.rb create --replicas 1 192.168.150.101:7001 192.168.150.101:7002 192.168.150.101:7003 192.168.150.101:8001 192.168.150.101:8002 192.168.150.101:8003
```

2）Redis5.0以后

我们使用的是Redis6.2.4版本，集群管理以及集成到了redis-cli中，格式如下：

```java
redis-cli --cluster create --cluster-replicas 1 192.168.150.101:7001 192.168.150.101:7002 192.168.150.101:7003 192.168.150.101:8001 192.168.150.101:8002 192.168.150.101:8003
```

命令说明：

- redis-cli --cluster

或者

./redis-trib.rb：代表集群操作命令

- create

：代表是创建集群

- --replicas 1

或者

--cluster-replicas 1 ：指定集群中每个master的副本个数为1，此时

节点总数 ÷ (replicas + 1) 得到的就是master的数量。因此节点列表中的前n个就是master，其它节点都是slave节点，随机分配到不同master

运行后的样子：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-51-42.jpg)

这里输入yes，则集群开始创建：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-51-53.jpg)

通过命令可以查看集群状态：

```java
redis-cli -p 7001 cluster nodes
```

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-52-08.jpg)

### 3.5、测试

尝试连接7001节点，存储一个数据：

```java
# 连接
redis-cli -p 7001
# 存储数据
set num 123
# 读取数据
get num
# 再次存储
set a 1
```

结果悲剧了：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-56-06.jpg)

集群操作时，需要给redis-cli加上-c参数才可以：

```java
redis-cli -c -p 7001
```

这次可以了：

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-03-20_19-56-23.jpg)
