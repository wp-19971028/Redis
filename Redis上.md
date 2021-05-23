# NoSQL数据库的发展历史简介

## web系统变迁历史

- web1. 0 时代简介
  - lweb 1.0是以编辑为特征，网站提供给用户的内容是网站编辑进行编辑处理后提供的，用户阅读网站提供的内容
  - 这个过程是网站到用户的**单向行为**
  -  web1.0时代的代表站点为新浪，搜狐，网易三大门户

| **搜狐** | ![img](./assetswps1.jpg) |
| -------- | ------------------------------------------------------------ |
| **网易** | ![img](./assets\wps2.jpg)                       |

性能要求：

- 基本上就是一些简单的静态页面渲染，不会涉及太多复杂业务逻辑，功能简单单一，基本上服务器性能不会有太大压力

总结: 就是用户访问网站只能看人家编辑好的信息,没有交互.

- web 2.0时代简介
  - 更注重用户的**交互**作用，用户既是网站内容的消费者（浏览者），也是网站内容的制造者
  -  加强了网站与用户之间的**互动**，网站内容基于用户提供（微博、天涯社区、自媒体）
  - 网站的诸多功能也由用户参与建设，实现了网站与用户双向的交流与参与
  - 用户在web 2.0网站系统内拥有自己的数据，并完全基于WEB，所有功能都能通过浏览器完成。
- 性能要求：
  - 随着Web 2.0时代到来，用户访问量大幅提升，同时产生大量用户数据。加上后来移动设备普及，所有的互联网平台都面临了巨大的性能挑战。**数据库服务器压力越来越大**
  - 如何应对数据库的压力成为的关键。不管任何应用，数据始终是核心。传统关系型数据库根本无法承载较高的并发，此时人们就开始**用Redis当成缓存，来缓解数据库的压力**

## 什么是NoSQL

- NoSQL最常见的解释是“non-relational”， 很多人也说它是“Not Only SQL” 
- NoSQL仅仅是一个概念，泛指非关系型的数据库
- 区别于关系数据库，它们不保证关系数据的ACID特性
-  NoSQL是一项全新的数据库革命性运动，提倡运用非关系型的数据存储，相对于铺天盖地的关系型数据库运用，这一概念无疑是一种全新的思维的注入

## NoSQL的特点

- 应用场景
  - 高并发读写
  - 海量数据读写
  - 高可扩展性
  - 速度快
- 不适用场景
  - 需要事务支持
  - 基于SQL的结构化查询存储,处理复杂关系
  - 需要即席查询(用户自定义查询条件的查询)

## NoSQL数据库介绍

- memcache
  - 很早出现的数据库
  - 数据都在内存里面,一般不持久化
  - 支持简单的key - value 模式
  - 一般是做为缓存数据库辅助持久化数据库
- Redis的介绍
  - 几乎覆盖了Memcached的绝大部分功能
  - 数据都存储在内存中,支持持久化,主要用做备份恢复
  - 除了支持简单的key-value 模式换支持多中数据结构的存储
    - list
    - set
    - hash
    - zset
    - 等等
  - 一般是做为缓存数据库辅助持久化的数据库
  - 现在市面上用的最多的内存数据库
- momgoDB
  - 高性能,开源,模式自由的文档型数据库
  - 数据都存储在内存中,如果内存不足把不常用的数据保存在磁盘中
  - 虽然是l key-value模式，但是对value（尤其是json）提供了丰富的查询功能
  - 支持二进制数据及大型对象
  - 可以根据数据特点替代RDBMS，成为独立的数据库。或者配合RDBMS，存储特定的数据。
- 列式存储HBASE介绍
  - HBase是**Hadoop**项目中的数据库。它用于需要对大量的数据进行随机、实时读写操作的场景中。HBase的目标就是处理数据量非常庞大的表，可以用普通的计算机处理超过10亿行数据，还可处理有数百万列元素的数据表。
  - 思想来源于bigtable(*BigTable*是Google设计的分布式数据存储系统，用来处理海量的数据的一种非关系型的数据库)

# Redis介绍

> redis官网地址：<https://redis.io/>
>
> 中文网站<http://www.redis.cn/>

## Redis的基本介绍

- Redis是当前比较热门的NoSQL之一
- 它是一个开源的、使用ANSI C语言编写的**key-value**存储系(区别于MySQL的二维表格形式存储)
- 和Memcache类似，但很大程度补偿了Memcache的不足，Redis数据都是缓存在计算机内存中，不同的是，Memcache只能将数据缓存到内存中，无法自动定期写入硬盘，这就表示，一断电或重启，内存清空，数据丢失.

## Redis的应用场景

- **取最新N个数据的操作**

> 比如典型的取网站最新文章，可以将最新的5000条评论ID放在Redis的List集合中，并将超出集合部分从数据库获取

- **排行榜应用，取TOP N操作**

> 这个需求与上面需求的不同之处在于，前面操作以时间为权重，这个是以某个条件为权重，比如按顶的次数排序，可以使用Redis的sorted set，将要排序的值设置成sorted set的score，将具体的数据设置成相应的value，每次只需要执行一条ZADD命令即可。

- **需要精准设定过期时间的应用**

> 比如可以把上面说到的sorted set的score值设置成过期时间的时间戳，那么就可以简单地通过过期时间排序，定时清除过期数据了，不仅是清除Redis中的过期数据，你完全可以把Redis里这个过期时间当成是对数据库中数据的索引，用Redis来找出哪些数据需要过期删除，然后再精准地从数据库中删除相应的记录。

- **计数器应用**

> Redis的命令都是原子性的，你可以轻松地利用INCR，DECR命令来构建计数器系统。

- **Uniq操作，获取某段时间所有数据排重值**

> 这个使用Redis的set数据结构最合适了，只需要不断地将数据往set中扔就行了，set意为集合，所以会自动排重。

- **实时系统，反垃圾系统**

> 通过上面说到的set功能，你可以知道一个终端用户是否进行了某个操作，可以找到其操作的集合并进行分析统计对比等。没有做不到，只有想不到。

- **缓存**

> 将数据直接存放到内存中，性能优于Memcached，数据结构更多样化。

- **Redis的特点**

  - 高效性
    -  Redis读取的速度是110000次/s，写的速度是81000次/s

  - 原子性
    - Redis的所有操作都是原子性的，同时Redis还支持对几个操作合并后的原子性执行。

  - 支持多种数据结构
    - string（字符串）
    - list（列表）
    - hash（哈希）
    - set（集合）
    - zset(有序集合)

  - 稳定性：持久化，主从复制（集群）

  - 其他特性：支持过期时间，支持事务，消息订阅。

# 单机安装

## windows安装Redis

- 解压的目录不要有中文
- 目录结构层次不要太深
- 硬盘空间剩余空间最少要大于你的内存空间，建议20G以上

- 下载 
  - https://github.com/tporadowski/redis/releases
  - ![1621767754161](./assets\1621767754161.png)
- 解压
  - ![1621767826274](./assets\1621767826274.png)
  - ![1621767847455](./assets\1621767847455.png)
- 启动
  - ![1621767938991](./assets\1621767938991.png)
  - ![1621767958032](./assets\1621767958032.png)
  - 这个不要关闭
  - ![1621767986840](./assets\1621767986840.png)
  - ![1621768009497](./assets\1621768009497.png)
  - 这样就OK了

## Linux安装Redis

在node01上执行

```sh
cd /export/software
#上传redis-3.2.8.tar.gz到linux此目录下
mkdir -p /export/server/
tar -zxvf redis-3.2.8.tar.gz -C ../server/
```

安装C程序环境

```sh
yum -y install gcc-c++
```

安装较新版本的tcl

推荐使用

```sh
yum  -y  install  tcl
```

也可

```sh
cd /export/software
wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz
解压tcl
tar -zxvf tcl8.6.1-src.tar.gz -C ../server/
进入指定目录
cd ../server/tcl8.6.1/unix/
./configure
make  && make  install
```

#### **编译redis**

```sh
cd /export/server/redis-3.2.8/
#或者使用命令  make  进行编译
make MALLOC=libc  
make test && make install PREFIX=/export/server/redis-3.2.8
```

配置hosts

w10上在C:\Windows\System32\drivers\etc\hostst添加

Linux上vim /etc/hosts添加

```sh
192.168.88.161 node1 node1.it.cn
192.168.88.162 node2 node2.it.cn
192.168.88.163 node3 node3.it.cn
```

修改redis配置文件

```sh
cd /export/server/redis-3.2.8/
mkdir -p /export/server/redis-3.2.8/log
mkdir -p /export/server/redis-3.2.8/data

vim redis.conf
# 修改第61行，接收的访问地址
bind node1.itcast.cn
# 修改第128行，后台守护执行
daemonize yes
# 修改第163行，日志目录
logfile "/export/server/redis-3.2.8/log/redis.log"
# 修改第247行，数据持久化目录
dir /export/server/redis-3.2.8/data
```

## 启动redis

```sh
cd  /export/server/redis-3.2.8/
bin/redis-server  redis.conf
```

## 连接redis客户端

```sh
cd /export/server/redis-3.2.8/
bin/redis-cli -h node1.it.cn
```

## Redis的数据类型

redis当中一共支持五种数据类型，分别是：

- string字符串

- list列表

- set集合

- hash表

- zset有序集合

通过这五种不同的数据类型，可以实现各种不同的功能，也可以应用在各种不同的场景。

![img](./assets\wps3.png) 

Redis当中各种数据类型结构如上图：

Redis当中各种数据类型的操作

<https://www.runoob.com/redis/redis-keys.html>



```sh
# 进入redis的目录
cd /export/server/redis-3.2.8/
# 启用node1 连接
bin/redis-cli -h node1.it.cn
# 设置一个键和值
node1.it.cn:6379> SET runoobkey redis
OK
# 获取这个键的值
node1.it.cn:6379> get runoobkey
"redis"
# 删除这个键
node1.it.cn:6379> del runoobkey
(integer) 1
# 查看所有的键
node1.it.cn:6379> keys *
(empty list or set)
# 序列化给定 key ，并返回被序列化的值。
## 设置值
node1.it.cn:6379> SET greeting "hello, dumping world!"
OK
## 获取值
node1.it.cn:6379> get greeting 
"hello, dumping world!"
## 序列化存在的key
node1.it.cn:6379> DUMP  greeting
"\x00\x15hello, dumping world!\a\x00,\x7f\xe7\xf1%\xed(W"
## 序列化步存在的key
node1.it.cn:6379> DUMP not-exists-key
(nil)

# 检查指定key是否存在
## 不存在
node1.it.cn:6379> exists a
(integer) 0
## 存在
node1.it.cn:6379> exists greeting
(integer) 1
# 为给定 key 设置过期时间，以秒计
node1.it.cn:6379> set a "hello "
OK
## 这里使用小写报错
node1.it.cn:6379> expirt a
(error) ERR unknown command 'expirt'
## 使用大写
node1.it.cn:6379> EXPIRE a 10
(integer) 1
## 十秒内值存在
node1.it.cn:6379> get a
"hello "
## 十秒外值就不存在
node1.it.cn:6379> get a
(nil)
# 返回 key 所储存的值的类型
## 字符串
node1.it.cn:6379> SET weather "sunny"
OK
node1.it.cn:6379> TYPE weather
string
## 列表
node1.it.cn:6379>  LPUSH book_list "programming in scala"
(integer) 1
node1.it.cn:6379> TYPE book_list
list
## 字典
node1.it.cn:6379> SADD pat "dog"
(integer) 1
node1.it.cn:6379> TYPE pat
set
# 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)
# 移除 key 的过期时间，key 将持久保持
node1.it.cn:6379> set set2 111
OK
node1.it.cn:6379> persist set2
(integer) 0
# 以毫秒为单位返回 key 的剩余的过期时间
node1.it.cn:6379> pttl set2
(integer) -1
# 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。
node1.it.cn:6379> ttl set2
(integer) -1
# 从当前数据库中随机返回一个 key 。
node1.it.cn:6379> keys *
 1) "pat"
 2) "runoobkey"
 3) "key"
 4) "weather"
 5) "a"
 6) "c"
 7) "key1"
 8) "bb"
 9) "book_list"
10) "greeting"
11) "set2"
12) "key2"
13) "d"
14) "hello"
15) "b"
node1.it.cn:6379> randomkey
"runoobkey"
# 修改 key 的名称
node1.it.cn:6379> rename set2 set5
OK
# 仅当 newkey 不存在时，将 key 改名为 newkey 。
node1.it.cn:6379> keys *
 1) "set5"
 2) "pat"
 3) "runoobkey"
 4) "key"
 5) "weather"
 6) "a"
 7) "c"
 8) "key1"
 9) "bb"
10) "book_list"
11) "greeting"
12) "key2"
13) "d"
14) "hello"
15) "b"
node1.it.cn:6379> renamenx set5 b
(integer) 0
node1.it.cn:6379> renamenx set5 set4
(integer) 1
# 返回 key 所储存的值的类型。
node1.it.cn:6379> type a
string
```

### 对字符串的操作

```sh
# 设置指定 key 的值
node1.it.cn:6379> set hello world
OK
# 获取指定 key 的值
node1.it.cn:6379> get hello
"world"
# 将给定 key 的值设为 value ，并返回 key 的旧值
node1.it.cn:6379> getset hello world2
"world"
# 获取所有(一个或多个)给定 key 的值。
node1.it.cn:6379> set a "a"
OK
node1.it.cn:6379> mget hello a
1) "world2"
2) "a"
#将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。
node1.it.cn:6379> setex b 10 "b"
OK
node1.it.cn:6379> get b
"b"
node1.it.cn:6379> get b
(nil)
# 只有在 key 不存在时设置 key 的值。
node1.it.cn:6379> setnx a "b"
(integer) 0
node1.it.cn:6379> setnx b "b"
(integer) 1
# 返回 key 所储存的字符串值的长度。
node1.it.cn:6379> strlen hello
(integer) 6
# 同时设置一个或多个 key-value 对
node1.it.cn:6379> mest a "a" b "b"
(error) ERR unknown command 'mest'
node1.it.cn:6379> mset a "a" b "b"
OK
# 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
node1.it.cn:6379> msetnx a "a" c "c"
(integer) 0
node1.it.cn:6379> msetnx d "d" c "c"
(integer) 1

# 这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。
node1.it.cn:6379> psetex aa 6000 "ggg"
OK
node1.it.cn:6379> get aa
"ggg"
node1.it.cn:6379> get aa
(nil)
# 将 key 中储存的数字值增一。
node1.it.cn:6379> set bb 1
OK
node1.it.cn:6379> incr bb
(integer) 2
node1.it.cn:6379> get bb
"2"
node1.it.cn:6379> incr bb
(integer) 3
node1.it.cn:6379> get bb
"3"
# 将 key 所储存的值加上给定的增量值（increment）
node1.it.cn:6379> incrby bb 5
(integer) 8
node1.it.cn:6379> get bb
"8"
# 将 key 所储存的值加上给定的浮点增量值（increment） 
node1.it.cn:6379> incrbyfloat bb 5.1
"13.1"
# 将 key 中储存的数字值减一
node1.it.cn:6379> set a 1
OK
node1.it.cn:6379> decr a
(integer) 0
# key 所储存的值减去给定的减量值（decrement）
node1.it.cn:6379> set b 10
OK
node1.it.cn:6379> decrby b 3
(integer) 7
node1.it.cn:6379> append a 1
(integer) 2
node1.it.cn:6379> get a
"01"
node1.it.cn:6379> set b 10
OK
```

### **对hash列表的操作**

Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）

基本操作

~~~sh
# 将哈希表 key1中的字段 field 的值设为 value
node1.it.cn:6379> hset key1 field value
(integer) 1
# 获取哈希表key1 中 field 的值
node1.it.cn:6379> hget key1 field
"value"
# 只有在字段 field 不存在时，设置哈希表字段的值。
node1.it.cn:6379> hsetnx key1 field value2
(integer) 0
node1.it.cn:6379> hsetnx key1 field2 value2
(integer) 1
# 同时将多个 field-value (域-值)对设置到哈希表 key 中。
node1.it.cn:6379> hmset key1 field3 value3 field4 value4
OK
# 查看哈希表 key 中，指定的字段是否存在。
node1.it.cn:6379> hexists key1 field
(integer) 1
node1.it.cn:6379> hexists key1 field6
(integer) 0
# 获取存储在哈希表中指定字段的值。
node1.it.cn:6379> hget key1 field2
"value2"
# 获取在哈希表中指定 key 的所有字段和值
node1.it.cn:6379> hgetall key1
 1) "field1"
 2) "value1"
 3) "field"
 4) "value"
 5) "field2"
 6) "value2"
 7) "field3"
 8) "value3"
 9) "field4"
10) "value4"
# 获取所有哈希表中的字段
node1.it.cn:6379> hkeys key1
1) "field1"
2) "field"
3) "field2"
4) "field3"
5) "field4"
# 获取哈希表中字段的数量
node1.it.cn:6379> hlen key1
(integer) 5
# 获取所有给定字段的值
node1.it.cn:6379> hmget key1 field3 field4
1) "value3"
2) "value4"

# 为哈希表 key 中的指定字段的整数值加上增量 
node1.it.cn:6379> hset key2 a 1
(integer) 1
node1.it.cn:6379> hincrby key2 a 3
(integer) 4
node1.it.cn:6379> hget key2 a
"4"
# 为哈希表 key 中的指定字段的浮点数值加上增量
node1.it.cn:6379> hincrbyfloat key2 a 0.8
"4.8"

# 获取哈希表中所有值
node1.it.cn:6379> hvals key1
1) "value1"
2) "value"
3) "value2"
4) "value3"
5) "value4"
# 删除一个或多个哈希表字段
node1.it.cn:6379> hdel key1 field3
(integer) 1
node1.it.cn:6379> hvals key1
1) "value1"
2) "value"
3) "value2"
4) "value4"

~~~

### 对List列表的操作

> list列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）
>
> 一个列表最多可以包含 232 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。
>
> 下表列出了列表相关的基本命令：

```sh
# 将一个或多个值插入到列表头部
node1.it.cn:6379> lpush list1 v1 v2 v3
(integer) 3
# 查看list当中所有的数据
node1.it.cn:6379> lrange list1 0 -1
1) "v3"
2) "v2"
3) "v1"
# 将一个值插入到已存在的列表头部
node1.it.cn:6379> lpushx list1 v4
(integer) 4
node1.it.cn:6379> lrange list1 0 -1
1) "v4"
2) "v3"
3) "v2"
4) "v1"
# 在列表中添加一个或多个值到尾部
node1.it.cn:6379> rpush list1 v5 v6
(integer) 6
node1.it.cn:6379> lrange list1 0 -1
1) "v4"
2) "v3"
3) "v2"
4) "v1"
5) "v5"
6) "v6"
# 为已存在的列表添加单个值到尾部
node1.it.cn:6379> rpushx list1 v7
(integer) 7
node1.it.cn:6379> lrange list1 0 -1
1) "v4"
2) "v3"
3) "v2"
4) "v1"
5) "v5"
6) "v6"
7) "v7"
# 在列表的元素前或者后插入元素
node1.it.cn:6379> linsert list1 before v7 v7-
(integer) 8
node1.it.cn:6379> lrange list1 0 -1
1) "v4"
2) "v3"
3) "v2"
4) "v1"
5) "v5"
6) "v6"
7) "v7-"
8) "v7"
# 通过索引获取列表中的元素
node1.it.cn:6379> lindex list1 6
"v7-"
# 通过索引设置列表元素的值
node1.it.cn:6379> lset list1 0 hello...
OK
node1.it.cn:6379> lrange list1 0 -1
1) "hello..."
2) "v3"
3) "v2"
4) "v1"
5) "v5"
6) "v6"
7) "v7-"
8) "v7"
# 获取列表长度 
node1.it.cn:6379> llen list1
(integer) 8

# 移出并获取列表的第一个元素
node1.it.cn:6379> lpop list1
"hello..."
node1.it.cn:6379> lrange list1 0 -1
1) "v3"
2) "v2"
3) "v1"
4) "v5"
5) "v6"
6) "v7-"
7) "v7"
# 移除列表的最后一个元素，返回值为移除的元素
node1.it.cn:6379> rpop list1
"v7"
node1.it.cn:6379> lrange list1 0 -1
1) "v3"
2) "v2"
3) "v1"
4) "v5"
5) "v6"
6) "v7-"
# 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
node1.it.cn:6379> blpop list1 200
1) "list1"
2) "v3"
node1.it.cn:6379> lrange list1 0 -1
1) "v2"
2) "v1"
3) "v5"
4) "v6"
5) "v7-"
# 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
node1.it.cn:6379> brpop list1 200
1) "list1"
2) "v7-"
node1.it.cn:6379> lrange list1 0 -1
1) "v2"
2) "v1"
3) "v5"
4) "v6"
# 移除列表的最后一个元素，并将该元素添加到另一个列表并返回
node1.it.cn:6379> rpoplpush list1 list2
"v6"
node1.it.cn:6379> lrange list1 0 -1
1) "v2"
2) "v1"
3) "v5"
node1.it.cn:6379> lrange list2 0 -1
1) "v6"
# 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
node1.it.cn:6379> brpoplpush list1 list2 200
"v5"
node1.it.cn:6379> lrange list1 0 -1
1) "v2"
2) "v1"
node1.it.cn:6379> lrange list2 0 -1
1) "v5"
2) "v6"
# 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
node1.it.cn:6379> ltrim list1 0 2
OK
node1.it.cn:6379> lrange list1 0 -1
1) "v2"
2) "v1"
# 删除指定key的列表
node1.it.cn:6379> del list2
(integer) 1

```

### **对位图BitMaps的操作**

- 计算机最小的存储单位是位bit，Bitmaps是针对位的操作的，相较于String、Hash、Set等存储方式更加节省空间

- Bitmaps不是一种数据结构，操作是基于String结构的，一个String最大可以存储512M，那么一个Bitmaps则可以设置2^32个位

- Bitmaps单独提供了一套命令，所以在Redis中使用Bitmaps和使用字符串的方法不太相同。可以**把Bitmaps想象成一个**存储0、1值的数组**，数组的每个单元值**只能存储0和1**，数组的下标在Bitmaps中叫做**偏移量

![img](./assets\wps4.jpg) 

BitMaps 命令说明：**将每个独立用户是否访问过网站存放在Bitmaps中， 将访问的用户记做1， 没有访问的用户记做0， 用偏移量作为用户的id** 。

#### 设置值

```sh
SETBIT key offset value
```

**setbit**命令设置的vlaue只能是**0**或**1**两个值

- 设置键的第offset个位的值（从0算起），假设现在有20个用户，**uid=0，5，11，15，19**的用户对网站进行了访问， 那么当前Bitmaps初始化结果如图所示

![img](./assets\wps5.jpg) 

- 具体操作过程如下， **unique:users:2016-04-05**代表2016-04-05这天的独立访问用户的Bitmaps

- ```sh
  setbit unique:users:2016-04-05 0 1
  setbit unique:users:2016-04-05 5 1
  setbit unique:users:2016-04-05 11 1
  setbit unique:users:2016-04-05 15 1
  setbit unique:users:2016-04-05 19 1
  ```

- 很多应用的用户id以一个指定数字（例如10000） 开头， 直接将用户id和Bitmaps的偏移量对应势必会造成一定的浪费， 通常的做法是每次做setbit操作时将用户id减去这个指定数字。

- 在第一次初始化Bitmaps时， 假如偏移量非常大， 那么整个初始化过程执行会比较慢， 可能会造成Redis的阻塞。

#### 获取值

~~~sh
GETBIT key offset
~~~

- 获取键的第offset位的值（从0开始算），例：下面操作获取id=8的用户是否在2016-04-05这天访问过， 返回0说明没有访问过

####  **获取Bitmaps指定范围值为1的个数**

```sh
BITCOUNT key [start end]
例：下面操作计算2016-04-05这天的独立访问用户数量：
bitcount unique:users:2016-04-05
```

#### **Bitmaps间的运算**

`BITOP operation destkey key [key, …]`

- bitop是一个复合操作， 它可以做多个Bitmaps的and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在destkey中。 假设2016-04-04访问网站的userid=1， 2， 5， 9， 如图3-13所示：

![img](./assets\wps6.jpg)

 ```sh
setbit unique:users:2016-04-04 1 1
setbit unique:users:2016-04-04 2 1
setbit unique:users:2016-04-04 5 1
setbit unique:users:2016-04-04 9 1
 ```

```sh
例1：下面操作计算出2016-04-04和2016-04-05两天都访问过网站的用户数量， 如下所示。

bitop and unique:users:and:2016-04-04_05 unique:users:2016-04-04 unique:users:2016-04-05
bitcount unique:users:2016-04-04_05



例2：如果想算出2016-04-04和2016-04-03任意一天都访问过网站的用户数量（例如月活跃就是类似这种） ， 可以使用or求并集， 具体命令如下：

bitop or unique:users:or:2016-04-04_05 unique:users:2016-04-04 unique:users:2016-04-05
bitcount unique:users:or:2016-04-04_05
```

![1621784539431](./assets\1621784539431.png)

### **对HyperLogLog结构的操作**

#### **应用场景**

HyperLogLog常用于大数据量的去重统计，比如页面访问量统计或者用户访问量统计。

> 要统计一个页面的访问量（PV），可以直接用redis计数器或者直接存数据库都可以实现，如果要统计一个页面的用户访问量（UV），一个用户一天内如果访问多次的话，也只能算一次，这样，我们可以使用**S****ET集合**来做，因为SET集合是有**去重**功能的，key存储页面对应的关键字，value存储对应的userid，这种方法是可行的。但如果访问量较多，假如有几千万的访问量，这就麻烦了。为了统计访问量，要频繁创建SET集合对象。

Redis实现HyperLogLog算法，HyperLogLog 这个数据结构的发明人 是Philippe Flajolet（菲利普·弗拉若莱）教授。Redis 在 2.8.9 版本添加了 HyperLogLog 结构。

####  **UV计算示例**

```sh
node1.it.cn:6379> help @hyperloglog

  PFADD key element [element ...]
  summary: Adds the specified elements to the specified HyperLogLog.
  since: 2.8.9

  PFCOUNT key [key ...]
  summary: Return the approximated cardinality（基数） of the set(s) observed by the HyperLogLog at key(s).
  since: 2.8.9

  PFMERGE destkey sourcekey [sourcekey ...]
  summary: Merge N different HyperLogLogs into a single one.
  since: 2.8.9
```

Redis集成的HyperLogLog使用语法主要有**pfadd**和**pfcount**，顾名思义，一个是来添加数据，一个是来统计的。为什么用**pf**？是因为HyperLogLog 这个数据结构的发明人 是Philippe Flajolet教授 ，所以用发明人的英文缩写，这样容易记住这个语法了。

 

下面我们通过一个示例，来演示如何计算uv。

```sh
node1.it.cn:6379> pfadd uv user1
(integer) 1
node1.it.cn:6379> keys *
1) "uv"
node1.it.cn:6379> pfcount uv
(integer) 1
node1.it.cn:6379> pfadd uv user2
(integer) 1
node1.it.cn:6379> pfcount uv
(integer) 2
node1.it.cn:6379> pfadd uv user3
(integer) 1
node1.it.cn:6379> pfcount uv
(integer) 3
node1.it.cn:6379> pfadd uv user4
(integer) 1
node1.it.cn:6379> pfcount uv
(integer) 4
node1.it.cn:6379> pfadd uv user5 user6 user7 user8 user9 user10
(integer) 1
node1.it.cn:6379> pfcount uv
(integer) 10
```

HyperLogLog算法一开始就是为了大数据量的统计而发明的，所以很适合那种数据量很大，然后又没要求不能有一点误差的计算，HyperLogLog 提供不精确的去重计数方案，虽然不精确但是也不是非常不精确，标准误差是 0.81%，不过这对于页面用户访问量是没影响的，因为这种统计可能是访问量非常巨大，但是又没必要做到绝对准确，访问量对准确率要求没那么高，但是性能存储方面要求就比较高了，而HyperLogLog正好符合这种要求，不会占用太多存储空间，同时性能不错

 

**pfadd**和**pfcount**常用于统计，需求：假如两个页面很相近，现在想统计这两个页面的用户访问量呢？这里就可以用**pfmerge**合并统计了，语法如例子：

```sh
node1.it.cn:6379> pfadd page1 user1 user2 user3 user4 user5
(integer) 1
node1.it.cn:6379> pfadd page2 user1 user2 user3 user6 user7
(integer) 1
node1.it.cn:6379> pfmerge page1+page2 page1 page2
OK
node1.it.cn:6379> pfcount page1+page2
(integer) 7
```

####  **HyperLogLog为什么适合做大量数据的统计**

- Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。

- 在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

- 但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

> 什么是基数？
>
> 比如：数据集{1, 3, 5, 7, 5, 7, 8}，那么这个数据集的基数集{1, 3, 5, 7, 8}，基数（不重复元素）为5。基数估计就是在误差可接受的范围内，快速计算基数。

