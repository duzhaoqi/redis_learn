#  基础设置

- 认证设置
- 删除数据库
- 事务机制
- 持久化


## 认证设置

在配置文件中找到`requirepass`，取消其前面的注释，后面跟的就是认证的字符串，在我们使用客户端登陆后，需要进行认证。  
```
# requirepass foobared
 requirepass du
```
认证方式如下：  
```
[root@ub1 ~]$./redis-cli 
127.0.0.1:6379> get name
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth du
OK
```
## 删除数据库
删除数据库的两个命令：  
```
fuashdb 删除单个
flushall 删除所有
```
## 事务机制

redis的事务机制和mysql的事物机制是不一样的，redis的事物机制是通过`multi`，`exec`，`watch`等命令实现事务机制，讲一个或多个命令归并为一个操作请求服务器按顺序执行的机制，如果在执行过程中出现问题，咋会回滚到事物开始之前的状态。

- multi：启动一个事务
- exec：执行事物

一次性将事务中的所有操作执行完成后返回给客户端；

```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name du
QUEUED
127.0.0.1:6379> set ip 11.11.11.11
QUEUED
127.0.0.1:6379> get name
QUEUED
127.0.0.1:6379> get ip
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) "du"
4) "11.11.11.11"
127.0.0.1:6379>
```
- watch ：乐观锁，在exec命令执行之前，用于监视指定的键，如果监视中的任意键的数据被修改，则服务器拒绝执行事物。  

### 示例：  
客户端A：  
```
127.0.0.1:6379> watch ip
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set ip 22.22.22.22
QUEUED
127.0.0.1:6379> get ip
QUEUED
127.0.0.1:6379>
```
客户端B:  
```
127.0.0.1:6379> set ip 33.33.33.33
OK
```
客户端A：  
```
127.0.0.1:6379> watch ip
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set ip 22.22.22.22
QUEUED
127.0.0.1:6379> get ip
QUEUED
127.0.0.1:6379>exec
(nil)
127.0.0.1:6379> get ip
"33.33.33.33"
```

---

## 持久化设置

redis就是一个工作在内存中的键值对数据库  

持久化的目的是为了保存数据，当redis出问题或者更新后可以继续使用数据内容。

### RDB方式：  
快照，是**二进制**格式内容，按照一定的规则将内存中的数据保存到磁盘上去，默认的文件名为dump.rdb  
客户端也可以显式的使用SAVE或者BGSAVE命令启动快照保存机制  

SAVE：同步方式，在主线程中保存快照，此时会阻塞所有客户端IO。  
BGSAVE：异步方式  

#### RDB的配置
```
save 900 1
save 300 10
save 60 10000

stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /opt/redis4/database/
```

### AOF方式：

记录每一次操作到指定文件的尾部，实现持久化，redis重启时，可以通过重新执行该文件中的命令重建数据库内容。    

BGREWRITEAOF：AOF文件的重写（防止文件过大）  
不会读取正在使用的AOF文件，而是通过内存中的数据以命令的方式保存在临时文件中，完成之后代替原来的AOF文件。  

#### AOF重写过程：  
1. redis主进程通过fork创建子进程
2. 子进程根据内存中的数据创建数据库重建命令序列于临时文件中；
3. 父进程续集接受client的请求，并会吧这些命令中的写操作继续追加到原来的AOF文件中去，额外的，这些新的写请求还会被放置到一个缓冲队列中。
4. 子进程重写完成，会通知父进程，父进程吧缓冲中的命令也写到临时文件中去；
5. 父进程用临时文件代替原来的AOF文件。

#### AOF的配置：
```
appendonly yes
appendfilename "appendonly.aof"
# appendfsync always
appendfsync everysec
# appendfsync no
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble no
```

注意：redis的持久化本身并不能代替备份，我们依旧需要制定备份策略，对redis数据库定期备份。  

RDB与AOF同时启用：  
1. BGSAVE与BGREWRITEAOF不会同时启用
2. 在redis启动用于恢复数据时，会优先使用AOF。

