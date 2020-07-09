## Redis能干嘛？

1，缓存

2，持久化（RDB，AOF）

3，发布订阅系统

4，地图分析

5，计数器

##  redisbenchmark:性能测试

redis-benchmark -h localhost -p 6379 -c 100 -n 100000

## 支持数据类型

字符串

hashmap

list

set

sorted set

地理空间

## 事务

事务要么同时成功，要么同时失败，原子性。

redis单命令保存原子性，但事务不保证原子性

一组命令的集合，一个事务中的所有命令都会被序列化，在事务执行中，顺序执行 

multi

//命令集

exec

编译时异常，代码有问题，exec会放弃整个命令集执行，都不会执行

运行时异常，其它命令是可以正常执行的

## 使用Watch当做redis乐观锁

watch money

multi

decrby money 10

incrby out 10

exec //在执行前，另一client修改了money,exec失败

需要unwatch,再次尝试

## jedis

使用java来操作redis，官方推荐java连着开发工具

```
Jedis jedis = new Jedis(host,port)
Transaction multi = jedis.multi();
try{
//命令集
multi.exec();
}catch(Exception e){
multi.discart();
}

jedis.close;
```

## SpringBoot整合

在springboot2.x之后，原来使用jedis被替换为lettuce?

jedis，采用了直连，多个线程操作的话，不安全，使用jedis pool连接池

lettuce:采用netty,实例可以在多个线程中进行共享，不存在线程不安全情况，可以减少线程数据



## Redis持久化

只做缓存，不需要持久化

### RDB（redis database）:快照方式 

save:阻塞当前redis服务器

bgsave:redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束，阻塞只发生在fork阶段。

### AOF(append only file)：追加方式 

将所有命令都记录下来，history(读操作不记录)，文件是appendonly.aof

一步是命令实时写入，第二步是对AOP文件重写。

命令写入=》追加到aof_buf =》同步到aof磁盘

aof重写是为了减少AOF文件大小，可以手动或者自动 bgrewriteaof



RDB性能好，有主从复制中，RDB就是备用的，AOF几乎不使用

## Redis发布订阅

subscribe订阅一个频道

publish发布消息到频道

redis是C语言实现的

## Redis主从复制

master(写请求)，slave(读请求)，读写分离

单台redis最大使用内存不要超过20G。

### 环境配置

只配置从库

```
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_replid:82aba7c89b7814ebcd332d993d2adb002dac6b43
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

```

1，根据不同config文件启动redis server,这时每个redis都是主节点

2，一般情况只用配置从机就好了 

redis-cli从节点， slaveof指定主机。 真实的配置在config里指定

从机只读，不可以写

主机断开，从机依旧连接到主机，默认不会选一台做为master

模式：一主多从，层层链路

### 篡位

当mater宕机了，需要选择一台slave作为master,有手动模式，和哨兵模式

1， 手动改为master, 使用slaveof no one作为master。一旦主机恢复，并不会自动成为master,仍需要手动改

2，哨兵：自动选举老大 redis-sentinel

1) 配置sentinel.conf

2) 启动哨兵进程

如果主机此时加来了，自动归为新的主机下。

## 缓存雪崩

缓存集中过期失效，或者redis宕机



## 缓存穿透（查不到）

redis缓存不存在，数据库也没有该数据,这样会并发查询该数据时，数据库压力大

解决方案： 1，布隆过滤器，2，缓存里放空对象

## 缓存预热

正式部署之前，

## 缓存更新

## 缓存降级

## 常用命令

### String

set,get,decr,incr,mget 等。

### Hash

hget,hset,hgetall 等。

### List

lpush,rpush,lpop,rpop,lrange等

### Set

sadd,spop,smembers,sunion 等

### Sorted Set

zadd,zrange,zrem,zcard等

## 集群

由于受到单机内存的限制，需要搭建集群

。。。



## 面试题收集

### 淘汰策略

**volatile-lru**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰

**volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰

**volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

**allkeys-lru**：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰

**allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰

**no-enviction**（驱逐）：禁止驱逐数据



