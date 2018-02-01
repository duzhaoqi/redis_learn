# 数据类型之lists


## BLPOP key [key ...] timeout

>起始版本：2.0.0  
>时间复杂度：O(1)

BLPOP 是阻塞式列表的弹出原语。 它是命令 LPOP 的阻塞版本，这是因为当给定列表内没有任何元素可供弹出的时候， 连接将被 BLPOP 命令阻塞。 当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。  

#### 非阻塞行为
当 BLPOP 被调用时，如果给定 key 内至少有一个非空列表，那么弹出遇到的第一个非空列表的头元素，并和被弹出元素所属的列表的名字 key 一起，组成结果返回给调用者。
当存在多个给定 key 时， BLPOP 按给定 key 参数排列的先后顺序，依次检查各个列表。 我们假设 key list1 不存在，而 list2 和 list3 都是非空列表。考虑以下的命令：  
`BLPOP list1 list2 list3 0`
BLPOP 保证返回一个存在于 list2 里的元素（因为它是从 list1 –> list2 –> list3 这个顺序查起的第一个非空列表）。  

#### 阻塞行为
如果所有给定 key 都不存在或包含空列表，那么 BLPOP 命令将阻塞连接， 直到有另一个客户端对给定的这些 key 的任意一个执行 LPUSH 或 RPUSH 命令为止。  
一旦有新的数据出现在其中一个列表里，那么这个命令会解除阻塞状态，并且返回 key 和弹出的元素值。  
当 BLPOP 命令引起客户端阻塞并且设置了一个非零的超时参数 timeout 的时候， 若经过了指定的 timeout 仍没有出现一个针对某一特定 key 的 push 操作，则客户端会解除阻塞状态并且返回一个 nil 的多组合值(multi-bulk value)。  
timeout 参数表示的是一个指定阻塞的最大秒数的整型值。当 timeout 为 0 是表示阻塞时间无限制。  
#### 什么 key 会先被处理？是什么客户端？什么元素？优先顺序细节。  
- 当客户端为多个 key 尝试阻塞的时候，若至少存在一个 key 拥有元素，那么返回的键值对(key/element pair)就是从左到右数第一个拥有一个或多个元素的key。 在这种情况下客户端不会被阻塞。比如对于这个例子 BLPOP key1 key2 key3 key4 0，假设 key2 和 key4 都非空， 那么就会返回 key2 里的一个元素。  
- 当多个客户端为同一个 key 阻塞的时候，第一个被处理的客户端是等待最长时间的那个（即第一个因为该key而阻塞的客户端）。 一旦一个客户端解除阻塞那么它就不会保持任何优先级，当它因为下一个 BLPOP 命令而再次被阻塞的时候，会在处理完那些 被同个 key 阻塞的客户端后才处理它（即从第一个被阻塞的处理到最后一个被阻塞的）。  
- 当一个客户端同时被多个 key 阻塞时，若多个 key 的元素同时可用（可能是因为事务或者某个Lua脚本向多个list添加元素）， 那么客户端会解除阻塞，并使用第一个接收到 push 操作的 key（假设它拥有足够的元素为我们的客户端服务，因为有可能存在其他客户端同样是被这个key阻塞着）。 从根本上来说，在执行完每个命令之后，Redis 会把一个所有 key 都获得数据并且至少使一个客户端阻塞了的 list 运行一次。 这个 list 按照新数据的接收时间进行整理，即是从第一个接收数据的 key 到最后一个。在处理每个 key 的时候，只要这个 key 里有元素， Redis就会对所有等待这个key的客户端按照“先进先出”(FIFO)的顺序进行服务。若这个 key 是空的，或者没有客户端在等待这个 key， 那么将会去处理下一个从之前的命令或事务或脚本中获得新数据的 key，如此等等。  
#### 当多个元素被 push 进入一个 list 时 BLPOP 的行为  
有时候一个 list 会在同一概念的命令的情况下接收到多个元素：
- 像 LPUSH mylist a b c 这样的可变 push 操作。
- 在对一个向同一个 list 进行多次 push 操作的 MULTI 块执行完 EXEC 语句后。
- 使用 Redis 2.6 或者更新的版本执行一个 Lua 脚本。

当多个元素被 push 进入一个被客户端阻塞着的 list 的时候，Redis 2.4 和 Redis 2.6 或者更新的版本所采取行为是不一样的。  
对于 Redis 2.6 来说，所采取的行为是先执行多个 push 命令，然后在执行了这个命令之后再去服务被阻塞的客户端。看看下面命令顺序。  
```
Client A:   BLPOP foo 0
Client B:   LPUSH foo a b c
```
如果上面的情况是发生在 Redis 2.6 或更高版本的服务器上，客户端 A 会接收到 c 元素，因为在 LPUSH 命令执行后，list 包含了 c,b,a 这三个元素，所以从左边取一个元素就会返回 c。  
相反，Redis 2.4 是以不同的方式工作的：客户端会在 push 操作的上下文中被服务，所以当 LPUSH foo a b c 开始往 list 中 push 第一个元素，它就被传送给客户端A，也就是客户端A会接收到 a（第一个被 push 的元素）。  
Redis 2.4的这种行为会在复制或者持续把数据存入AOF文件的时候引发很多问题，所以为了防止这些问题，很多更一般性的、并且在语义上更简单的行为被引入到 Redis 2.6 中。  
需要注意的是，一个Lua脚本或者一个 MULTI / EXEC 块可能会 push 一堆元素进入一个 list 后，再 删除这个 list。 在这种情况下，被阻塞的客户端完全不会被服务，并且只要在执行某个单一命令、事务或者脚本后 list 中没有出现元素，它就会被继续阻塞下去。  

#### 在一个 MULTI / EXEC 事务中的 BLPOP
BLPOP 可以用于流水线（pipeline，发送多个命令并且批量读取回复），特别是当它是流水线里的最后一个命令的时候，这种设定更加有意义。  
在一个 MULTI / EXEC 块里面使用 BLPOP 并没有很大意义，因为它要求整个服务器被阻塞以保证块执行时的原子性，这就阻止了其他客户端执行一个 push 操作。 因此，一个在 MULTI / EXEC 里面的 BLPOP 命令会在 list 为空的时候返回一个 nil 值，这跟超时(timeout)的时候发生的一样。  
如果你喜欢科幻小说，那么想象一下时间是以无限的速度在 MULTI / EXEC 块中流逝……  

#### 返回值
多批量回复(multi-bulk-reply): 具体来说:  
- 当没有元素的时候会弹出一个 nil 的多批量值，并且 timeout 过期。
- 当有元素弹出时会返回一个双元素的多批量值，其中第一个元素是弹出元素的 key，第二个元素是 value。

#### 例子
```
redis> DEL list1 list2
(integer) 0
redis> RPUSH list1 a b c
(integer) 3
redis> BLPOP list1 list2 0
1) "list1"
2) "a"
```

#### 可靠的队列
当 BLPOP 返回一个元素给客户端的时候，它也从 list 中把该元素移除。这意味着该元素就只存在于客户端的上下文中：如果客户端在处理这个返回元素的过程崩溃了，那么这个元素就永远丢失了。  
在一些我们希望是更可靠的消息传递系统中的应用上，这可能会导致一些问题。在这种时候，请查看 BRPOPLPUSH 命令，这是 BLPOP 的一个变形，它会在把返回元素传给客户端之前先把该元素加入到一个目标 list 中。  

#### 模式：事件提醒
用来阻塞 list 的操作有可能是不同的阻塞原语。 比如在某些应用里，你也许会为了等待新元素进入 Redis Set 而阻塞队列，直到有个新元素加入到 Set 中，这样就可以在不轮询的情况下获得元素。 这就要求要有一个 SPOP 的阻塞版本，而这事实上并不可用。但是我们可以通过阻塞 list 操作轻易完成这个任务。  
消费者会做的：  
```
LOOP forever
    WHILE SPOP(key) returns elements
        ... process elements ...
    END
    BRPOP helper_key
END
```

而在生产者这角度我们可以这样简单地使用：
```
MULTI
SADD key element
LPUSH helper_key x
EXEC
```

---

## BRPOP key [key ...] timeout

>起始版本：2.0.0  
>时间复杂度：O(1)


BRPOP 是一个阻塞的列表弹出原语。 它是 RPOP 的阻塞版本，因为这个命令会在给定list无法弹出任何元素的时候阻塞连接。 该命令会按照给出的 key 顺序查看 list，并在找到的第一个非空 list 的尾部弹出一个元素。  
请在 BLPOP 文档 中查看该命令的准确语义，因为 BRPOP 和 BLPOP 基本是完全一样的，除了它们一个是从尾部弹出元素，而另一个是从头部弹出元素。  

#### 返回值
多批量回复(multi-bulk-reply): 具体来说:  
当没有元素可以被弹出时返回一个 nil 的多批量值，并且 timeout 过期。  
当有元素弹出时会返回一个双元素的多批量值，其中第一个元素是弹出元素的 key，第二个元素是 value。  
#### 例子：
```
redis> DEL list1 list2
(integer) 0
redis> RPUSH list1 a b c
(integer) 3
redis> BRPOP list1 list2 0
1) "list1"
2) "c"
```

---



## BRPOPLPUSH source destination timeout

>起始版本：2.2.0  
>时间复杂度：O(1)

BRPOPLPUSH 是 RPOPLPUSH 的阻塞版本。 当 source 包含元素的时候，这个命令表现得跟 RPOPLPUSH 一模一样。 当 source 是空的时候，Redis将会阻塞这个连接，直到另一个客户端 push 元素进入或者达到 timeout 时限。 timeout 为 0 能用于无限期阻塞客户端。  
查看 RPOPLPUSH 以了解更多信息。  

#### 返回值
批量回复(bulk-reply): 元素从 source 中弹出来，并压入 destination 中。 如果达到 timeout 时限，会返回一个空的多批量回复(nil-reply)。  

#### 模式：可靠的队列
请参考RPOPLPUSH 命令文档。

#### 模式：循环列表
请参考RPOPLPUSH 命令文档。

---

## LINDEX key index

>起始版本：1.0.0   
>时间复杂度：O(N) where N is the number of elements to traverse to get to the element at index. This makes asking for the first or the last element of the list O(1).

返回列表里的元素的索引 index 存储在 key 里面。 下标是从0开始索引的，所以 0 是表示第一个元素， 1 表示第二个元素，并以此类推。 负数索引用于指定从列表尾部开始索引的元素。在这种方法下，-1 表示最后一个元素，-2 表示倒数第二个元素，并以此往前推。  
当 key 位置的值不是一个列表的时候，会返回一个error。  

#### 返回值
bulk-reply：请求的对应元素，或者当 index 超过范围的时候返回 nil。

#### 例子
```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LINDEX mylist 0
"Hello"
redis> LINDEX mylist -1
"World"
redis> LINDEX mylist 3
(nil)
redis> 
```

---

## LINSERT key BEFORE|AFTER pivot value

>起始版本：2.2.0  
>时间复杂度：O(N) where N is the number of elements to traverse before seeing the value pivot. This means that inserting somewhere on the left end on the list (head) can be considered O(1) and inserting somewhere on the right end (tail) is O(N).

把 value 插入存于 key 的列表中在基准值 pivot 的前面或后面。  
当 key 不存在时，这个list会被看作是空list，任何操作都不会发生。  
当 key 存在，但保存的不是一个list的时候，会返回error。  

#### 返回值
integer-reply: 经过插入操作后的list长度，或者当 pivot 值找不到的时候返回 -1。  

#### 例子
```
redis> RPUSH mylist "Hello"
(integer) 1
redis> RPUSH mylist "World"
(integer) 2
redis> LINSERT mylist BEFORE "World" "There"
(integer) 3
redis> LRANGE mylist 0 -1
1) "Hello"
2) "There"
3) "World"
redis> 
```

---

## LLEN key

>起始版本：1.0.0  
时间复杂度：O(1)

返回存储在 key 里的list的长度。 如果 key 不存在，那么就被看作是空list，并且返回长度为 0。 当存储在 key 里的值不是一个list的话，会返回error。  

#### 返回值

integer-reply: key对应的list的长度。

#### 例子
```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LLEN mylist
(integer) 2
redis> 
```

---

