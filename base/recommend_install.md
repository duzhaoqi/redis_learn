# redis的介绍与安装

[toc]

## 什么是redis

>Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。 --redis中文官网


说实话，我也是刚学，对redis很不了解，只知道他说是一个数据库，一个简单的数据库，可以支持事务，有五种数据类型，和各种操作。具体他能够干些啥东西我也不是很清楚，我会在学习过程中一一步一步理解，一步一步记录，来完成redis的学习。

## redis的安装

### 环境

我采用的是Ubuntu server 16的版本，redis是在[官网](https://redis.io/)下载的最新版，`4.0.6`
然后安装`make`和`gcc`，这是我们编译使用的工具，`sudo apt install make gcc`.


### 安装

我的环境：
`/opt/backup/` ：用于存放安装包，或解压后的包
`/opt/` ：软件安装的目录

将redis的压缩包放到`/opt/backup/`中，解压：
```
tar xf redis-4.0.6.tar.gz
```

创建安装目录：
```
sudo mkdir /opt/redis4
```

进入到解压后的redis包中，编译安装：
```
sudo make PREFIX=/opt/redis4 install
```
报错（详情看README.md）
更改编译命令：
```
sudo make MALLOC=libc PREFIX=/opt/redis4 install
```

创建一个配置文件的目录在`/opt/redis4`下：
```
mkdir conf
```

拷贝`redis.conf`到`/opt/redis4/conf/`


### 服务脚本

拷贝启动脚本到`/etc/init.d/`，并改名：
```
cp /opt/backup/redis-4.0.6/utils/redis_init_script /etc/init.d/redis4
```
修改参数：
```
... ...
EXEC=/opt/redis4/bin/redis-server #修改目录
CLIEXEC=/opt/redis4/bin/redis-cli #修改目录
... ...
CONF="/opt/redis4/conf/redis.conf" #修改目录

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF & #转到后台进程
        fi
... ...
```

### 持久化与日志

默认持久化是快照方式，直接在`/opt/redis4` 创建dump.rdb

我们进行修改，在redis.conf中：
将`dir ./`修改成`dir  /opt/redis4/data/` 

Redis 默认将日志输出到 /dev/null(即舍弃)，我们可以通过更改 redis.conf 文件里 GENERAL 大项下的 logfile 配置将日志保留到指定文件：
`logfile ""`
改为
`logfile /opt/redis4/log/redis.log`


### 环境变量设置

在`/etc/profile.d/`下编写脚本：`redis.sh`

内容：
```
export PATH=/opt/redis4/bin:$PATH
```

使其生效：`source /etc/profile.d/redis.sh`

### 测试

```
root@ub3:~# redis-cli ping
PONG
root@ub3:~# redis-cli
127.0.0.1:6379> set name redis
OK
127.0.0.1:6379> get name
"redis"
```

### 我的一些配置

```
bind *
timeout 300
daemonize yes
loglevel notice
logfile /opt/redis4/log/redis.log
rdbchecksum yes
dbfilename dump.rdb
dir /opt/redis4/database/
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
```
我的这些配置都是最基础的配置，以后更复杂的配置就在后续的笔记中增加吧。