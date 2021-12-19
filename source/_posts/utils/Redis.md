---
title: 'Redis基础'
date: 2020-09-20 21:13:45
tags: [学习]
categories: utils
published: true
hideInList: false
feature: 
isTop: false
---

## 软件安装

### 安装docker

CentOS 7 使用yum作为安装程序。安装之前需要检查内核版本

```shell
1.检查内核版本
uname -r 

2.更新内核
yum update

3.安装Docker
yum install docker

4.启动Docker
systemctl start docker
```

<!-- more -->

CentOS 8更改了安装程序，改而使用了dnf 作为安装程序。

```shell
1. 安装Docker
dnf install docker-ce --nobest

2. 允许使用docker
systemctl enable docker

3. 启动Docker
systemctl start docker
```

### 安装redis

使用方法：镜像操作

```shell
1.搜索镜像
docker search redis

2.拉取镜像
docker pull redis

# Using default tag: latest 使用最新的标签
3.查看当前镜像
docker images
#[root@aliC ~]# docker images
#REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
#redis               latest              84c5f6e03bf0        6 hours ago         104MB
#hello-world         latest              bf756fb1ae65        8 months ago        13.3kB

4.下载指定版本镜像
docker pull redis 版本

5. 删除镜像
docker rmi bf756fb1ae65 #镜像的id
```

容器操作：

```shell
1.根据镜像启动容器
docker run -d -p 6379:6379 redis
# -p 是配置端口映射
# -d 是后台运行

3.查看运行中的容器
docker ps

4.停止容器
docker stop 容器id

5.查看所有的容器
docker ps -a

6.重新运行容器
docker start 容器id

7.删除容器
docker rm 容器id
```

阿里云的允许端口访问：要在控制台启用开放6379端口

### 运行redis

```shell
1.进入redis目录
docker exec -it myredis /bin/bash

2.打开控制台
redis-cli

3.退出
exit

4.退出
exit

```

## redis的基本指令

### redis的五大基本数据类型

最常用的设置值和获取值

 ```shell
#设置一对键值对
127.0.0.1:6379> set a a
OK
#获取键所对应的值
127.0.0.1:6379> get a
"a"
#是否存在某个键
127.0.0.1:6379> exists a
(integer) 1
#获取全部的键值对
127.0.0.1:6379> keys *
1) "a"
2) "test"
#获取某键的类型
127.0.0.1:6379> type a
string
#删除某键
127.0.0.1:6379> del a
(integer) 1
#为键值对设置过期时间
127.0.0.1:6379> expire a 10
(integer) 1
#查看还有多少秒过期，-1代表永不过期，-2代表已经过期
127.0.0.1:6379> ttl a
(integer) 8
127.0.0.1:6379> ttl a
(integer) 2
#清空当前库
127.0.0.1:6379> flushdb
OK
#清空所有库
127.0.0.1:6379> flushall
OK
#查看当前数据库的key的数量
127.0.0.1:6379> dbsize
(integer) 1
 ```

### String

String类型是redis中最基本的数据类型，一个redis中的字符串value最多可以是512M

```shell
127.0.0.1:6379> set k1 123
ok
#追加
127.0.0.1:6379> append k1 ji
(integer) 5
127.0.0.1:6379> get k1
"123ji"
#获取字符串长度
127.0.0.1:6379> strlen k1
(integer) 5
#只有在key不存在时设置key的值
127.0.0.1:6379> setnx k2 100
(integer) 1
#对数据值进行加减值的操作
127.0.0.1:6379> incr k2
(integer) 101
127.0.0.1:6379> decr k2
(integer) 100
127.0.0.1:6379> incrby k2 20
(integer) 120
127.0.0.1:6379> decrby k2 50
(integer) 70
#对多个键值对进行操作
mset mget msetnx 
127.0.0.1:6379> set k1 qwertyui
OK
#获取闭区间的字符串的值的范围
127.0.0.1:6379> getrange k1 1 3
"wer"
#从起始位置开始覆盖值
127.0.0.1:6379> setrange k1 1 wqeqe
(integer) 8
#设置键值的同时设置过期时间
setnx
#设置键的同时获取旧的value的值
getset
```

### List

单键多值

是简单的字符串列表，他的底层是一个双向链表，对两端的操作性很高，通过下表操作中间的性能很差。

```shell
127.0.0.1:6379> rpop test
"5"
127.0.0.1:6379> lpop test
"e"
127.0.0.1:6379> rpush test2 x y z
(integer) 3
127.0.0.1:6379> rpoplpush test test2
"4"
127.0.0.1:6379> lrange test 0 3
1) "d"
2) "c"
3) "b"
4) "a"
127.0.0.1:6379> lrange test 0 -1
1) "d"
2) "c"
3) "b"
4) "a"
5) "1"
6) "2"
7) "3"
127.0.0.1:6379> lrem test 1 1
```

### set

```shell
#给集合中添加指定元素
127.0.0.1:6379> sadd testSet a b c s d w qw 1 3 qw ftq
(integer) 10
#查看集合中的元素
127.0.0.1:6379> smembers testSet
 1) "w"
 2) "d"
 3) "s"
 4) "1"
 5) "b"
 6) "qw"
 7) "a"
 8) "c"
 9) "ftq"
10) "3"
#查看集合中是否存在某个元素
127.0.0.1:6379> sismember testSet a
(integer) 1
#删除集合中的某个元素
127.0.0.1:6379> srem testSet a 1
(integer) 2
127.0.0.1:6379> smembers testSet
1) "b"
2) "qw"
3) "w"
4) "s"
5) "c"
6) "d"
7) "ftq"
8) "3"
#从集合中弹出一个元素(随机)
127.0.0.1:6379> spop testSet
"c"
#从集合中取出N个元素(并不删除)
127.0.0.1:6379> srandmember testSet 4
1) "qw"
2) "w"
3) "d"
4) "b"

127.0.0.1:6379> sadd testSet1 a d g e s zx v s
(integer) 7
#取交集
127.0.0.1:6379> sinter testSet testSet1
1) "s"
2) "d"
#取并集
127.0.0.1:6379> sunion testSet testSet1
 1) "qw"
 2) "e"
 3) "g"
 4) "w"
 5) "s"
 6) "zx"
 7) "a"
 8) "d"
 9) "ftq"
10) "3"
11) "v"
12) "b"
#取差集
127.0.0.1:6379> sdiff testSet testSet1
1) "3"
2) "ftq"
3) "qw"
4) "w"
5) "b"
```

### Hash

特别适合存储对象，是一个String类型的field和value的映射表

类似于Java里面的`Map<String, String>`

```shell
#设置一个Hash里的key-value
127.0.0.1:6379> hget set1 user
"1010"
#是否存在某个
127.0.0.1:6379> hexists set1 user
(integer) 1
#获取全部的value
127.0.0.1:6379> hvals set1
1) "1010"
#获取所有的key和value
127.0.0.1:6379> hgetall set1
1) "user"
2) "1010"
#给数字的值添加一个数字
127.0.0.1:6379> hincrby set1 user 20
(integer) 1030
127.0.0.1:6379> hsetnx set1 name 23
(integer) 1
```

### Zset

是一个没有重复元素的字符串集合。里面的每一个成员都关联了一个评分，这个评分被用来从最低分到最高分的方式排序集合中的成员。集合中的成员是唯一的但是评分是可以重复。

元素为键分数为值

```shell
#添加成员的方式是`评分 成员`
127.0.0.1:6379> zadd zset1 100 a 20 b 300 c
(integer) 3
#查询所有的成员
127.0.0.1:6379> zrange zset1 0 -1
1) "b"
2) "a"
3) "c"
127.0.0.1:6379> zadd zset1 400 a
(integer) 0
127.0.0.1:6379> zrange zset1 0 -1
1) "b"
2) "c"
3) "a"
#查询指定分数
127.0.0.1:6379> zrangebyscore zset1 20 300
1) "b"
2) "c"
#增加score，删除元素
zincrby zrem zcount
```

## redis缓存穿透和雪崩

###  缓存穿透

查一个不存在的数据的数据的时候，发现redis中没有命中就会去想持久层数据库查询。发现也没有就会请求失败。多次请求失败导致了缓存穿透问题

指的是对某个一定不存在的数据进行请求，该请求将会穿透缓存到达数据库。

解决方案：

- 对这些不存在的数据缓存一个空数据；
- 对这类请求进行过滤。

#### 布隆过滤器

布隆过滤器是一种数据结构，对所有可能查询的参数以hash的形式进行存储，在控制层先进行校验，不符合就丢弃，避免了对底层存储系统的查询压力。

#### 缓存空对象

没有被命中时

问题：

- 控制在缓存中的存在会浪费缓存的空间
- 即使设置了过期的时间也会有数据不一致的问题

### 缓存击穿

有一个key非常的热点，查的量太大，在缓存过期的一瞬间大量的数据就会击穿缓存。 

解决方案：

#### 设置热点数据永不过期

热点数据不设置过期时间也会造成缓存压力的问题

#### 加互斥锁

每一个key同一时间只能有一个线程能去访问，其余进行等待

### 缓存雪崩

在某个时间，缓存集中失效，Redis宕机。

解决方案：

#### redis高可用

增加redis的数目，异地多活

#### 限流降级

在缓存失效后可以使用加锁或者队列来控制读数据库写缓存的线程数量，例如对一个key同一时间只允许一个线程进行访问，其他线程都在等待。

#### 数据预热

在正式部署之前，将可能的数据先预先访问一遍，是的大量的数据先去加载到缓存中。在即将发生大并发访问前手动触发加载缓存，让缓存失效的时间点尽量均匀。























































































