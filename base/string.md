# 数据类型之String

>应用场景：String是最常用的一种数据类型，普通的key/ value 存储都可以归为此类.即可以完全实现目前 Memcached 的功能，并且效率更高。还可以享受Redis的定时持久化，操作日志及 Replication等功能。除了提供与 Memcached 一样的get、set、incr、decr 等操作外，Redis还提供了下面一些操作：

---
## SET key value   
>可用版本：1.0.0+  
>时间复杂度：O(1)

设置指定 key 的值，如果值存在，将会被覆盖，无论什么类型。任何与密钥关联的生存时间都将在成功的set操作中被丢弃。
#### 选项
从redis版本２．６．１开始，支持指定一下的选项：
- EX seconds --按秒设置过期时间
- PX milliseconds --按毫秒设置过期时间
- NX --仅在key不存在的时候设置值
- XX --仅在key存在的时候设置值

注意：自从SET命令的选项被SETNX,SETEX,PSETEX,代替以后，在将来的版本中可能这三个选项会被反对，并最终被丢弃。
#### 返回值
simple-string-reply:如果SET命令正常执行那么回返回OK，否则如果加了NX 或者 XX选项，但是没有设置条件。那么会返回nil。
#### 例子
```
redis> SET mykey "Hello"
OK
redis> GET mykey
"Hello"
redis>
```

---
## GET key  
>可用版本：1.0.0+  
>时间复杂度：O(1)

返回key的value。如果key不存在，返回特殊值nil。如果key的value不是string，就返回错误，因为GET只处理string类型的values。
#### 返回值
simple-string-reply:key对应的value，或者nil（key不存在时）

### 例子
```
redis> GET nonexisting
(nil)
redis> SET mykey "Hello"
OK
redis> GET mykey
"Hello"
redis>   
```
---
## SETRANGE key offset value
>起始版本：2.2.0  
时间复杂度：O(1)

这个命令的作用是覆盖key对应的string的一部分，从指定的offset处开始，覆盖value的长度。如果offset比当前key对应string还要长，那这个string后面就补0以达到offset。不存在的keys被认为是空字符串，所以这个命令可以确保key有一个足够大的字符串，能在offset处设置value。

注意，offset最大可以是229-1(536870911),因为redis字符串限制在512M大小。如果你需要超过这个大小，你可以用多个keys。

**警告**：当set最后一个字节并且key还没有一个字符串value或者其value是个比较小的字符串时，Redis需要立即分配所有内存，这有可能会导致服务阻塞一会。在一台2010MacBook Pro上，set536870911字节（分配512MB）需要～300ms，set134217728字节(分配128MB)需要～80ms，set33554432比特位（分配32MB）需要～30ms，set8388608比特（分配8MB）需要8ms。注意，一旦第一次内存分配完，后面对同一个key调用SETRANGE就不会预先得到内存分配。

#### 模式

正因为有了SETRANGE和类似功能的GETRANGE命令，你可以把Redis的字符串当成线性数组，随机访问只要O(1)复杂度。这在很多真实场景应用里非常快和高效。

#### 返回值

integer-reply：该命令修改后的字符串长度

#### 例子

基本使用方法:
```
redis> SET key1 "Hello World"
OK
redis> SETRANGE key1 6 "Redis"
(integer) 11
redis> GET key1
"Hello Redis"
redis> 
```
补0的例子:
```
redis> SETRANGE key2 6 "Redis"
(integer) 11
redis> GET key2
"\x00\x00\x00\x00\x00\x00Redis"
redis> 
```
---

## GETRANGE key start end 
>可用版本：2.4.0+  
>时间复杂度：O(1)

**警告**：这个命令是被改成GETRANGE的，在小于2.0的Redis版本中叫SUBSTR。 返回key对应的字符串value的子串，这个子串是由start和end位移决定的（两者都在string内）。可以用负的位移来表示从string尾部开始数的下标。所以-1就是最后一个字符，-2就是倒数第二个，以此类推。

这个函数处理超出范围的请求时，都把结果限制在string内。

#### 返回值

bulk-reply

#### 例子
```
redis> SET mykey "This is a string"
OK
redis> GETRANGE mykey 0 3
"This"
redis> GETRANGE mykey -3 -1
"ing"
redis> GETRANGE mykey 0 -1
"This is a string"
redis> GETRANGE mykey 10 100
"string"
redis>   
```
---

## GETSET key value  
>可用版本：1.0.0+  
>时间复杂度：O(1)

自动将key对应到value并且返回原来key对应的value。如果key存在但是对应的value不是字符串，就返回错误。  
**设计模式 **  
GETSET可以和INCR一起使用实现支持重置的计数功能。举个例子：每当有事件发生的时候，一段程序都会调用INCR给key mycounter加1，但是有时我们需要获取计数器的值，并且自动将其重置为0。这可以通过GETSET mycounter “0”来实现：
```
INCR mycounter
GETSET mycounter "0"
GET mycounter
```
#### 返回值
bulk-string-reply: 返回之前的旧值，如果之前Key不存在将返回nil。

#### 例子
```
redis> INCR mycounter
(integer) 1
redis> GETSET mycounter "0"
"1"
redis> GET mycounter
"0"
redis>   
```
---


## MSET key value [key value ...]  
>起始版本：1.0.1  
时间复杂度：O(N) where N is the number of keys to set.

对应给定的keys到他们相应的values上。MSET会用新的value替换已经存在的value，就像普通的SET命令一样。如果你不想覆盖已经存在的values，请参看命令MSETNX。
MSET是原子的，所以所有给定的keys是一次性set的。客户端不可能看到这种一部分keys被更新而另外的没有改变的情况。

#### 返回值

simple-string-reply：总是OK，因为MSET不会失败。

#### 例子
```
redis> MSET key1 "Hello" key2 "World"
OK
redis> GET key1
"Hello"
redis> GET key2
"World"
redis> 
```
---
## MGET key1 [key2..]  
>起始版本：1.0.0  
时间复杂度：O(N) where N is the number of keys to retrieve.

返回所有指定的key的value。对于每个不对应string或者不存在的key，都返回特殊值nil。正因为此，这个操作从来不会失败。

#### 返回值

array-reply: 指定的key对应的values的list

#### 例子
```
redis> SET key1 "Hello"
OK
redis> SET key2 "World"
OK
redis> MGET key1 key2 nonexisting
1) "Hello"
2) "World"
3) (nil)
redis>
```
---

## SETBIT key offset value  
>起始版本：2.2.0  
时间复杂度：O(1)

设置或者清空key的value(字符串)在offset处的bit值。

那个位置的bit要么被设置，要么被清空，这个由value（只能是0或者1）来决定。当key不存在的时候，就创建一个新的字符串value。要确保这个字符串大到在offset处有bit值。参数offset需要大于等于0，并且小于232(限制bitmap大小为512)。当key对应的字符串增大的时候，新增的部分bit值都是设置为0。

**警告**：当set最后一个bit(offset等于232-1)并且key还没有一个字符串value或者其value是个比较小的字符串时，Redis需要立即分配所有内存，这有可能会导致服务阻塞一会。在一台2010MacBook Pro上，offset为232-1（分配512MB）需要～300ms，offset为230-1(分配128MB)需要～80ms，offset为228-1（分配32MB）需要～30ms，offset为226-1（分配8MB）需要8ms。注意，一旦第一次内存分配完，后面对同一个key调用SETBIT就不会预先得到内存分配。

#### 返回值

integer-reply：在offset处原来的bit值

#### 例子
```
redis> SETBIT mykey 7 1
(integer) 0
redis> SETBIT mykey 7 0
(integer) 1
redis> GET mykey
"\x00"
redis> 
```
---

## GETBIT key offset  
>起始版本：2.2.0+  
>时间复杂度：O(1)

返回key对应的string在`offset`处的`bit`值 当`offset`超出了字符串长度的时候，这个字符串就被假定为由`0比特`填充的连续空间。当`key`不存在的时候，它就认为是一个空字符串，所以`offset`总是超出范围，然后`value`也被认为是由`0比特`填充的连续空间。到内存分配。

#### 返回值  
integer-reply：在offset处的bit值

#### 例子 
```
redis> SETBIT mykey 7 1
(integer) 0
redis> GETBIT mykey 0
(integer) 0
redis> GETBIT mykey 7
(integer) 1
redis> GETBIT mykey 100
(integer) 0
redis>   
```
---

## SETEX key seconds value  
>起始版本：2.0.0  
时间复杂度：O(1)

设置key对应字符串value，并且设置key在给定的seconds时间之后超时过期。这个命令等效于执行下面的命令：
```
SET mykey value
EXPIRE mykey seconds
```
SETEX是原子的，也可以通过把上面两个命令放到MULTI/EXEC块中执行的方式重现。相比连续执行上面两个命令，它更快，因为当Redis当做缓存使用时，这个操作更加常用。

#### 返回值  
simple-string-reply

#### 例子
```
redis> SETEX mykey 10 “Hello” 
OK 
redis> TTL mykey 
(integer) 10 
redis> GET mykey 
“Hello” 
redis>  
```
---

## SETNX key value  
>起始版本：1.0.0  
时间复杂度：O(1)

将key设置值为value，如果key不存在，这种情况下等同SET命令。 当key存在时，什么也不做。SETNX是”SET if Not eXists”的简写。

#### 返回值
Integer reply, 特定值:
- 1 如果key被设置了
- 0 如果key没有被设置

#### 例子
```
redis> SETNX mykey "Hello"
(integer) 1
redis> SETNX mykey "World"
(integer) 0
redis> GET mykey
"Hello"
redis> 
```

设计模式：使用!SETNX加锁  
  
**请注意：**

1. 不鼓励以下模式来实现the Redlock algorithm ，该算法实现起来有一些复杂，但是提供了更好的保证并且具有容错性。
2. 无论如何，我们保留旧的模式，因为肯定存在一些已实现的方法链接到该页面作为引用。而且，这是一个有趣的例子说明Redis命令能够被用来作为编程原语的。
3. 无论如何，即使假设一个单例的加锁原语，但是从 2.6.12 开始，可以创建一个更加简单的加锁原语，相当于使用SET命令来获取锁，并且用一个简单的 Lua 脚本来释放锁。该模式被记录在SET命令的页面中。

也就是说，SETNX能够被使用并且以前也在被使用去作为一个加锁原语。例如，获取键为foo的锁，客户端可以尝试一下操作：

```
SETNX lock.foo <current Unix time + lock timeout + 1>
```
如果客户端获得锁，`SETNX`返回1，那么将`lock.foo`键的`Unix`时间设置为不在被认为有效的时间。客户端随后会使用`DEL lock.foo`去释放该锁。  
如果SETNX返回0，那么该键已经被其他的客户端锁定。如果这是一个非阻塞的锁，才能立刻返回给调用者，或者尝试重新获取该锁，直到成功或者过期超时。

#### 处理死锁  
以上加锁算法存在一个问题：如果客户端出现故障，崩溃或者其他情况无法释放该锁会发生什么情况？这是能够检测到这种情况，因为该锁包含一个Unix时间戳，如果这样一个时间戳等于当前的Unix时间，该锁将不再有效。

当以下这种情况发生时，我们不能调用DEL来删除该锁，并且尝试执行一个SETNX，因为这里存在一个竞态条件，当多个客户端察觉到一个过期的锁并且都尝试去释放它。
- C1 和 C2 读lock.foo检查时间戳，因为他们执行完SETNX后都被返回了0，因为锁仍然被 C3 所持有，并且 C3 已经崩溃。
- C1 发送DEL lock.foo
- C1 发送SETNX lock.foo命令并且成功返回
- C2 发送DEL lock.foo
- C2 发送SETNX lock.foo命令并且成功返回
- **错误**：由于竞态条件导致 C1 和 C2 都获取到了锁  

幸运的是，可以使用以下的算法来避免这种情况，请看 C4 客户端所使用的好的算法：

- C4 发送SETNX lock.foo为了获得该锁
- 已经崩溃的客户端 C3 仍然持有该锁，所以Redis将会返回0给 C4
- C4 发送GET lock.foo检查该锁是否已经过期。如果没有过期，C4 客户端将会睡眠一会，并且从一开始进行重试操作
- 另一种情况，如果因为 lock.foo键的Unix时间小于当前的Unix时间而导致该锁已经过期，C4 会尝试执行以下的操作：

```
GETSET lock.foo <current Unix timestamp + lock timeout + 1>
```
- 由于GETSET 的语意，C4会检查已经过期的旧值是否仍然存储在lock.foo中。如果是的话，C4 会获得锁
- 如果另一个客户端，假如为 C5 ，比 C4 更快的通过GETSET操作获取到锁，那么 C4 执行GETSET操作会被返回一个不过期的时间戳。C4 将会从第一个步骤重新开始。请注意：即使 C4 在将来几秒设置该键，这也不是问题。  
为了使这种加锁算法更加的健壮，持有锁的客户端应该总是要检查是否超时，保证使用DEL释放锁之前不会过期，因为客户端故障的情况可能是复杂的，不止是崩溃，还会阻塞一段时间，阻止一些操作的执行，并且在阻塞恢复后尝试执行DEL（此时，该LOCK已经被其他客户端所持有）  
---
  
## STRLEN key  
>起始版本：2.2.0  
时间复杂度：O(1)

返回key的string类型value的长度。如果key对应的非string类型，就返回错误。

#### 返回值

integer-reply：key对应的字符串value的长度，或者0（key不存在）

#### 例子
```
redis> SET mykey "Hello world"
OK
redis> STRLEN mykey
(integer) 11
redis> STRLEN nonexisting
(integer) 0
redis> 
```
---
## MSETNX key value [key value ...]   
>起始版本：1.0.1  
时间复杂度：O(N) where N is the number of keys to set.

对应给定的keys到他们相应的values上。只要有一个key已经存在，MSETNX一个操作都不会执行。 由于这种特性，MSETNX可以实现要么所有的操作都成功，要么一个都不执行，这样可以用来设置不同的key，来表示一个唯一的对象的不同字段。

MSETNX是原子的，所以所有给定的keys是一次性set的。客户端不可能看到这种一部分keys被更新而另外的没有改变的情况。

#### 返回值

integer-reply，只有以下两种值：

1 如果所有的key被set
0 如果没有key被set(至少其中有一个key是存在的)
#### 例子
```
redis> MSETNX key1 "Hello" key2 "there"
(integer) 1
redis> MSETNX key2 "there" key3 "world"
(integer) 0
redis> MGET key1 key2 key3
1) "Hello"
2) "there"
3) (nil)
redis>   
```
---

## PSETEX key milliseconds value  
>起始版本：2.6.0  
时间复杂度：O(1)

这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。  

#### 例子
```
redis> PSETEX mykey 1000 "Hello"
OK
redis> PTTL mykey
(integer) 999
redis> GET mykey
"Hello"
redis>
```
---

## INCR key  
>起始版本：1.0.0  
时间复杂度：O(1)

对存储在指定key的数值执行原子的加1操作。  
如果指定的key不存在，那么在执行incr操作之前，会先将它的值设定为0。  
如果指定的key中存储的值不是字符串类型（fix：）或者存储的字符串类型不能表示为一个整数，  
那么执行这个命令时服务器会返回一个错误(eq:(error) ERR value is not an integer or out of range)。  
这个操作仅限于64位的有符号整型数据。

**注意**: 由于redis并没有一个明确的类型来表示整型数据，所以这个操作是一个字符串操作。

执行这个操作的时候，key对应存储的字符串被解析为10进制的**64位有符号整型数据。**

事实上，Redis 内部采用整数形式（Integer representation）来存储对应的整数值，所以对该类字符串值实际上是用整数保存，也就不存在存储整数的字符串表示（String representation）所带来的额外消耗。

#### 返回值
integer-reply:执行递增操作后key对应的值。

#### 例子
```
redis> SET mykey "10"
OK
redis> INCR mykey
(integer) 11
redis> GET mykey
"11"
redis> 
```

#### 实例：计数器
Redis的原子递增操作最常用的使用场景是计数器。  
使用思路是：每次有相关操作的时候，就向Redis服务器发送一个incr命令。  
例如这样一个场景：我们有一个web应用，我们想记录每个用户每天访问这个网站的次数。  
web应用只需要通过拼接用户id和代表当前时间的字符串作为key，每次用户访问这个页面的时候对这个key执行一下incr命令。  
这个场景可以有很多种扩展方法:

- 通过结合使用INCR和EXPIRE命令，可以实现一个只记录用户在指定间隔时间内的访问次数的计数器
- 客户端可以通过GETSET命令获取当前计数器的值并且重置为0
- 通过类似于DECR或者INCRBY等原子递增/递减的命令，可以根据用户的操作来增加或者减少某些值 比如在线游戏，需要对用户的游戏分数进行实时控制，分数可能增加也可能减少。

#### 实例: 限速器
限速器是一种可以限制某些操作执行速率的特殊场景。  
传统的例子就是限制某个公共api的请求数目。  
假设我们要解决如下问题：限制某个api每秒每个ip的请求次数不超过10次。  
我们可以通过incr命令来实现两种方法解决这个问题。  

#### 实例: 限速器 1
更加简单和直接的实现如下：
```
FUNCTION LIMIT_API_CALL(ip)
ts = CURRENT_UNIX_TIME()
keyname = ip+":"+ts
current = GET(keyname)
IF current != NULL AND current > 10 THEN
    ERROR "too many requests per second"
ELSE
    MULTI
        INCR(keyname,1)
        EXPIRE(keyname,10)
    EXEC
    PERFORM_API_CALL()
END
```
这种方法的基本点是每个ip每秒生成一个可以记录请求数的计数器。  
但是这些计数器每次递增的时候都设置了10秒的过期时间，这样在进入下一秒之后，redis会自动删除前一秒的计数器。  
注意上面伪代码中我们用到了MULTI和EXEC命令，将递增操作和设置过期时间的操作放在了一个事务中， 从而保证了两个操作的原子性。

#### 实例: 限速器 2
另外一个实现是对每个ip只用一个单独的计数器（不是每秒生成一个），但是需要注意避免竟态条件。 我们会对多种不同的变量进行测试。
```
FUNCTION LIMIT_API_CALL(ip):
current = GET(ip)
IF current != NULL AND current > 10 THEN
    ERROR "too many requests per second"
ELSE
    value = INCR(ip)
    IF value == 1 THEN
        EXPIRE(value,1)
    END
    PERFORM_API_CALL()
END
```
上述方法的思路是，从第一个请求开始设置过期时间为1秒。如果1秒内请求数超过了10个，那么会抛异常。  
否则，计数器会清零。  
上述代码中，可能会进入竞态条件，比如客户端在执行INCR之后，没有成功设置EXPIRE时间。这个ip的key 会造成内存泄漏，直到下次有同一个ip发送相同的请求过来。  
把上述INCR和EXPIRE命令写在lua脚本并执行EVAL命令可以避免上述问题（只有redis版本>＝2.6才可以使用）
```
local current
current = redis.call("incr",KEYS[1])
if tonumber(current) == 1 then
    redis.call("expire",KEYS[1],1)
end
```
还可以通过使用redis的list来解决上述问题避免进入竞态条件。  
实现代码更加复杂并且利用了一些redis的新的feature，可以记录当前请求的客户端ip地址。这个有没有好处 取决于应用程序本身。
```
FUNCTION LIMIT_API_CALL(ip)
current = LLEN(ip)
IF current > 10 THEN
    ERROR "too many requests per second"
ELSE
    IF EXISTS(ip) == FALSE
        MULTI
            RPUSH(ip,ip)
            EXPIRE(ip,1)
        EXEC
    ELSE
        RPUSHX(ip,ip)
    END
    PERFORM_API_CALL()
END
```

RPUSHX命令会往list中插入一个元素，如果key存在的话  
上述实现也可能会出现竞态，比如我们在执行EXISTS指令之后返回了false，但是另外一个客户端创建了这个key。  
后果就是我们会少记录一个请求。但是这种情况很少出现，所以我们的请求限速器还是能够运行良好的。  

---
## DECR key
>起始版本：1.0.0  
时间复杂度：O(1)

对key对应的数字做减1操作。如果key不存在，那么在操作之前，这个key对应的值会被置为0。如果key有一个错误类型的value或者是一个不能表示成数字的字符串，就返回错误。这个操作最大支持在64位有符号的整型数字。

查看命令INCR了解关于增减操作的额外信息。

#### 返回值
数字：减小之后的value

#### 例子
```
redis> SET mykey "10"
OK
redis> DECR mykey
(integer) 9
redis> SET mykey "234293482390480948029348230948"
OK
redis> DECR mykey
ERR value is not an integer or out of range
redis> 
```

---

## INCRBY key increment  
>始版本：1.0.0  
时间复杂度：O(1)

将key对应的数字加decrement。如果key不存在，操作之前，key就会被置为0。如果key的value类型错误或者是个不能表示成数字的字符串，就返回错误。这个操作最多支持64位有符号的正型数字。

查看命令INCR了解关于增减操作的额外信息。

#### 返回值

integer-reply： 增加之后的value值。

#### 例子
```
redis> SET mykey "10"
OK
redis> INCRBY mykey 5
(integer) 15
redis>   
```
---

## INCRBYFLOAT key increment  
>起始版本：2.6.0  
时间复杂度：O(1)

通过指定浮点数key来增长浮点数(存放于string中)的值. 当键不存在时,先将其值设为0再操作.下面任一情况都会返回错误:

key 包含非法值(不是一个string).  
当前的key或者相加后的值不能解析为一个双精度的浮点值.(超出精度范围了)  
如果操作命令成功, 相加后的值将替换原值存储在对应的键值上, 并以string的类型返回. string中已存的值或者相加参数可以任意选用指数符号,但相加计算的结果会以科学计数法的格式存储. 无论各计算的内部精度如何, 输出精度都固定为小数点后17位.

#### 返回值
Bulk-string-reply: 当前key增加increment后的值。

#### 例子
```
redis> SET mykey 10.50
OK
redis> INCRBYFLOAT mykey 0.1
"10.6"
redis> SET mykey 5.0e3
OK
redis> INCRBYFLOAT mykey 2.0e2
"5200"
redis>
```
#### 执行细节
该命令总是衍生为一个链接复制以及追加文件的set操作 , 所以底层浮点数的实现的差异并不是造成不一致的源头 

---

## DECRBY key decrement  
>起始版本：1.0.0  
时间复杂度：O(1)

将key对应的数字减decrement。如果key不存在，操作之前，key就会被置为0。如果key的value类型错误或者是个不能表示成数字的字符串，就返回错误。这个操作最多支持64位有符号的正型数字。

查看命令INCR了解关于增减操作的额外信息。似。

#### 返回值
返回一个数字：减少之后的value值。

#### 例子
```
redis> SET mykey "10"
OK
redis> DECRBY mykey 5
(integer) 5
redis> 
```

## APPEND key value  
>起始版本：2.0.0  
时间复杂度：O(1)

如果 key 已经存在，并且值为字符串，那么这个命令会把 value 追加到原来值（value）的结尾。 如果 key 不存在，那么它将首先创建一个空字符串的key，再执行追加操作，这种情况 APPEND 将类似于 SET 操作。

#### 返回值
Integer reply：返回append后字符串值（value）的长度。

#### 例子
```
redis> EXISTS mykey
(integer) 0
redis> APPEND mykey "Hello"
(integer) 5
redis> APPEND mykey " World"
(integer) 11
redis> GET mykey
"Hello World"
redis>
```
#### 模式：节拍序列(Time series)
APPEND 命令可以用来连接一系列固定长度的样例,与使用列表相比这样更加紧凑. 通常会用来记录节拍序列. 每收到一个新的节拍样例就可以这样记录:

```
APPEND timeseries "fixed-size sample"
```
在节拍序列里, 可以很容易地访问序列中的每个元素:

- STRLEN 可以用来计算样例个数.
- GETRANGE 允许随机访问序列中的各个元素. 如果序列中有明确的节拍信息, 在Redis 2.6中就可以使用GETRANGE配合Lua脚本来实现一个二分查找算法.
- SETRANGE 可以用来覆写已有的节拍序列.

该模式的局限在于只能做追加操作. Redis目前缺少剪裁字符串的命令, 所以无法方便地把序列剪裁成指定的尺寸. 但是, 节拍序列在空间占用上效率极好.

**小贴士**: 在键值中组合Unix时间戳, 可以在构建一系列相关键值时缩短键值长度,更优雅地分配Redis实例.

使用定长字符串进行温度采样的例子(在实际使用时,采用二进制格式会更好).
```
redis> APPEND ts "0043"
(integer) 4
redis> APPEND ts "0035"
(integer) 8
redis> GETRANGE ts 0 3
"0043"
redis> GETRANGE ts 4 7
"0035"
redis>  
```