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



## Redis持久化

### RDB:快照方式 

save:阻塞当前redis服务器

bgsave:redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束，阻塞只发生在fork阶段。

### AOF：追加方式 

一步是命令实时写入，第二步是对AOP文件重写。

命令写入=》追加到aof_buf =》同步到aof磁盘

aof重写是为了减少AOF文件大小，可以手动或者自动 bgrewriteaof





## 缓存雪崩、缓存穿透、缓存预热、缓存更新、缓存降级

