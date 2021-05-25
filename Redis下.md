# Redis Java API操作

## 创建maven工程

| groupId    | cn.it    |
| ---------- | -------- |
| artifactId | redis_op |

## 导入POM依赖

```xml
<dependencies>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>6.14.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                    <!--    <verbal>true</verbal>-->
                </configuration>
            </plugin>
        </plugins>
    </build>
```

## 连接以及关闭redis客户端

> 实现步骤：
>
> 1. 创建JedisPoolConfig配置对象，指定最大空闲连接为10个、最大等待时间为3000毫秒、最大连接数为50、最小空闲连接5个
>
> 2. 创建JedisPool
>
> 3. 使用@Test注解，编写测试用例，查看Redis中所有的key
>
>    a) 从Redis连接池获取Redis连接
>
>    b) 调用keys方法获取所有的key
>
>    c) 遍历打印所有key

```java
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.util.Set;

public class RedisTest {
    private JedisPool jedisPool;
    private JedisPoolConfig config;

    @BeforeTest
    public void redisConnectionPool(){
        config = new JedisPoolConfig();
        config.setMaxIdle(10);
        config.setMaxWaitMillis(3000);
        config.setMaxTotal(50);
        config.setMinIdle(5);
        jedisPool = new JedisPool(config, "127.0.0.1", 6379);
    }

    @Test
    public void testConnect() {
        Jedis jedis = jedisPool.getResource();
        Set<String> keySet = jedis.keys("*");
        for (String s : keySet) {
            System.out.print(s + " ");
        }
    }

    @AfterTest
    public void closePool(){
        jedisPool.close();
    }
}
```

## **操作string类型数据**

> 1. 添加一个string类型数据，key为pv，用于保存pv的值，初始值为0
>
> 2. 查询该key对应的数据
>
> 3. 修改pv为1000
>
> 4. 实现整形数据原子自增操作 +1
>
> 5. 实现整形该数据原子自增操作 +1000

```java
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.util.Set;

public class RedisTest {
    private JedisPool jedisPool;
    private JedisPoolConfig config;

    @BeforeTest
    public void redisConnectionPool(){
        config = new JedisPoolConfig();
        config.setMaxIdle(10);
        config.setMaxWaitMillis(3000);
        config.setMaxTotal(50);
        config.setMinIdle(5);
        jedisPool = new JedisPool(config, "127.0.0.1", 6379);
    }

    @Test
    public void stringOpTest(){
        Jedis connection = jedisPool.getResource();
        // 1.  添加一个string类型数据，key为pv，初始值为0
        connection.set("pv","0");
        // 2.  查询该key对应的数据
        System.out.println("原始pv为:" + connection.get("pv"));
        // 3.  修改pv为1000
        connection.set("pv", "1000");
        System.out.println("修改pv为:" + connection.get("pv"));
        // 4.  实现整形数据原子自增操作 +1
        connection.incr("pv");
        System.out.println("pv自增1：" + connection.get("pv"));

        // 5.  实现整形该数据原子自增操作 +1000
        connection.incrBy("pv", 1000);
        System.out.println("pv自增1000：" + connection.get("pv"));

    }

    @AfterTest
    public void closePool(){
        jedisPool.close();
    }
}
```

![1621859370167](./assets\1621859370167.png)

##  **操作hash列表类型数据**

1. 往Hash结构中添加以下商品库存

   a) iphone11 => 10000

   b) macbookpro => 9000

2. 获取Hash中所有的商品

3. 新增3000个macbookpro库存

4. 删除整个Hash的数据

```sql
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.util.Map;
import java.util.Set;

public class RedisTest {
    private JedisPool jedisPool;
    private JedisPoolConfig config;

    @BeforeTest
    public void redisConnectionPool(){
        config = new JedisPoolConfig();
        config.setMaxIdle(10);
        config.setMaxWaitMillis(3000);
        config.setMaxTotal(50);
        config.setMinIdle(5);
        jedisPool = new JedisPool(config, "127.0.0.1", 6379);
    }


    @Test
    public void hashOpTest() {
        Jedis connection = jedisPool.getResource();
        // 1.  往Hash结构中添加以下商品库存
        // a)  iphone11 => 10000
        // b)  macbookpro => 9000
        connection.hset("goodsStore", "iphone11", "10000");
        connection.hset("goodsStore", "macbookpro", "9000");

        // 2.  获取Hash中所有的商品
        Map<String, String> keyValues = connection.hgetAll("goodsStore");
        for (String s : keyValues.keySet()) {
            System.out.println(s + " => " + keyValues.get(s));
        }

        // 3.  修改Hash中macbookpro数量为12000
        // 方式一：
        // connection.hset("goodsStore", "macbookpro", "12000");
        // 方式二：
        connection.hincrBy("goodsStore", "macbookpro", 3000);
        System.out.println("新增3000个库存后：macbookpro => " + connection.hget("goodsStore", "macbookpro"));

        // 4.  删除整个Hash的数据
        connection.del("goodsStore");
    }

    @AfterTest
    public void closePool(){
        jedisPool.close();
    }
}
```

![1621859518838](./assets\1621859518838.png)

## **操作list类型数据**

1. 向list的左边插入以下三个手机号码：18511310001、18912301231、18123123312

2. 从右边移除一个手机号码

3. 获取list所有的值

```java
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.util.List;
import java.util.Map;
import java.util.Set;

public class RedisTest {
    private JedisPool jedisPool;
    private JedisPoolConfig config;

    @BeforeTest
    public void redisConnectionPool(){
        config = new JedisPoolConfig();
        config.setMaxIdle(10);
        config.setMaxWaitMillis(3000);
        config.setMaxTotal(50);
        config.setMinIdle(5);
        jedisPool = new JedisPool(config, "127.0.0.1", 6379);
    }


    @Test
    public void listOpTest() {
        Jedis connection = jedisPool.getResource();
        // 1.  向list的左边插入以下三个手机号码：18511310001、18912301231、18123123312
        connection.lpush("telephone", "18511310001", "18912301231", "18123123312");

        // 2.  从右边移除一个手机号码
        connection.rpop("telephone");

        // 3.  获取list所有的值
        List<String> telList = connection.lrange("telephone", 0, -1);
        for (String tel : telList) {
            System.out.print(tel + " ");
        }
    }

    @AfterTest
    public void closePool(){
        jedisPool.close();
    }
}
```



![1621864937407](./assets\1621864937407.png)

## **操作set类型的数据**

使用set来保存uv值，为了方便计算，将用户名保存到uv中。

1. 往一个set中添加页面 page1 的uv，用户user1访问一次该页面

2. user2访问一次该页面

3. user1再次访问一次该页面

4. 最后获取 page1的uv值

```java
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.util.List;
import java.util.Map;
import java.util.Set;

public class RedisTest {
    private JedisPool jedisPool;
    private JedisPoolConfig config;

    @BeforeTest
    public void redisConnectionPool(){
        config = new JedisPoolConfig();
        config.setMaxIdle(10);
        config.setMaxWaitMillis(3000);
        config.setMaxTotal(50);
        config.setMinIdle(5);
        jedisPool = new JedisPool(config, "127.0.0.1", 6379);
    }


    @Test
    public void setOpTest() {
        Jedis connection = jedisPool.getResource();

        // 1.  往一个set中添加页面 page1 的uv，用户user1访问一次该页面
        connection.sadd("page1", "user1");

        // 2.  user2访问一次该页面
        connection.sadd("page1", "user2");

        // 3.  user1再次访问一次该页面
        connection.sadd("page1", "user1");

        // 4.  最后获取 page1的uv值
        Long uv = connection.scard("page1");
        System.out.println("page1页面的UV为：" + uv);
    }

    @AfterTest
    public void closePool(){
        jedisPool.close();
    }
}
```

# **Redis的持久化**

> 由于redis是一个内存数据库，所有的数据都是保存在内存当中的，内存当中的数据极易丢失，所以redis的数据持久化就显得尤为重要，在redis当中，提供了两种数据持久化的方式，分别为RDB以及AOF，且Redis默认开启的数据持久化方式为RDB方式。

##  **RDB持久化方案**

### **介绍**

> Redis会定期保存数据快照至一个rbd文件中，并在启动时自动加载rdb文件，恢复之前保存的数据。可以在配置文件中配置Redis进行快照保存的时机：

`save [seconds][changes]`

> 意为在seconds秒内如果发生了changes次数据修改，则进行一次RDB快照保存，例如

`save 60 100`

> 会让Redis每60秒检查一次数据变更情况，如果发生了100次或以上的数据变更，则进行RDB快照保存。可以配置多条save指令，让Redis执行多级的快照保存策略。Redis默认开启RDB快照。也可以通过SAVE或者BGSAVE命令手动触发RDB快照保存。SAVE 和 BGSAVE 两个命令都会调用 rdbSave 函数，但它们调用的方式各有不同：

- SAVE 直接调用 rdbSave ，阻塞 Redis 主进程，直到保存完成为止。在主进程阻塞期间，服务器不能处理客户端的任何请求。

- BGSAVE 则 fork 出一个子进程，子进程负责调用 rdbSave ，并在保存完成之后向主进程发送信号，通知保存已完成。 Redis 服务器在BGSAVE 执行期间仍然可以继续处理客户端的请求。

### **RDB方案优点**

1. 对性能影响最小。如前文所述，Redis在保存RDB快照时会fork出子进程进行，几乎不影响Redis处理客户端请求的效率。

2. 每次快照会生成一个完整的数据快照文件，所以可以辅以其他手段保存多个时间点的快照（例如把每天0点的快照备份至其他存储媒介中），作为非常可靠的灾难恢复手段。

3. 使用RDB文件进行数据恢复比使用AOF要快很多

### **RDB方案缺点**

1. 快照是定期生成的，所以在Redis crash时或多或少会丢失一部分数据

2. 如果数据集非常大且CPU不够强（比如单核CPU），Redis在fork子进程时可能会消耗相对较长的时间，影响Redis对外提供服务的能力

### **RDB配置**

1.  修改redis的配置文件

```sh
cd /export/server/redis-3.2.8/
vim redis.conf
# 第202行
save 900 1
save 300 10
save 60 10000
save 5 1
```

这三个选项是redis的配置文件默认自带的存储机制。表示每隔多少秒，有多少个key发生变化就生成一份dump.rdb文件，作为redis的快照文件

例如：save  60  10000 表示在60秒内，有10000个key发生变化，就会生成一份redis的快照

查看备份

```sh
[root@node1 redis-3.2.8]# cd data
[root@node1 data]# ll
总用量 4
-rw-r--r-- 1 root root 452 5月  24 16:44 dump.rdb
```

2.  重新启动redis服务

每次生成新的dump.rdb都会覆盖掉之前的老的快照

```sh
ps -ef | grep redis
bin/redis-cli -h node1.it.cn shutdown
bin/redis-server redis.conf
```

## **AOF持久化方案**

### **介绍**

采用AOF持久方式时，Redis会把每一个写请求都记录在一个日志文件里。在Redis重启时，会把AOF文件中记录的所有写操作顺序执行一遍，确保数据恢复到最新。

### 开启AOF

AOF默认是关闭的，如要开启，进行如下配置：

# 第594行appendonly yes

###  配置AOF

AOF提供了三种fsync配置：always/everysec/no，通过配置项[appendfsync]指定：

1. **appendfsync no**：不进行fsync，将flush文件的时机交给OS决定，速度最快

2. **appendfsync always**：每写入一条日志就进行一次fsync操作，数据安全性最高，但速度最慢

3. **appendfsync everysec**：折中的做法，交由后台线程每秒fsync一次

### AOF rewrite

> 随着AOF不断地记录写操作日志，因为所有的写操作都会记录，所以必定会出现一些无用的日志。大量无用的日志会让AOF文件过大，也会让数据恢复的时间过长。不过Redis提供了AOF rewrite功能，可以重写AOF文件，只保留能够把数据恢复到最新状态的最小写操作集。

 AOF rewrite可以通过BGREWRITEAOF命令触发，也可以配置Redis定期自动进行：

```sh
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

- Redis在每次AOF rewrite时，会记录完成rewrite后的AOF日志大小，当AOF日志大小在该基础上增长了100%后，自动进行AOF rewrite

- auto-aof-rewrite-min-size最开始的AOF文件必须要触发这个文件才触发，后面的每次重写就不会根据这个变量了。该变量仅初始化启动Redis有效。

### **AOF优点**

1. 最安全，在启用appendfsync为always时，任何已写入的数据都不会丢失，使用在启用appendfsync everysec也至多只会丢失1秒的数据

2. AOF文件在发生断电等问题时也不会损坏，即使出现了某条日志只写入了一半的情况，也可以使用redis-check-aof工具轻松修复

3. AOF文件易读，可修改，在进行某些错误的数据清除操作后，只要AOF文件没有rewrite，就可以把AOF文件备份出来，把错误的命令删除，然后恢复数据。

### **AOF的缺点**

1. AOF文件通常比RDB文件更大

2. 性能消耗比RDB高

3. 数据恢复速度比RDB慢

> Redis的数据持久化工作本身就会带来延迟，需要根据数据的安全级别和性能要求制定合理的持久化策略：
>
> - AOF + fsync always的设置虽然能够绝对确保数据安全，但每个操作都会触发一次fsync，会对Redis的性能有比较明显的影响
>
> - AOF + fsync every second是比较好的折中方案，每秒fsync一次
>
> - AOF + fsync never会提供AOF持久化方案下的最优性能
>
> 使用RDB持久化通常会提供比使用AOF更高的性能，但需要注意RDB的策略配置

### RDB or AOF

> 每一次RDB快照和AOF Rewrite都需要Redis主进程进行fork操作。fork操作本身可能会产生较高的耗时，与CPU和Redis占用的内存大小有关。根据具体的情况合理配置RDB快照和AOF Rewrite时机，避免过于频繁的fork带来的延迟

> Redis在fork子进程时需要将内存分页表拷贝至子进程，以占用了24GB内存的Redis实例为例，共需要拷贝48MB的数据。在使用单Xeon 2.27Ghz的物理机上，这一fork操作耗时216ms。

# **Redis 高级使用**(了解)

## **Redis 事务**(啥用都没有面试问到面试官是傻逼)

###  **Redis事务简介**

> Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

**总结说：Redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令**

- **Redis事务没有隔离级别的概念**

> 批量操作在发送 EXEC 命令前被放入队列缓存，并不会被实际执行，也就不存在事务内的查询要看到事务里的更新，事务外查询不能看到。

- **Redis不保证原子性**

> Redis中，单条命令是原子性执行的，但事务不保证原子性，且没有回滚。事务中任意命令执行失败，其余的命令仍会被执行。

一个事务从开始到执行会经历以下三个阶段：

- 第一阶段：开始事务

- 第二阶段：命令入队

- 第三阶段、执行事务。

 

Redis事务相关命令：

- MULTI

> 开启事务，redis会将后续的命令逐个放入队列中，然后使用EXEC命令来原子化执行这个命令队列

- EXEC

> 执行事务中的所有操作命令

- DISCARD

> 取消事务，放弃执行事务块中的所有命令

- WATCH

> 监视一个或多个key，如果事务在执行前，这个key（或多个key）被其他命令修改，则事务被中断，不会执行事务中的任何命令

- UNWATCH

> 取消WATCH对所有key的监视

### **Redis事务演示**

1. MULTI开始一个事务：给k1、k2分别赋值，在事务中修改k1、k2，执行事务后，查看k1、k2值都被修改。

```sh
node1.it.cn:6379> set k1 v1
OK
node1.it.cn:6379> set k2 v2
OK
node1.it.cn:6379> multi
OK
node1.it.cn:6379> set k1 11
QUEUED
node1.it.cn:6379> set k2 22
QUEUED
node1.it.cn:6379> exec
1) OK
2) OK
node1.it.cn:6379> get k1
"11"
node1.it.cn:6379> get k2
"22"
```

2. 事务失败处理：语法错误（编译器错误），在开启事务后，修改k1值为11，k2值为22，但k2语法错误，最终导致事务提交失败，k1、k2保留原值。

```sh
node1.it.cn:6379> flushdb
OK
node1.it.cn:6379> keys *
(empty list or set)
node1.it.cn:6379> set k1 v1
OK
node1.it.cn:6379> set k2 v2
OK
node1.it.cn:6379> multi
OK
node1.it.cn:6379> set k1 11
QUEUED
node1.it.cn:6379> sets k2 22
(error) ERR unknown command 'sets'
node1.it.cn:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
node1.it.cn:6379> get k1
"v1"
node1.it.cn:6379> get k2
"v2"
```

3. Redis类型错误（运行时错误），在开启事务后，修改k1值为11，k2值为22，但将k2的类型作为List，在运行时检测类型错误，最终导致事务提交失败，此时事务并没有回滚，而是跳过错误命令继续执行， 结果k1值改变、k2保留原值。

```sh
node1.it.cn:6379> flushdb
OK
node1.it.cn:6379> keys *
(empty list or set)
node1.it.cn:6379> set k1 v1
OK
node1.it.cn:6379> set k2 v2
OK
node1.it.cn:6379> multi
OK
node1.it.cn:6379> set k1 11
QUEUED
node1.it.cn:6379> lpush k2 22
QUEUED
node1.it.cn:6379> exec
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
node1.it.cn:6379> get k1
"11"
node1.it.cn:6379> get k2
"v2"
```

**4. DISCARD**取消事务

```sh
node1.it.cn:6379> multi
OK
node1.it.cn:6379> set k6 v6
QUEUED
node1.it.cn:6379> set k7 v7
QUEUED
node1.it.cn:6379> discard
OK
node1.it.cn:6379> get k6
(nil)
node1.it.cn:6379> get k7
(nil)
```

### **为什么Redis不支持事务回滚？**

> 多数事务失败是由语法错误或者数据结构类型错误导致的，语法错误说明在命令入队前就进行检测的，而类型错误是在执行时检测的，Redis为提升性能而采用这种简单的事务，这是不同于关系型数据库的，特别要注意区分。Redis之所以保持这样简易的事务，完全是为了保证高并发下的核心问题——**性能**。

## **Redis 过期策略**

> Redis是key-value数据库，可以设置Redis中缓存的key的过期时间。Redis的过期策略就是指当Redis中缓存的key过期了，Redis如何处理。

过期策略通常有以下三种：

- **定时过期**

> 每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，**对内存很友好**；但是会**占用大量的CPU资源**去处理过期的数据，从而影响缓存的响应时间和吞吐量。

- **惰性过期**

> 只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以**最大化地节省CPU资源，却对内存非常不友好**。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。

- **定期过期**

> 每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。

## **内存淘汰策略**

Redis的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据，在Redis的配置文件中描述如下：

```sh
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#最大内存策略：当到达最大使用内存时，你可以在下面5种行为中选择，Redis如何选择淘汰数据库键
#当内存不足以容纳新写入数据时

# volatile-lru -> remove the key with an expire set using an LRU algorithm
# volatile-lru ：在设置了过期时间的键空间中，移除最近最少使用的key。这种情况一般是把 redis 既当缓存，又做持久化存储的时候才用。

# allkeys-lru -> remove any key according to the LRU algorithm
# allkeys-lru ： 移除最近最少使用的key （推荐）

# volatile-random -> remove a random key with an expire set
# volatile-random ： 在设置了过期时间的键空间中，随机移除一个键，不推荐

# allkeys-random -> remove a random key, any key
# allkeys-random ： 直接在键空间中随机移除一个键，弄啥叻

# volatile-ttl -> remove the key with the nearest expire time (minor TTL)
# volatile-ttl ： 在设置了过期时间的键空间中，有更早过期时间的key优先移除 不推荐

# noeviction -> don't expire at all, just return an error on write operations
# noeviction ： 不做过键处理，只返回一个写操作错误。 不推荐

# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
# 上面所有的策略下，在没有合适的淘汰删除的键时，执行写操作时，Redis 会返回一个错误。下面是写入命令：
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort

# 过期策略默认是：
# The default is:
# maxmemory-policy noeviction
```

实际项目中设置内存淘汰策略：**maxmemory-policy allkeys-lru**，移除最近最少使用的key。

# **Redis的主从复制架构**

## **简介**

主从复制，是指**将一台Redis服务器的数据，复制到其他的Redis服务器**。前者称为主节点(master)，后者称为从节点(slave)，数据的复制是单向的，只能由主节点到从节点。

![img](./assets\wps1.jpg) 

默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

### **一主一从**

​	如下图所示左边是Master节点，右边是slave节点，即主节点和从节点。从节点也是可以对外提供服务的，主节点是有数据的，从节点可以通过复制操作将主节点的数据同步过来，并且随着主节点数据不断写入，从节点数据也会做同步的更新。

![img](./assets\wps2-1621869571336.jpg) 

**从节点起到的就是数据备份的效果**。

### **一主多从**

除了一主一从模型之外，Redis还提供了一主多从的模型，也就是一个master可以有多个slave，也就相当于有了多份的数据副本。

![img](./assets\wps3.jpg) 

可以做一个更加高可用的选择，例如一个master和一个slave挂掉了，还能有其他的slave数据备份。

### **主从复制原理**

1. 当从数据库启动后，会向主数据库发送SYNC命令

2. 主数据库接收到SYNC命令后开始在后台保存快照（RDB持久化），并将保存快照期间接收到的命令缓存再来

3. 快照完成后，Redis（Master）将快照文件和所有缓存的命令发送给从数据库

4. Redis（Slave）接收到RDB和缓存命令时，会开始载入快照文件并执行接收到的缓存的命令

5. 后续，每当主数据库接收到写命令时，就会将命令同步给从数据库。所以3和4只会在初始化的时候执行

## **主从复制的应用场景**

### **读写分离**

- 通过主从复制可以实现读写分离，以提高服务器的负载能力

- 在常见的场景中（例如：电商网站），读的频率大于写

- 当单机Redis无法应付大量的读请求时（尤其是消耗资源的请求），就可以通过主从复制功能来建立多个从数据库节点，主数据库只进行写操作，从数据库负责读操作

- 这种主从复制，比较适合用来处理读多写少的场景，而当单个主数据库不能满足需求时，就需要使用Redis 3.0后推出的集群功能

### **从数据库持久化**

- Redis中相对耗时的操作就是持久化，为了提高性能，可以通过主从复制创建一个或多个从数据库，并在从数据库中启用持久化，同时在主数据库中禁用持久化（例如：禁用AOF）

- 当从数据库崩溃重启后主数据库会自动将数据同步过来，无需担心数据丢失

- 而当主数据库崩溃时，后续我们可以通过哨兵（Sentinel）来解决

# **另外两台服务器安装Redis**

## **安装Redis依赖环境**

在node2.it.cn和node3.it.cn执行以下命令安装依赖环境

`yum -y install gcc-c++`

##  **上传Redis压缩包**

在node2.it.cn与node3.it服务器上面上传Redis压缩包，然后进行解压，并将安装包上传到/export/software路径下

```sh
cd /export/software
#上传tar包到目录下
mkdir ../server/
tar -zxvf redis-3.2.8.tar.gz -C ../server/
```

## **服务器安装tcl**

在node2.it.cn与node3.it.cn服务器执行以下命令在线装TCL

`yum  -y  install  tcl`

## **编译redis**

node2.it.cn与node3.it.cn执行以下命令进行编译Redis

执行以下命令进行编译：

```sh
cd /export/server/redis-3.2.8/
#或者使用命令  make  进行编译
make MALLOC=libc   
make test && make install PREFIX=/export/server/redis-3.2.8
```

## **修改redis配置文件**

**node2.it.cn服务器修改配置文件**

```sh
cd /export/server/redis-3.2.8/
mkdir -p /export/server/redis-3.2.8/log
mkdir -p /export/server/redis-3.2.8/data

vim redis.conf
# 修改第61行
bind node2.itcast.cn
# 修改第128行
daemonize yes
# 修改第163行
logfile "/export/server/redis-3.2.8/log/redis.log"
# 修改第247行
dir /export/server/redis-3.2.8/data
# 修改第266行，配置node2.it.cn为第一台服务器的slave节点
slaveof node1.it.cn 6379
```

**node3.it.cn服务器修改配置文件**

```sh
cd /export/server/redis-3.2.8/
mkdir -p /export/server/redis-3.2.8/log
mkdir -p /export/server/redis-3.2.8/data

vim redis.conf
# 修改第61行
bind node3.itcast.cn
# 修改第128行
daemonize yes
# 修改第163行
logfile "/export/server/redis-3.2.8/log/redis.log"
# 修改第247行
dir /export/server/redis-3.2.8/data
# 修改第266行，配置node3.it.cn为第一台服务器的slave节点
slaveof node1.it.cn 6379
```

##   **启动Redis服务**

```sh
node2.it.cn执行以下命令启动Redis服务
cd  /export/server/redis-3.2.8/
bin/redis-server redis.conf

node3.it.cn执行以下命令启动Redis服务
cd  /export/server/redis-3.2.8/
bin/redis-server redis.conf
```

启动成功便可以实现redis的主从复制，node1.it.cn可以读写操作，node2.itcast.cn与node3.it.cn只支持读取操作。

# Redis中的Sentinel架构

## **Sentinel介绍**

Sentinel（哨兵）是Redis的高可用性解决方案：由一个或多个Sentinel实例 组成的Sentinel系统可以监视任意多个主服务器，以及这些主服务器属下的所有从服务器，并在被监视的主服务器进入下线状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器。

例如：

![img](./assets\wps4-1621870968946.jpg) 



在

Server1 

掉线后：

![1621871073572](./assets\1621871073572.png)





升级Server2 为新的主服务器：![img](./assets\wps6-1621870968947.jpg)



##  **配置哨兵**

![img](./assets\wps7.jpg) 

## **三台机器修改哨兵配置文件**

## 三台机器执行以下命令修改redis的哨兵配置文件

```sh
cd /export/server/redis-3.2.8
vim sentinel.conf
```

**配置监听的主服务器**

1. 修改node1.it.cn的sentinel.conf文件

```sh
#修改第15行， bind配置，每台机器修改为自己对应的主机名
bind node1.it.cn  
# 在下方添加配置，让sentinel服务后台运行
daemonize yes
#修改第71行，三台机器监控的主节点，现在主节点是node1.itcast.cn服务器
sentinel monitor mymaster node1.it.cn 6379 2
```

参数说明

- sentinel monitor代表监控

- mymaster代表服务器的名称，可以自定义

- node1.it.cn代表监控的主服务器，6379代表端口

2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。

如果Redis是有密码的，需要指定密码

```sh
# sentinel author-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
# sentinel auth-pass <master-name> <password>
```

3分发到node2.it.cn和node3.it.cn

```sh
scp sentinel.conf node2.it.cn:$PWD
scp sentinel.conf node3.it.cn:$PWD
```

4分别修改配置中bind的服务器主机名

node2.it.cn

```sh
cd /export/server/redis-3.2.8
vim sentinel.conf
# 修改第18行
bind node2.it.cn
```

node3.it.cn

```sh
cd /export/server/redis-3.2.8
vim sentinel.conf
# 修改第18行
bind node3.it.cn
```

##  **三台机器启动哨兵服务**

 ```sh
cd /export/server/redis-3.2.8
bin/redis-sentinel sentinel.conf
 ```

```sh
node1.it.cn
[root@node1 redis-3.2.8]# ps aux | grep redis
root      18911  0.0  0.0 136920  2456 ?        Ssl  08:58   0:04 bin/redis-server node1.it.cn:6379
root      19112  0.0  0.0 149232  5152 pts/1    S+   09:16   0:00 vim redis.conf
root      20544  0.1  0.0 135728  2328 ?        Ssl  10:48   0:00 bin/redis-sentinel node1.it.cn:26379 [sentinel]
root      20548  0.0  0.0 112712   960 pts/3    S+   10:48   0:00 grep --color=auto redis

node2.it.cn
[root@node2 redis-3.2.8]# ps aux | grep redis
root      26260  0.0  0.1 139200  4456 ?        Ssl  10:34   0:00 bin/redis-server node2.it.cn:6379
root      26421  0.1  0.0 139204  2440 ?        Ssl  10:48   0:00 bin/redis-sentinel node2.it.cn:26379 [sentinel]
root      26438  0.0  0.0 112812   972 pts/1    S+   10:49   0:00 grep --color=auto redis

node3.it.cn
[root@node3 redis-3.2.8]# ps aux | grep redis
root      22325  0.0  0.0 135992  2376 ?        Ssl  10:34   0:00 bin/redis-server node3.it.cn:6379
root      22463  0.1  0.0 135836  2384 ?        Ssl  10:48   0:00 bin/redis-sentinel node3.it.cn:26379 [sentinel]
root      22475  0.0  0.0 112812   972 pts/1    S+   10:49   0:00 grep --color=auto redis
```

node1服务器杀死redis服务进程

```sh
查看Sentinel master的状态
bin/redis-cli -h node2.it.cn -p 26379
使用ping命令检查哨兵是否工作，如果正常会返回PONG
node2.it.cn:26379> ping
PONG
node2.it.cn:26379> info
... ... ...

# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.88.161:6379,slaves=2,sentinels=3
```

# Redis的sentinel模式代码开发连接

通过哨兵连接，要指定哨兵的地址，并使用JedisSentinelPool来创建连接池。

实现步骤：

1. 在 cn.it.redis.api_test 包下创建一个新的类 ReidsSentinelTest

2. 构建JedisPoolConfig配置对象

3. 创建一个HashSet，用来保存哨兵节点配置信息（记得一定要写端口号）

4. 构建JedisSentinelPool连接池

5. 使用sentinelPool连接池获取连接

 ```java
public class ReidsSentinelTest {
    private JedisPoolConfig jedisPoolConfig;
    private JedisSentinelPool jedisSentinelPool;

    @BeforeTest
    public void beforeTest() {
        // 1. 构建JedisPoolConfig配置对象
        jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(50);
        jedisPoolConfig.setMaxIdle(10);
        jedisPoolConfig.setMinIdle(5);
        jedisPoolConfig.setMaxWaitMillis(10000);

        // 2. 创建一个HashSet，用来保存哨兵节点配置信息
        HashSet<String> sentinelNodeSet = new HashSet<>(Arrays.asList("node1.it.cn:26379", "node2.it.cn:26379", "node3.it.cn:26379"));

        // 3. 构建JedisSentinelPool连接池
        jedisSentinelPool = new JedisSentinelPool("mymaster", sentinelNodeSet, jedisPoolConfig);

    }

    @Test
    public void keysOpTest() {
        // 使用sentinelPool连接池获取连接
        Jedis connection = jedisSentinelPool.getResource();
        Set<String> keySet = connection.keys("*");

        for (String key : keySet) {
            System.out.println(key);
        }
    }

    @AfterTest
    public void afterTest() {
        jedisSentinelPool.close();
    }
}
 ```

# **RedisCluster集群**

Redis最开始使用**主从模式做集群**，若master宕机需要手动配置slave转为master；后来为了**高可用提出来哨兵模式**，该模式下有一个哨兵监视master和slave，若master宕机可自动将slave转为master，但它也有一个问题，就是不能动态扩充；所以在Redis 3.x提出**cluster集群**模式。

## **引言**

Redis Cluster是Redis官方提供的Redis集群功能，为什么要实现Redis Cluster？

1. 主从复制不能实现**高可用**

2. 随着公司发展，用户数量增多，并发越来越多，业务需要更高的QPS，而主从复制中单机的**QPS**可能无法满足业务需求；

3. 数据量的考虑，现有服务器内存不能满足业务数据的需要时，单纯向服务器添加内存不能达到要求，此时需要考虑**分布式**需求，把数据分布到不同服务器上；

4. 网络流量需求，业务的流量已经超过服务器的网卡的上限值，可考虑使用分布式来进行**分流；**

5. 离线计算，需要中间环节**缓冲**等其他需求；

在存储引擎框架（MySQL、HDFS、HBase、Redis、Elasticsearch等）中，只要数据量很大时，单机无法承受压力，最好的方式就是：**数据分布**进行存储管理。

**对Redis 内存数据库来说：全量数据，单机Redis节点无法满足要求，按照分区规则把数据分到若干个子集当中。**

![img](./assets\wps8.jpg) 

Redis集群数据分布方式：

**虚拟槽分区**：虚拟槽分区是Redis Cluster采用的分区方式，预设虚拟槽，每个槽就相当于一个数字，有一定范围。每个槽映射一个数据子集，一般比节点数大。Redis Cluster中预设虚拟槽的范围为0到16383。

![img](./assets\wps9.jpg) 

## **Redis Cluster 设计**

Redis Cluster是分布式架构，有多个节点，每个节点都负责进行数据读写操作，每个节点之间会进行通信。Redis Cluster采用**无中心结构**，每个节点保存数据和整个集群状态，每个节点都和其他所有节点连接。

![img](./assets\wps10.jpg) 

结构特点：

- 所有的redis节点彼此互联(PING-PONG机制)，内部使用**二进制协议**优化传输速度和带宽；

- 节点的fail是通过集群中**超过半数**的节点检测失效时才生效；

- 客户端与redis节点直连，不需要中间proxy层，客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可；

- redis-cluster 把所有的物理节点映射到**[0-16383]slot**上（不一定是平均分配）,cluster 负责维护**node<->slot<->value**；

- Redis集群预分好**16384**个桶（Slot），当需要在 Redis 集群中放置一个 key-value 时，根据 CRC16(key) & 16384的值，决定将一个key放到哪个桶中；

![img](./assets\wps11.jpg) 

Redis 集群的优势：

- **缓存永不宕机**：启动集群，永远让集群的一部分起作用。主节点失效了子节点能迅速改变角色成为主节点，整个集群的部分节点失败或者不可达的情况下能够继续处理命令；

- **迅速恢复数据**：持久化数据，能在宕机后迅速解决数据丢失的问题；

- Redis可以使用所有机器的内存，变相扩展性能；

- 使Redis的计算能力通过简单地增加服务器得到成倍提升，Redis的网络带宽也会随着计算机和网卡的增加而成倍增长；

- Redis集群没有中心节点，不会因为某个节点成为整个集群的性能瓶颈;

- 异步处理数据，实现快速读写；

Redis 3.0以后，节点之间通过去中心化的方式提供了完整的**sharding(数据分片)**、**replication(复制机制、Cluster具备感知准备的能力)**、**failover故障转移解决方案**。

![img](./assets\wps12.jpg)

## **Redis Cluster 搭建**

Redis3.0及以上版本实现，集群中至少应该有**奇数个节点**，所以**至少有三个节点**，官方推荐**三主三从**的配置方式。**Redis 3.x和Redis4.x 搭建集群是需要手动安装ruby组件的，比较麻烦。**

2018年十月 Redis 发布了稳定版本的 5.0 版本，推出了各种新特性，其中一点是放弃 Ruby的集群方式，改为 使用 C语言编写的**redis-cli**的方式，是集群的构建方式复杂度大大降低。Redis cluster tutorial：<https://redis.io/topics/cluster-tutorial> 。

![img](./assets\wps1-1621903324441.jpg) 

基于Redis-5.0.8版本，在三台机器上搭建6个节点的Redis集群：**三主三从架构**。

## **环境准备**

关闭以前Redis主从复制和哨兵模式监控的所有服务，备注：如果以前没有安装过Redis服务，不用执行此步骤操作。

```sh
# ============= node1.it.cn、node2.it.cn和node3.it.cn =============
# 关闭哨兵服务SentinelServer
ps -ef|grep redis
kill -9 哨兵的进程ID

# 关闭Redis服务
redis-cli -h node1.it.cn -p 6379 SHUTDOWN
redis-cli -h node2.it.cn -p 6379 SHUTDOWN
redis-cli -h node3.it.cn -p 6379 SHUTDOWN
```

安装Redis编译环境：GCC和TCL。

`yum -y install gcc-c++ tcl`

## **上传和解压**

将Redis-5.0.8软件安装包上传至 **/export/software** 目录，并解压与安装。

```sh
# node01, 上传安装包至/export/softwares
cd /export/software
rz

# 解压
cd /export/software
chmod u+x redis-5.0.8.tar.gz 
tar -zxvf redis-5.0.8.tar.gz -C /export/server/
```

## **编译安装**

编译Redis 源码，并安装至【**/export/server/redis-5.0.8-bin**】目录。

```sh
# node01, 编译、安装、创建软连接
# 进入源码目录
cd /export/server/redis-5.0.8
# 编译
make
# 安装至指定目录
make PREFIX=/export/server/redis-5.0.8-bin install

# 创建安装目录软连接
cd /export/server
ln -s redis-5.0.8-bin redis
```

配置环境变量（如果以前安装过Redis，配置过环境变量，就不用配置）。

```sh
# 配置环境变量
vim /etc/profile
# ======================== 添加如下内容 ========================
    # REDIS HOME
export REDIS_HOME=/export/server/redis
export PATH=:$PATH:$REDIS_HOME/bin

# 执行生效
source /etc/profile
```

## **拷贝配置文件**

从Redis-5.0.8源码目录下拷贝配置文件：**redis.conf**至Redis 安装目录。

```sh
# ====================== node01 上操作 ======================
# 拷贝配置文件
cd /export/server/redis-5.0.8
cp redis.conf /export/server/redis
```

## **修改配置文件**

每台机器上启动2个Redis服务，一个**主节点服务：7001**，一个**从节点服务：7002**，如下图所示：

![img](./assets\wps2-1621903731105.jpg) 

在Redis安装目录下创建7001和7002目录，分别存储Redis服务配置文件、日志及数据文件。

```sh
# 创建目录：7001和7002
cd /export/server/redis
mkdir -p 7001 7002
```

拷贝配置文件：**redis.conf**至7001目录，并重命名为**redis_7001.conf**。

```sh
cd /export/server/redis
cp redis.conf 7001/redis_7001.conf
```

编辑配置文件：**redis_7001.conf**，内容如下：

```sh
cd /export/server/redis/7001
vim redis_7001.conf
## =========================== 修改内容说明如下 ===========================
## 69行，配置redis服务器接受链接的网卡
bind 0.0.0.0
## 88行，关闭保护模式
protected-mode no
## 92行，设置端口号
port 7001
## 136行，redis后台运行
daemonize yes
## 158行，Redis服务进程PID存储文件名称
pidfile /var/run/redis_7001.pid

## 171行，设置redis服务日志存储路径
logfile "/export/server/redis-5.0.8-bin/7001/log/redis.log"
## 263行，设置redis持久化数据存储目录
dir /export/server/redis-5.0.8-bin/7001/data/

## 699行，启动AOF方式持久化
appendonly yes

## 832行，启动Redis Cluster
cluster-enabled yes
## 840行，Redis服务配置保存文件名称
cluster-config-file nodes-7001.conf
## 847行，超时时间
cluster-node-timeout 15000
```



创建日志目录和数据目录：

```sh
mkdir -p /export/server/redis/7001/log
mkdir -p /export/server/redis/7001/data
```

配置7002端口号启动Redis服务，操作命令如下：

```sh
## 拷贝配置文件
cd /export/server/redis
cp 7001/redis_7001.conf 7002/redis_7002.conf

## 修改配置文件：redis_7002.conf
cd /export/server/redis/7002
vim redis_7002.conf
# 进入vim编辑之后，执行以下代码将7001全部替换成7002
:%s/7001/7002/g   # 表示:%s/old/new/g  g表示全部替换 

# 创建目录
mkdir -p /export/server/redis/7002/log
mkdir -p /export/server/redis/7002/data
```

## **发送安装包**

将node1.it.cn上配置好的Redis安装包，发送至node2.it.cn和node3.it.cn，每台机器运行2个Redis服务，端口号分别为7001和7002，具体命令如下：

```sh
# 发送安装包
cd /export/server
scp -r redis-5.0.8-bin root@node2.it.cn:$PWD
scp -r redis-5.0.8-bin root@node3.it.cn:$PWD

# 在node2和node3创建软连接
cd /export/server
ln -s redis-5.0.8-bin redis

# 配置环境变量
vim /etc/profile
# ======================== 添加如下内容 ========================
    # REDIS HOME
export REDIS_HOME=/export/server/redis
export PATH=:$PATH:$REDIS_HOME/bin
# 执行生效
source /etc/profile
```

##  **启动Redi服务**

在三台机器node01、node02和node03，分别启动6个Redis服务，命令如下：

```sh
# 启动7001端口Redis服务
/export/server/redis/bin/redis-server /export/server/redis/7001/redis_7001.conf

# 启动7002端口Redis服务
/export/server/redis/bin/redis-server /export/server/redis/7002/redis_7002.conf
```

Redis服务启动完成以后，查看如下：

![img](./assets\wps3-1621904129773.jpg) 

![img](./assets\wps4-1621904129800.jpg) 

![img](./assets\wps5-1621904129800.jpg)

## **启动集群**

Redis5.x版本之后，通过redis-cli客户端命令来进行创建集群，注意：**Redis对主机名解析不友好，使用IP地址**。

```sh
# 任意选择一台机器执行如下命令，创建集群
/export/server/redis/bin/redis-cli --cluster create 192.168.88.161:7001 192.168.88.162:7002 192.168.88.163:7001 192.168.88.161:7002 192.168.88.162:7001 192.168.88.163:7002 --cluster-replicas 1
```

**使用IP地址**。

启动集群日志信息如下：

```sh
>>> Performing hash slots allocation on 6 nodes...
## ====== 进行Slot槽范文划分 ======
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 – 16383
## ====== 主从分配，一主一从 ======
Adding replica 192.168.88.161:7001 to 192.168.88.161:7001
Adding replica 192.168.88.162:7002 to 192.168.88.162:7002
Adding replica 192.168.88.163:7002 to 192.168.88.163:7001
## ====== 主从节点配对信息 ======
M: 97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.100:7001
   slots:[0-5460] (5461 slots) master
M: 6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.101:7002
   slots:[5461-10922] (5462 slots) master
M: d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.102:7001
   slots:[10923-16383] (5461 slots) master
S: b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.100:7002
   replicates d9bf2ac8eec5637ed7ec50061419ff6b951eef0b
S: b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.101:7001
   replicates 97e1d381d59561e075ac813e2df7fed00114687e
S: c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.102:7002
   replicates 6e0353e92377a71d691a853152673a8774d11dc2
## ====== 是否同意上述划分，一般都是yes ======   
Can I set the above configuration? (type 'yes' to accept): yes
## ====== 节点配置更新，进入集群，主从节点，高可用 ======
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 192.168.88.100:7001)
M: 97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.100:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.102:7002
   slots: (0 slots) slave
   replicates 6e0353e92377a71d691a853152673a8774d11dc2
S: b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.101:7001
   slots: (0 slots) slave
   replicates 97e1d381d59561e075ac813e2df7fed00114687e
S: b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.100:7002
   slots: (0 slots) slave
   replicates d9bf2ac8eec5637ed7ec50061419ff6b951eef0b
M: 6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.101:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.102:7001
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

##  **测试集群**

在任意一台机器，使用**redis-cli**客户端命令连接Redis服务：

`redis-cli -c -p 7001`

输入命令：**cluster nodes**（查看集群信息）和**info replication**（主从信息）：

```sh
127.0.0.1:7001> cluster nodes
c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.163:7002@17002 slave 6e0353e92377a71d691a853152673a8774d11dc2 0 1598239340178 6 connected
b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.162:7001@17001 slave 97e1d381d59561e075ac813e2df7fed00114687e 0 1598239338161 5 connected
b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.161:7002@17002 slave d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 0 1598239337000 4 connected
6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.162:7002@17002 master - 0 1598239338000 2 connected 5461-10922
d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.163:7001@17001 master - 0 1598239339170 3 connected 10923-16383
97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.161:7001@17001 myself,master - 0 1598239338000 1 connected 0-5460 

127.0.0.1:7001> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.88.162,port=7001,state=online,offset=840,lag=1
master_replid:e669abc55325a3ea3f4273aeed92c3370568fb34
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:854
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:854
```

测试数据，设置Key值和查询Key的值。

```sh
# node1(192.168.88.163:7001)，redis-cli登录
127.0.0.1:7001> KEYS *
(empty list or set)
127.0.0.1:7001> set k1 v1
-> Redirected to slot [12706] located at 192.168.88.102:7001      # 自动定向到node03上主服务
OK
192.168.88.162:7001> set k2 v2
-> -> Redirected to slot [449] located at 192.168.88.100:7001		  # 自动定向到node02上主服务
OK
192.168.88.161:7001> set k3 v3
OK
192.168.88.161:7001> get k1
-> Redirected to slot [12706] located at 192.168.88.102:7001	 # 自动定向到node03上主服务
"v1"
192.168.88.162:7001> get k2
-> Redirected to slot [449] located at 192.168.88.100:7001		# 自动定向到node03上主服务
"v2"
192.168.88.163:7001> get k3
"v3"
192.168.88.163:7001> KEYS *
1) "k3"
2) "k2"
```

## **启动关闭集群**

编写脚本，方便启动和关闭Redis集群：**redis-cluster-start.sh**和**redis-cluster-stop.sh**。

进入Redis安装目录中bin目录，创建脚本文件

```sh
cd /export/server/redis-5.0.8-bin/bin/

touch redis-cluster-start.sh 
touch redis-cluster-stop.sh

# 给以执行权限
chmod u+x redis-cluster-start.sh 
chmod u+x redis-cluster-stop.sh
```

启动集群：**redis-cluster-start.sh**

```sh
vim redis-cluster-start.sh
#!/bin/bash

REDIS_HOME=/export/server/redis
# Start Server
## node1.itcast.cn
ssh node1.it.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7001/redis_7001.conf"
ssh node1.it.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7002/redis_7002.conf"
## node02
ssh node2.it.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7001/redis_7001.conf"
ssh node2.it.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7002/redis_7002.conf"
## node03
ssh node3.it.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7001/redis_7001.conf"
ssh node3.it.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7002/redis_7002.conf"
```

关闭集群：**redis-cluster-stop.sh**

```sh
vim redis-cluster-stop.sh
#!/bin/bash

REDIS_HOME=/export/server/redis
# Stop Server
## node01
${REDIS_HOME}/bin/redis-cli -h node1.it.cn -p 7001 SHUTDOWN
${REDIS_HOME}/bin/redis-cli -h node1.it.cn -p 7002 SHUTDOWN
## node02
${REDIS_HOME}/bin/redis-cli -h node2.it.cn -p 7001 SHUTDOWN
${REDIS_HOME}/bin/redis-cli -h node2.it.cn -p 7002 SHUTDOWN
## node03
${REDIS_HOME}/bin/redis-cli -h node3.it.cn -p 7001 SHUTDOWN
${REDIS_HOME}/bin/redis-cli -h node3.it.cn -p 7002 SHUTDOWN
```

## **主从切换**

测试Redis Cluster中主从服务切换，首先查看集群各个服务状态：

![1621904917595](./assets\1621904917595.png)

在node3上将7001端口Redis 服务关掉：**SHUTDOWN**

**../redis-cli -h node3.it.cn -p 7001 SHUTDOWN**

```sh
[root@node3 server]# ps aux | grep redis
root     112830  0.1  0.3 161164 14084 ?        Ssl  11:32   0:00 /export/server/redis/bin/redis-server 0.0.0.0:7002 [cluster]
root     112993  0.0  0.0 112812   972 pts/1    S+   11:36   0:00 grep --color=auto redis
```

再次查看集群状态信息：

![img](./assets\wps6-1621905001294.jpg) 

重新启动node03上7001端口Redis服务，查看集群状态信息

```sh
127.0.0.1:7001> cluster nodes
c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.163:7002@17002 slave 6e0353e92377a71d691a853152673a8774d11dc2 0 1598240006108 6 connected
6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.162:7002@17002 master - 0 1598240004000 2 connected 5461-10922
d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.163:7001@17001 master - 0 1598240007115 3 connected 10923-16383
97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.161:7001@17001 myself,master - 0 1598240004000 1 connected 0-5460
b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.162:7001@17001 slave 97e1d381d59561e075ac813e2df7fed00114687e 0 1598240006000 5 connected
b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.161:7002@17002 slave d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 0 1598240004093 4 connected
127.0.0.1:7001> redis-cli –h node3.itcast.cn –p 7001 SHUTDOWN

127.0.0.1:7001> cluster nodes
c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.163:7002@17002 slave 6e0353e92377a71d691a853152673a8774d11dc2 0 1598240223000 6 connected
6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.162:7002@17002 master - 0 1598240224000 2 connected 5461-10922
d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.102:7001@17001 master,fail - 1598240171708 1598240170299 3 disconnected
97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.161:7001@17001 myself,master - 0 1598240224000 1 connected 0-5460
b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.162:7001@17001 slave 97e1d381d59561e075ac813e2df7fed00114687e 0 1598240224734 5 connected
b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.161:7002@17002 master - 0 1598240223000 7 connected 10923-16383
```

## **Redis Cluster 管理**

**redis-cli**集群命令帮助：

```sh
[root@node01 ~]# redis-cli --cluster help
Cluster Manager Commands:
  create         host1:port1 ... hostN:portN
                 --cluster-replicas <arg>
  check          host:port
                 --cluster-search-multiple-owners
  info           host:port
  fix            host:port
                 --cluster-search-multiple-owners
  reshard        host:port
                 --cluster-from <arg>
                 --cluster-to <arg>
                 --cluster-slots <arg>
                 --cluster-yes
                 --cluster-timeout <arg>
                 --cluster-pipeline <arg>
                 --cluster-replace
  rebalance      host:port
                 --cluster-weight <node1=w1...nodeN=wN>
                 --cluster-use-empty-masters
                 --cluster-timeout <arg>
                 --cluster-simulate
                 --cluster-pipeline <arg>
                 --cluster-threshold <arg>
                 --cluster-replace
  add-node       new_host:new_port existing_host:existing_port
                 --cluster-slave
                 --cluster-master-id <arg>
  del-node       host:port node_id
  call           host:port command arg arg .. arg
  set-timeout    host:port milliseconds
  import         host:port
                 --cluster-from <arg>
                 --cluster-copy
                 --cluster-replace
  help           

For check, fix, reshard, del-node, set-timeout you can specify the host and port of any working node in the cluster.
```

在实际项目中可能由于Redis Cluster中节点宕机或者增加新节点，需要操作命令管理，主要操作如下。

![1621905211688](./assets\1621905211688.png)

#  **JavaAPI操作redis集群**

连接Redis集群，需要使用JedisCluster来获取Redis连接。

 

实现步骤：

1. 在cn.it.redis.api_test包下创建一个新的类：RedisClusterTest

2. 创建一个HashSet<HostAndPort>，用于保存集群中所有节点的机器名和端口号

3. 创建JedisPoolConfig对象，用于配置Redis连接池配置

4. 创建JedisCluster对象

5. 使用JedisCluster对象设置一个key，然后获取key对应的值

```java
public class RedisClusterTest {
    private JedisPoolConfig jedisPoolConfig;
    private JedisCluster jedisCluster;

    @BeforeTest
    public void beforeTest() {
        // 1. 创建一个HashSet<HostAndPort>，用于保存集群中所有节点的机器名和端口号
        HashSet<HostAndPort> hostAndPortSet = new HashSet<>();
        hostAndPortSet.add(new HostAndPort("node1.it.cn", 7001));
        hostAndPortSet.add(new HostAndPort("node1.it.cn", 7002));
        hostAndPortSet.add(new HostAndPort("node2.it.cn", 7001));
        hostAndPortSet.add(new HostAndPort("node2.it.cn", 7002));
        hostAndPortSet.add(new HostAndPort("node3.it.cn", 7001));
        hostAndPortSet.add(new HostAndPort("node3.it.cn", 7002));

        // 2. 创建JedisPoolConfig对象，用于配置Redis连接池配置
        jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxIdle(10);
        jedisPoolConfig.setMinIdle(5);
        jedisPoolConfig.setMaxWaitMillis(5000);
        jedisPoolConfig.setMaxTotal(50);

        // 3. 创建JedisCluster对象
        jedisCluster = new JedisCluster(hostAndPortSet);
    }

    @Test
    public void clusterOpTest() {
        // 设置一个key
        jedisCluster.set("pv", "1");

        // 获取key
        System.out.println(jedisCluster.get("pv"));
    }

    @AfterTest
    public void afterTest() {
        try {
            jedisCluster.close();
        } catch (IOException e) {
            System.out.println("关闭Cluster集群连接失败！");
            e.printStackTrace();
        }
    }
}
```

