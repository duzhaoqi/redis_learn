# Redis配置详解




## redis是什么

redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sortedset --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。`区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。`

`Redis 是一个高性能的key-value数据库。` redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。

Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。

redis的官网地址，非常好记，是redis.io。
目前，Vmware在资助着redis项目的开发和维护。


## redis的特性

**1.** 完全居于内存，数据实时的读写内存，定时闪回到文件中。采用单线程,避免了不必要的上下文切换和竞争条件  
**2.** 支持高并发量，官方宣传支持10万级别的并发读写  
**3.** 支持持久存储，机器重启后的，重新加载模式，不会掉数据  
**4.** 海量数据存储，分布式系统支持，数据一致性保证，方便的集群节点添加/删除  
**5.** Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储。   
**6.** 灾难恢复--memcache挂掉后，数据不可恢复; redis数据丢失后可以通过aof恢复；  
**7.** 虚拟内存--Redis当物理内存用完时，可以将一些很久没用到的value 交换到磁盘；  
**8.** Redis支持数据的备份，即master-slave模式的数据备份；

----------


## 各功能模块
`File Event`: 处理文件事件，接受它们发来的命令请求（读事件），并将命令的执行结果返回给客户端（写事件）)  
`Time Event`: 时间事件(更新统计信息，清理过期数据，附属节点同步，定期持久化等)  
`AOF`: 命令日志的数据持久化  
`RDB`：实际的数据持久化  
`Lua Environment `: Lua 脚本的运行环境. 为了让 Lua 环境符合 Redis 脚本功能的需求，Redis 对 Lua 环境进行了一系列的修改， 包括添加函数库、更换随机函数、保护全局变量， 等等  
`Command table(命令表)`：在执行命令时，根据字符来查找相应命令的实现函数。  
`Share Objects（对象共享）`：  
- 主要存储常见的值：
 - **a.**各种命令常见的返回值，例如返回值OK、ERROR、WRONGTYPE等字符；
 - **b.** 小于 redis.h/REDIS_SHARED_INTEGERS (默认1000)的所有整数。通过预分配的一些常见的值对象，并在多个数据结构之间共享对象，程序避免了重复分配的麻烦。也就是说，这些常见的值在内存中只有一份。  

`Databases`：Redis数据库是真正存储数据的地方。当然，数据库本身也是存储在内存中的。



## redis配置文件

**redis的配置文件内容包括：**  
- 网络配置项  
- 基本配置项  
- 持久化相关配置  
- 复制相关的配置  
- 安全相关配置  
- Limit相关的配置  
- Cluster相关配置  
- SlowLog相关的配置  
- Advanced配置  

接下来，我将对其重要的选项进行说明，其实在配置文件中，有对各个选项的详细的英文解释。

### 网络配置项

`bind IP` 绑定的IP  
`port PORT` 端口号  
`protected-mode `是否开启保护模式，默认开启。要是配置里没有指定bind和密码。开启该参数后，redis只会本地进行访问，拒绝外部访问。  
`tcp-backlog` 定义了每一个端口最大的监听队列的长度  
`unixsocket` 是否开放Unix套接字接口   
`timeout`：客户端连接的空闲超时时长；  

----------


### 通用配置项

`daemonize`, 是否以守护进程启动  
`supervised`, 可以通过upstart和systemd管理Redis守护进程，这个参数是和具体的操作系统相关的  
`loglevel`, 日志级别  
`pidfile`, 进程号文件  
`logfile`, 日志文件  
`databases`：设定数据库数量，默认为16个，每个数据库的名字均为整数，从0开始编号，默认操作的数据库为0；  
切换数据库的方法：`SELECT <dbid>`

----------


### 快照配置

`save900 1` #900秒有一个key变化，就做一个保存  
`save300 10` #300秒有10个key变化，就做一个保存，这里需要和开发沟通  
`save60 10000` #60秒有10000个key变化就做一个保存  
`stop-writes-on-bgsave-error yes` #在出现错误的时候，是不是要停止保存  
`rdbcompression yes` #使用压缩rdb文件，rdb文件压缩使用LZF压缩算法，yes：压缩，但是需要一些cpu的消耗。no：不压缩，需要更多的磁盘空间  
`rdbchecksum yes `#是否校验rdb文件。从rdb格式的第五个版本开始，在rdb文件的末尾会带上CRC64的校验和。这跟有利于文件的容错性，但是在保存rdb文件的时候，会有大概10%的性能损耗，所以如果你追求高性能，可以关闭该配置。  
`dbfilenamedump.rdb` #rdb文件的名称  
`dir./ `数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录  

----------


### Limits相关的配置

`maxclients `设置能连上redis的最大客户端连接数量  
`maxmemory <bytes>` redis配置的最大内存容量。当内存满了，需要配合maxmemory-policy策略进
行处理。  
`maxmemory-policy noeviction`
淘汰策略：**volatile-lru, allkeys-lru, volatile-random, allkeys-random, volatile-ttl, noeviction**
内存容量超过maxmemory后的处理策略。
- ` volatile-lru`：利用LRU算法移除设置过过期时间的key。  
- ` volatile-random`：随机移除设置过过期时间的key。  
- ` volatile-ttl`：移除即将过期的key，根据最近过期时间来删除（辅以TTL）  
- ` allkeys-lru`：利用LRU算法移除任何key。  
- ` allkeys-random`：随机移除任何key。  
- ` noeviction`：不移除任何key，只是返回一个写错误。  

上面的这些驱逐策略，如果redis没有合适的key驱逐，对于写命令，还是会返回错误。redis将不再接收写请求，只接收get请求。写命令包括：set setnx  
`maxmemory-samples 5 `#淘汰算法运行时的采样样本数；  

----------


### 持久化配置
在**APPEND ONLY MODE**模块下， 默认redis使用的是rdb方式持久化，这种方式在许多应用中已经足够用了。但是redis如果中途宕机，会导致可能有几分钟的数据丢失，根据save来策略进行持久化，Append Only File是另一种持久化方式，可以提供更好的持久化特性。Redis会把每次写入的数据在接收后都写入 appendonly.aof 文件，每次启动时Redis都会先把这个文件的数据读入内存里，先忽略RDB文件。

`appendonly yes` #启动aof模式  
 aof文件名(default: "appendonly.aof")  
`appendfilename "appendonly.aof"` #据读入内存里，先忽略RDB文件  

`appendfsync`
Redis supports three different modes:
`no`：redis不执行主动同步操作，而是OS进行；  
`everysec`：每秒一次；  
`always`：每语句一次；  

如果Redis只是将客户端修改数据库的指令重现存储在AOF文件中，那么AOF文件的大小会不断的增加，因为AOF文件只是简单的重现存储了客户端的指令，而并没有进行合并。对于该问题最简单的处理方式，即当AOF文件满足一定条件时就对AOF进行

`rewrite`，rewrite是根据当前内存数据库中的数据进行遍历写到一个临时的AOF文件，待写完后替换掉原来的AOF文件即可。
redis重写会将多个key、value对集合来用一条命令表达。在rewrite期间的写操作会保存在内存的rewrite buffer中，rewrite成功后这些操作也会复制到临时文件中，在最后临时文件会代替AOF文件。

`no-appendfsync-on-rewrite no`在aof重写或者写入rdb文件的时候，会执行大量IO，此时对于everysec和always的aof模式来说，执行fsync会造成阻塞过长时间，noappendfsync-on-rewrite字段设置为默认设置为no。如果对延迟要求很高的应用，这个字段可以设置为yes，否则还是设置为no，这样对持久化特性来说这是更安全的选择。设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,等rewrite完成后再写入，默认为no，**建议yes**。Linux的默认fsync策略是30秒。可能丢失30秒数据。

`auto-aof-rewrite-percentage 100` aof自动重写配置。当目前aof文件大小超过上一次重写的aof文件大小的百分之多少进行重写，即当aof文件增长到一定大小的时候Redis能够调用bgrewrite aof对日志文件进行重写。当前AOF文件大小是上次日志重写得到AOF文件大小的二倍（设置为100）时，自动启动新的日志重写过程。

`auto-aof-rewrite-min-size 64mb` #设置允许重写的最小aof文件大小，避免了达到约定百分比但尺寸仍然很小的情况还要重写上述两个条件同时满足时，方会触发重写AOF；与上次aof文件大小相比，其增长量超过100%，且大小不少于64MB;

`aof-load-truncated yes` #指redis在恢复时,会忽略最后一条可能存在问题的指令。aof文件可能在尾部是不完整的，出现这种现象，可以选择让redis退出，或者导入尽可能多的数据。如果选择的是yes，当截断的aof文件被导入的时候，会自动发布一个log给客户端然后load。如果是no，用户必须手动redis-check-aof修复AOF文件才可以。

**`注意`**：持久机制本身不能取代备份；应该制订备份策略，对redis库定期备份；Redis服务器启动时用持久化的数据文件恢复数据，会优先使用AOF；


`RDB`：snapshotting, 二进制格式；按事先定制的策略，周期性地将数据从内存同步至磁盘；数据文件默认为dump.rdb；

客户端显式使用SAVE或BGSAVE命令来手动启动快照保存机制；  
`SAVE`：同步，即在主线程中保存快照，此时会阻塞所有客户端请求；  
`BGSAVE`：异步；backgroud  
`AOF`：Append Only File, fsync  
记录每次写操作至指定的文件尾部实现的持久化；当redis重启时，可通过重新执行文件中的命令在内存中重建出数据库；  
`BGREWRITEAOF`：AOF文件重写；  
不会读取正在使用AOF文件，而是通过将内存中的数据以命令的方式保存至临时文件中，完成之后替换原来的AOF文件；

----------


### SlowLog相关的配置
`slowlog-log-slower-than 10000`
当命令的执行超过了指定时间，单位是微秒；  
`slowlog-max-len 128`
慢查询日志长度。当一个新的命令被写进日志的时候，最老的那个记录会被删掉。  

### ADVANCED配置
`hash-max-ziplist-entries 512`  
`hash-max-ziplist-value 64`  
设置ziplist的键数量最大值，每个值的最大空间；  
