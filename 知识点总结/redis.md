# redis

redis默认有16个数据库，默认使用第0个

redis单线程，6.0之后支持多线程

# Redis数据类型

```bash
keys * #查看所有key
set name zhangsan #set key
EXISTS name #判断当前key是否存在
move name 1 #移动当前的key到数据库1
EXPIRE name 10 #设置key的过期时间，单位为秒
ttl name #查看当前key的剩余时间
type name #查看当前key的类型
```



## 五大基本数据类型

### String

```bash
set key1 v1
APPEND key1 "hello" #追加字符串，如果当前key不存在，就相当于set key
STRLEN key1 #获取字符串长度
get key1    #获得值
################
set views 0 #初始浏览量为0
incr views # 自增1 浏览量+1
decr views # 自减1 浏览量-1
INCRBY views 10 #设置自增步长
DECRBY views 10 #设置自减步长
#################
GETRANGE key1 0 3  #截取字符串[0,3]
SETRANGE key1 1 xx #替换指定位置开始的字符串
#################
# setex (set with expire) #设置过期时间
# setnx (set if not exist)#不存在再设置

setex key3 30 "hello" # 设置key3的值为hello，30秒后过期
setnx mykey "redis"   # 如果mykey不存在，创建mykey
setnx mykey "MongoDB" # 如果mykey存在，创建失败

##################
mset k1 v1 k2 v2 k3 v3  #同时设置多个值
mget k1 k2 k3  #同时获取多个值
msetnx k1 v1 k4 v4 #操作会失败，因为msetnx是一个原子性操作，要么一起成功，要么一起失败
```

### List

所有的list命令都是用l开头的

```bash
LPUSH list one #将一个值或多个值插入列表头部（左）
RPUSH list two #将一个值或多个值插入列表尾部（右）
LRANGE list 0 -1  #获取list中的值
LRANGE list 0 1   #通过区间获取具体的值
#####################
Lpop list #移除list的第一个元素
Rpop list #移除list的最后一个元素
#####################
lindex list 1 #通过下标获得list中的某一个值
Llen list #返回列表长度
Lrem list 1 one # 移除list集合中指定个数的value，精确匹配
ltrim mylist 1 2 #通过下标截取指定长度，但list会被改变，截断了只剩下截取的元素
rpoolpush mylist myotherlist #移除列表的最后一个元素，将他移动到新的列表中
lset list 0 item # 更新当前下标的值
```

### set

```bash
sadd myset "hello"   # set集合中添加元素
SMEMBERS myset       # 查看指定set的所有值
SISMEMBER myset hello #判断某一个值是不是在set集合中
scard myset          # 获取set集合中的内容元素个数
srem myset hello     #移除set集合中的指定元素
SRANDMEMBER myset    #随机抽选出一个元素
SRANDMEMBER myset 2  #随机选出指定个数元素
spop myset           #随机删除一个元素
smove myset myset2 "hello" #移动指定元素（myset到myset2）
SDIFF key1 key2     #差集
SINTER key1 key2    #交集
SUNION key1 key2    #并集
```

### Hash

可以看作是一个Map集合，Map<key,Map<key,value>>

```bash
hset myhash field1 hello  #设置一个key-value
hget myhash field1  #获取一个字段值
hmset myhash field1 hello field2 world  #设置多个key-value
hmget myhash field1 field2 #获取多个字段值
hgetall myhash #获取全部数据

```



### Zset

## 三种特殊数据类型

# 事务

Redis事务本质：一组命令的集合，一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行。

Redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

==Redis事务没有隔离级别的概念==

所有的命令在事务中，并没有直接被执行，只有发起执行命令的时候才会执行。

==Redis单条命令保证原子性，但是事务不保证原子性==

## Redis事务三个阶段

+ 开始事务（multi）

+ 命令入队 (...)

+ 执行事务 (exec) 

## Redis事务使用实例

> 正常执行

> 放弃事务 discard

> 若在事务队列中存在命令性错误（类似于java编译性错误），则执行EXEC命令时，所有命令都不会执行

> 若在事务队列中存在语法性错误（类似于java的1/0的运行时异常），则执行EXEC命令时，其他正确命令会被执行，错误命令抛出异常。

## Redis监控测试

悲观锁：

+ 很悲观，认为什么时候都会出现问题，无论什么都会加锁。

乐观锁：

+ 很乐观，认为什么时候都不会出现问题，所以不会上锁。更新数据的时候去判断一下，在此期间是否有人修改过这个数据

+ 获取version

+ 更新的时候比较version

> 正常执行

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money #监视money对象
OK
127.0.0.1:6379> multi #事务正常结束，数据期间没有发生变动，这个时候就正常执行成功！
OK
127.0.0.1:6379> decrby money 20
QUEUED
127.0.0.1:6379> incrby out 20
QUEUED
127.0.0.1:6379> exec
1) (integer) 80
2) (integer) 20
```

> 测试多线程修改值，使用watch可以当做redis的乐观锁操作！

+ 线程1

```bash
127.0.0.1:6379> watch money  #监视money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 10
QUEUED
127.0.0.1:6379> incrby out 10
QUEUED
127.0.0.1:6379> exec  #执行之前，另外一个线程，修改了我们的值，这个时候，事务执行会失败
(nil)
```

+ 线程2

```bash
127.0.0.1:6379> get money
"80"
127.0.0.1:6379> set money 1000
OK
```

修改失败后，通过unwatch进行解锁，再获取最新值进行监视即可

# Jedis

> Jedis是Redis官方推荐的java连接开发工具，使用java操作redis

# SpringBoot整合

SrpingBoot2.x之后，原来使用的jedis被替换为了lettuce

jedis：采用直连，多线程操作不安全，如果要避免不安全，使用jedis pool连接池。（更像BIO)

lettuce：采用netty，实例可以在多个线程中进行共享，不存在线程不安全的情况，可以减少线程数据。(更像NIO)

1、导入依赖

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
```



2、添加配置

```properties
spring.redis.host=172.16.96.67
spring.redis.port=6379
```



3、测试

```java
package com.redis;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.core.RedisTemplate;

@SpringBootTest
class DemoApplicationTests {

	@Autowired
	private RedisTemplate redisTemplate;

	@Test
	void contextLoads() {
		//redisTemplate   操作不同数据类型
		//opsForValue	  操作字符串 类似String
		//opsForList	  操作List   类似List
		//opsForSet
		//opsForHash
		//opsForZSet
		//opsForGeo
		//opsForHyperLogLog

		//事务和基本的CRUD都可以直接通过redisTemplate操作
		//获取redis的连接对象
		//		RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
		//		connection.flushDb();
		//		connection.flushAll();
		redisTemplate.opsForValue().set("mykey","myvalue");
		System.out.println(redisTemplate.opsForValue().get("mykey"));

	}

}

```

# Redis持久化

Redis是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么服务器进程一旦退出，服务器的数据库状态也会消失，所以Redis提供了持久化功能。

## RDB(Redis DataBase)

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是Snapshot快照，它恢复时是将快照文件直接读到内存里。

Redis会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加高效。RDB的缺点就是最后一次持久化后的数据可能丢失，我们默认的就是RDB，一般情况下不需要修改这个配置！

==rdb保存的文件是dump.rdb==

### 触发机制

* save规则满足的情况下，会自动触发rdb规则
* 执行flushall命令
* 退出redis

触发规则后自动生成一个dump.rdb

### 如何恢复rdb文件

* 将rdb文件放在我们redis启动目录就可以，redis启动的时候会自动检查dump.rdb恢复其中的数据

* 查看需要存放的位置

```bash
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/redis/bin"  # 如果在这个目录下存在dump.rdb文件，启动就会自动恢复其中的数据
```

**优点：**

1、适合大规模的数据恢复

2、对数据的完整性要求不高

**缺点：**

1、需要一定的时间间隔进行操作。如果redis意外宕机，这个最后一次修改数据就没有了。

2、fork进程的时候，会占用一定的内存空间

## AOF(Append Only File)

将我们的所有命令都记录下来，恢复的时候就把文件中的命令全部执行一遍

以日志的形式来记录每个写操作，将Redis执行过的所有命令记录下来(读操作不记录)，只许追加文件内容但不能改写，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次完成数据的恢复工作。

==AOF保存的是appendonly.aof文件==

AOF默认是不开启的，需要修改redis.conf相关配置，将appendonly改为yes就可以开启AOF

如果AOF文件有问题，redis是无法启动的，需要利用`redis-check-aof --fix`进行修复

### AOF三种配置选择

```bash
appendonly no #默认是不开启aof的，默认使用rdb方式进行持久化，在大部分情况下，rdb完全够用
appendfilename "appendonly.aof"#aof默认文件名字
# appendfsync always #每次修改都会sync同步，消耗性能
appendfsync everysec #每秒执行一次sync，可能会丢失这1s的数据
# appendfsync no #不执行sync，这个时候操作系统会自己同步数据，速度最快
```

特点：

1、每次修改都同步，文件完整性较好

2、每秒同步一次，可能会丢失一秒的数据

3、从不同步效率最高

优点：



缺点：

1、相对与数据文件来说，aof远大于rdb，修复的速度也比rdb慢

2、aof运行效率也要比rdb慢，redis默认的配置就是rdb持久化 

# Redis发布订阅

redis发布订阅(pub/sub)是一种==消息通信模式==：发送者(pub)发送消息，订阅者(sub)接收消息。微博、微信、关注系统！

Redis客户端可以订阅任意数量的频道

> 测试

订阅端：

```bash
127.0.0.1:6379> subscribe piemon #订阅一个频道piemon
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "piemon"
3) (integer) 1
#等待读取推送的信息
1) "message" #消息
2) "piemon"  #频道
3) "hello"   #具体内容

1) "message"
2) "piemon"
3) "hello,redis"
```

> 原理



发送端：

```bash
127.0.0.1:6379> publish piemon hello #发布者发布消息到频道
(integer) 1
127.0.0.1:6379> publish piemon "hello,redis"#发布者发布消息到频道
(integer) 1

```



# Redis主从复制



# Redis缓存穿透和雪崩

Redis缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，也带来了一些问题。其中，最要害的问题就是数据的一致性问题，从严格意义上说，这个问题无解。如果对数据的一致性要求很高，就不能使用缓存。

另外的的典型问题就是缓存穿透、缓存雪崩和缓存击穿。目前，业界也有比较流行的解决方案。

## 缓存穿透

### 概念

缓存穿透的概念很简单，用户想要查询一个数据，发现redis内存数据库中没有，也就是缓存没有命中，于是向持久层数据库查询，发现也没有，于是本次查询失败。当用户很多时，缓存都没有命中，于是都去请求了持久层数据库，这会给持久层数据库造成很大的压力，这时就相当于出现了缓存穿透。

### 解决方案

> 布隆过滤器

布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力。

![img](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1603080996319&di=58f4480aec391b84d8e33989e58b79f9&imgtype=0&src=http%3A%2F%2Fwww.uml.org.cn%2Fsjjm%2Fimages%2F20200506213.png.jpg)

这种方法会存在两个问题：

1、如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键；

2、即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。

## 缓存击穿

### 概念

缓存击穿是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库。

当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且回写缓存，会导致数据库瞬间压力过大。

### 解决方案

> 设置热点数据永不过期

从缓存层面看，没有设置过期时间，所以不会出现热点key过期后产生的问题

> 加互斥锁

分布式锁：使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验很大。

## 缓存雪崩

### 概念

缓存雪崩是指在某一个时间段，缓存集中过期失效，比如Redis宕机

### 解决方案

> Redis高可用

搭建Redis集群

> 限流降级

在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

> 数据预热

数据预热就是在正式部署之前，先把可能的数据先预热访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间尽量均匀。