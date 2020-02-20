# Memcached 基础

### 简介

Memcached是一个自由开源的，高性能，分布式内存对象缓存系统。（类似于Redis）

Memcached是一种基于内存的key-value存储，用来存储小块的任意数据（字符串、对象）。这些数据可以是数据库调用、API调用或者是页面渲染的结果。

一般的使用目的是，通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。

### 特征

memcached作为高速运行的分布式缓存服务器，具有以下的特点。

- 协议简单
- 基于libevent的事件处理（libevent是一个基于事件触发的网络库）
- 内置内存存储方式
- memcached不互相通信的分布式

### Memcached 运行

```bash
#作为前台程序运行 （事件处理型程序）
memcached/bin/memcached -p 11211 -m 64m -vv
#作为后台程序运行 （任务级程序）
memcached -p 11211 -m 64m -d
```

### Memcached 连接

```bash
telnet host port
```

telnet是远程登陆的命令，用于本地计算机和远程主机的连接

curl是url下载工具，c是client。

### 读写实例

```bash
telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1
Escape character is '^]'.
set foo 0 0 3		set保存命令
bar 
STORED          数据输入输出成功，则输出STORED
get foo					get读取命令
VALUE foo 0 3
bar
END
quit
```



## Memcached存储命令

### set命令（create and update）

```bash
set key flags exptime bytes 
value 
```

- **flags**：可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 
- **exptime**：在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）
- **bytes**：在缓存中存储的字节数

```bash
set runoob 0 900 9
memcached      ---value
STORED         ---结果

get runoob
VALUE runoob 0 9
memcached

END
```

### add（create，但不能update）

```bash
add key flags exptime bytes [noreply]
value
```

如果 add 的 key 已经存在，则不会更新数据(过期的 key 会更新)，之前的值将仍然保持相同，并且您将获得响应 **NOT_STORED**。

### replace(相当于纯update)

```bash
replace key flags exptime bytes [noreply]
value
```

### append命令

Memcached append 命令用于向已存在 **key(键)** 的 **value(数据值)** 后面追加数据 。

```shell
set runoob 0 900 9
memcached
STORED

append runoob 0 900 5
redis
STORED

get runoob 0 900 14
memcachedredis
END
```

### prepend命令

加在value的前面

### Memcached CAS 命令（用于实现分布式）

Memcached CAS（Check-And-Set 或 Compare-And-Swap） 命令用于执行一个"检查并设置"的操作

它仅在当前客户端最后一次取值后，该key 对应的值没有被其他客户端修改的情况下， 才能够将值写入。

检查是通过cas_token参数进行的， 这个参数是Memcach指定给已经存在的元素的一个唯一的64位值。

```bash
cas key flags exptime bytes unique_cas_token [noreply]
value
```

- **unique_cas_token**通过 gets 命令获取的一个唯一的64位值。
- **noreply（可选）**： 该参数告知服务器不需要返回数据

## Memcached 查找命令

### get命令

```bash
get key
```

### gets命令(用于获取带有CAS令牌存的value)

```bash
gets key
```

### delete命令

```shell
delete key
```



### MemCached和Redis的区别

|                | Redis                                                        | MemCached                                                    |
| :------------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
|      集群      | 支持                                                         | 支持                                                         |
|    数据类型    | String，List ，Set，Sorted Set，Hash                         | 简单数据类型                                                 |
|     持久性     | 支持数据持久存储，可以将内存中的数据保存在磁盘中             | 不支持数据持久存储                                           |
|   分布式存储   | 支持master-slave复制模式（主从模式）。原则：Master会将数据同步到slave，而slave不会将数据同步到master。Slave启动时会连接master来同步数据。**这是一个典型的分布式读写分离模型。我们可以利用master来插入数据，slave提供检索服务。这样可以有效减少单个机器的并发访问数量** | 使用一致性hash做分布式                                       |
| 数据一致性不同 | 单核，保证了数据按顺序提交，单进程单线程。redis利用队列技术将并发访问变为串行访问。 | 可以多核，需要使用CAS保证数据以整形。CAS（Check and Set）是一个确保并发一致性的机制，属于“乐观锁”；原理：拿版本号，操作，对比版本号，如果一致就操作，不一致就放弃操作 |
|   value大小    | 最大1GB                                                      | 1MB                                                          |
|      局限      | 数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此Redis适合的场景主要局限在较小数据量的高性能操作和运算上。 | 数据类型少，单个数据容量小                                   |

