# 数据结构之sets



相关命令

- SADD
- SCARD
- SDIFF
- SDIFFSTORE
- SINTER
- SINTERSTORE
- SISMEMBER
- SMEMBERS
- SMOVE
- SPOP
- SRANDMEMBER
- SREM
- SSCAN
- SUNION
- SUNIONSTORE


## SADD key member [member ...]

>起始版本：1.0.0  
时间复杂度：O(N) where N is the number of members to be added.

添加一个或多个指定的member元素到集合的 key中.指定的一个或者多个元素member 如果已经在集合key中存在则忽略.如果集合key 不存在，则新建集合key,并添加member元素到集合key中.  
如果key 的类型不是集合则返回错误.

#### 返回值
integer-reply:返回新成功添加到集合里元素的数量，不包括已经存在于集合中的元素.
#### 历史
    = 2.4: 接受多个member 参数. Redis 2.4 以前的版本每次只能添加一个member元素.
#### 例子
```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SADD myset "World"
(integer) 0
redis> SMEMBERS myset
1) "World"
2) "Hello"
redis> 
```

---

## SCARD key

>起始版本：1.0.0  
时间复杂度：O(1)

返回集合存储的key的基数 (集合元素的数量).

#### 返回值
integer-reply: 集合的基数(元素的数量),如果key不存在,则返回 0.

#### 举例
```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SCARD myset
(integer) 2
redis> 
```

--- 

## SDIFF key [key ...]

>起始版本：1.0.0  
时间复杂度：O(N) where N is the total number of elements in all given sets.

返回一个集合与给定集合的差集的元素.

举例:
```
key1 = {a,b,c,d}
key2 = {c}
key3 = {a,c,e}
SDIFF key1 key2 key3 = {b,d}
```
不存在的key认为是空集.
#### 返回值
array-reply:结果集的元素.

```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SDIFF key1 key2
1) "a"
2) "b"
redis> 
```

---

## SDIFFSTORE destination key [key ...]

>起始版本：1.0.0  
时间复杂度：O(N) where N is the total number of elements in all given sets.  

该命令类似于 SDIFF, 不同之处在于该命令不返回结果集，而是将结果存放在destination集合中.  
如果destination已经存在, 则将其覆盖重写.  

#### 返回值
integer-reply: 结果集元素的个数.

#### 例子
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SDIFFSTORE key key1 key2
(integer) 2
redis> SMEMBERS key
1) "b"
2) "a"
redis> 
```

---

## SINTER key [key ...]

>起始版本：1.0.0  
时间复杂度：O(N*M) worst case where N is the cardinality of the smallest set and M is the number of sets.  

返回指定所有的集合的成员的交集.  
例如:   
```
key1 = {a,b,c,d}
key2 = {c}
key3 = {a,c,e}
SINTER key1 key2 key3 = {c}
```
如果key不存在则被认为是一个空的集合,当给定的集合为空的时候,结果也为空.(一个集合为空，结果一直为空).  

#### 返回值
array-reply: 结果集成员的列表.

#### 例子
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SINTER key1 key2
1) "c"
redis> 
```

---

## SINTERSTORE destination key [key ...]

>起始版本：1.0.0  
时间复杂度：O(N*M) worst case where N is the cardinality of the smallest set and M is the number of sets.  

这个命令与SINTER命令类似, 但是它并不是直接返回结果集,而是将结果保存在 destination集合中.  
如果destination 集合存在, 则会被重写.  

#### 返回值
integer-reply: 结果集中成员的个数.

#### 例子
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SINTERSTORE key key1 key2
(integer) 1
redis> SMEMBERS key
1) "c"
redis> 
```

---

## SISMEMBER key member  

>起始版本：1.0.0  
时间复杂度：O(1)

返回成员 member 是否是存储的集合 key的成员.  

#### 返回值
integer-reply,详细说明:  
如果member元素是集合key的成员，则返回1  
如果member元素不是key的成员，或者集合key不存在，则返回0  

#### 举例
```
redis> SADD myset "one"
(integer) 1
redis> SISMEMBER myset "one"
(integer) 1
redis> SISMEMBER myset "two"
(integer) 0
redis> 
```

---

## SMEMBERS key

>起始版本：1.0.0  
时间复杂度：O(N) where N is the set cardinality.

返回key集合所有的元素.  
该命令的作用与使用一个参数的SINTER 命令作用相同.  

#### 返回值
array-reply:集合中的所有元素.

#### 举例
```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SMEMBERS myset
1) "World"
2) "Hello"
redis> 
```

---

## SMOVE source destination member

>起始版本：1.0.0  
时间复杂度：O(1)

将member从source集合移动到destination集合中. 对于其他的客户端,在特定的时间元素将会作为source或者destination集合的成员出现.  
如果source 集合不存在或者不包含指定的元素,这smove命令不执行任何操作并且返回0.否则对象将会从source集合中移除，并添加到destination集合中去，如果destination集合已经存在该元素，则smove命令仅将该元素充source集合中移除. 如果source 和destination不是集合类型,则返回错误.  

#### 返回值
integer-reply  
如果该元素成功移除,返回1  
如果该元素不是 source集合成员,无任何操作,则返回0.  

#### 举例
```
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myotherset "three"
(integer) 1
redis> SMOVE myset myotherset "two"
(integer) 1
redis> SMEMBERS myset
1) "one"
redis> SMEMBERS myotherset
1) "three"
2) "two"
redis> 
```

---

## SPOP key [count]

>起始版本：1.0.0  
时间复杂度： O（1）  

从设置值存储中移除并返回一个或多个随机元素key。  
这个操作类似于SRANDMEMBER返回一个或多个随机元素但不删除它的集合。  
该count参数将在更高版本中提供，并且在2.6,2.8,3.0中不可用  

#### 返回
散装串答复：被删除的元素，或者nil当key不存在。

#### 例子
```
SADD myset "one"
SADD myset "two"
SADD myset "three"
SPOP myset
SMEMBERS myset
SADD myset "four"
SADD myset "five"
SPOP myset 3
SMEMBERS myset
```

#### 计数通过时的行为规范
如果count大于Set内部的元素数量，则该命令将只返回整个集合而不包含其他元素。  

#### 返回元素的分配
请注意，当您需要保证返回元素的均匀分布时，此命令不适用。有关用于SPOP的算法的更多信息，请查找Knuth采样和Floyd采样算法。

#### 计数参数扩展
Redis 3.2将会是第一个count可以传递可选参数的版本，SPOP以便在一次调用中检索多个元素。该实现已经在unstable分支中可用。

--- 

## SRANDMEMBER key [count]

>起始版本：1.0.0  
时间复杂度：Without the count argument O(1), otherwise O(N) where N is the absolute value of the passed count.  

仅提供key参数,那么随机返回key集合中的一个元素.  

Redis 2.6开始, 可以接受 count 参数,如果count是整数且小于元素的个数，返回含有 count 个不同的元素的数组,如果count是个整数且大于集合中元素的个数时,仅返回整个集合的所有元素,当count是负数,则会返回一个包含count的绝对值的个数元素的数组，如果count的绝对值大于元素的个数,则返回的结果集里会出现一个元素出现多次的情况.  

仅提供key参数时,该命令作用类似于SPOP命令, 不同的是SPOP命令会将被选择的随机元素从集合中移除, 而SRANDMEMBER仅仅是返回该随记元素,而不做任何操作.  

#### 返回值
bulk-string-reply: 不使用count 参数的情况下该命令返回随机的元素,如果key不存在则返回nil.
array-reply: 使用count参数,则返回一个随机的元素数组,如果key不存在则返回一个空的数组.

#### 举例
```
redis> SADD myset one two three
(integer) 3
redis> SRANDMEMBER myset
"one"
redis> SRANDMEMBER myset 2
1) "three"
2) "one"
redis> SRANDMEMBER myset -5
1) "one"
2) "one"
3) "one"
4) "one"
5) "one"
redis> 
```

#### 计数通过时的行为规范

当一个count参数被传递并且是肯定的时候，元素被返回，好像每个被选中的元素都被从集合中移除（就像宾果游戏中的数字提取一样）。但是元素不会从Set中删除。所以基本上：  
- 没有重复的元素被返回。
- 如果count大于Set内部的元素数量，则该命令将只返回整个集合而不包含其他元素。  
- 
如果计数是负数，则行为会发生变化，并且提取会发生，就如同每次提取后将提取的元素放入包中一样，因此可能会重复元素，并且总是返回请求的元素数量，因为我们可以重复相同的元素元素一次又一次，除了一个空的Set（不存在的键），总是会产生一个空数组。  

#### 返回元素的分配
当元素个数很少时，返回元素的分布还不够完善，这是因为我们使用了一个近似的随机元素函数，这个函数并不能保证良好的分布。 
 
所使用的算法在dict.c中实现，对哈希表桶进行采样以找到非空的桶。一旦找到一个非空的桶，因为我们在我们的哈希表实现中使用链接，检查内部元素的数量，并选择一个随机元素。  

这意味着如果在整个散列表中有两个非空桶，并且一个有三个元素，而另一个只有一个，那么桶中单独存在的元素将以更高的概率返回。  

---

## SREM key member [member ...]

>起始版本：1.0.0  
时间复杂度：O(N) where N is the number of members to be removed.  

在key集合中移除指定的元素. 如果指定的元素不是key集合中的元素则忽略 如果key集合不存在则被视为一个空的集合，该命令返回0.  
如果key的类型不是一个集合,则返回错误.  

#### 返回值
integer-reply:从集合中移除元素的个数，不包括不存在的成员.

#### 历史
	= 2.4: 接受多个 member 元素参数. Redis 2.4 之前的版本每次只能移除一个元素.
#### 举例
```
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myset "three"
(integer) 1
redis> SREM myset "one"
(integer) 1
redis> SREM myset "four"
(integer) 0
redis> SMEMBERS myset
1) "three"
2) "two"
redis> 
```

---

## SUNION key [key ...]

>起始版本：1.0.0  
时间复杂度：O(N) where N is the total number of elements in all given sets.

返回给定的多个集合的并集中的所有成员.

#### 例如:
```
key1 = {a,b,c,d}
key2 = {c}
key3 = {a,c,e}
SUNION key1 key2 key3 = {a,b,c,d,e}
```
不存在的key可以认为是空的集合.

#### 返回值
array-reply:并集的成员列表

#### 举例
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SUNION key1 key2
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
redis> 
```

---

## SUNIONSTORE destination key [key ...]

>起始版本：1.0.0  
时间复杂度：O(N) where N is the total number of elements in all given sets.  

该命令作用类似于SUNION命令,不同的是它并不返回结果集,而是将结果存储在destination集合中.  
如果destination 已经存在,则将其覆盖.  

#### 返回值
integer-reply:结果集中元素的个数.

#### 举例
```
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SUNIONSTORE key key1 key2
(integer) 5
redis> SMEMBERS key
1) "c"
2) "e"
3) "b"
4) "a"
5) "d"
redis>
```

---