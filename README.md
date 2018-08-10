# redis_py
本项目用途记录Redis的python实现

# 注意 
在阅读项目前，务必掌握Redis的以下几个要素
- Redis安装
- 对Redis的学习目的有比较清晰的认识
- Redis的基础数据结构

## Redis安装
主要有四种方式
- Docker安装
- GitHub源码编译
- apt-get install(Ubuntu),yum install(RedHat),brew install(Mac)
- Web Redis

## Redis基础数据结构
- String（字符串）
- hash（哈希）
- zset（有序集合）
- list（列表）
- set（集合）
### String

> Redis的String是动态的，可修改的，内部结构实现类似Java的ArrayList,当字符串长度小于1M，每次扩容都是加倍现有的空间，超过1M一次只多扩1M，字符串最大空间为512M.
```
# 单个键值对
set key value -> 设置key/value
get key -> 获取key对应的value值
exists key -> 是否存在key
del key ->删除key
# 批量键值对
mset key1 value1 key2 value2
mget key1 key2 -> 获取value列表
# 过期和set命令扩展
expire key time # time单位为秒
ttl key ->查剩余过期时间
setex key time value -> 等价于set + expire
setnx key value -> 如果key不存在就执行set创建，否则不创建
# 计数(value是一个整数)
incr key -> value递增1
incrby key num -> value递增num个单位，num为正负整数，value范围为signed long的最大最小值
```
### list
> list是链表实现的，因此list的插入和删除操作非常快，时间复杂度为O(1),索引复杂度为O(n),list常用来做异步队列使用，将需要延后处理的任务结构体序列化成字符串塞进Redis的列表，另一个线程从这个列表中轮询数据进行处理.
```
# 队列/栈
rpush name values... -> 入队n个值,队列名称name
llen  name-> 获取队列长度
lpop name -> 出队
lindex name index -> 获取下标index的元素，index为-1表示最后一个元素，O(n)慎用
lrange name start end -> 获取start->end的元素，O(n)慎用
ltrim name start end -> 清楚start ->end以外的其他元素
```

### Hash
> 相当于Java里的HashMap,实现上也是相似的数组 + 链表二维结构，不同的是Redis的字典值只能是字符串，Redis的Hash在rehash时会保留新旧hash结构，查询时同时查询俩个hash结构然后在后续的定时任务中以及hash操作指令中，将旧的hash结构内容迁移到新的 hash结构中，当搬迁完成会使用新的hash的结构取而代之.
```
hset name  obj value -> 设置obj值为value,字典名为name
hgetall name -> 获取字典所有的obj
hlen name -> 获取字典的长度
hget name obj ->获取字典obj的值
hincrby name obj 1 -> 字典obj的值递增1个单位
```

### Set
> 相当于Java中的HashSet,无序且唯一的，内部实现相当于一个特殊的Hash字典，字典所有的value都是null,当集合最后一个元素移除之后，数据结构自动删除.
```
sadd name values -> 设置无序集合多个value值
smembers name -> 顺序和插入并不一致
sismember name value -> 查询某个value是否存在
scard name -> 获取集合长度
spop name -> 弹出一个
```
### zset 
> 类似于Java的SortedSet 和HashMap的结合体，一方面它是一个set,保证了内部value的唯一性，另一方面它可以给每个value赋予一个score，代表这个value的排序权重，它的内部实现使用的是一种叫做**跳跃列表**的数据结构,zset中最后一个value被移除后，数据结构自动删除，内寸被删除.

```
zadd name obj.value1,obj.value2 -> 设置obj对象，按照value1排序
zrange name start end -> 按照value1顺序输出，排序范围为start~end
zrevrange name start end -> 按照value1逆序输出，排序范围为start~end
zcard name ->相当于count()
zrank name obj.value2 -> 返回value2值排名
zrangebyscore name start end -> 根据分值区间遍历zset
zrangebyscore name -inf end -> 根据分值区间遍历zset，-inf表示无穷大
zrem name -> 删除
```


## 容器型数据结构的通用规则
list/set/hash/zset 属于容器型数据结构，共享下面2条通用规则。
1.create if not exists
2.drop if no elements

# 最后
本项目的学习总结来源老钱的《Redis深度历险：核心原理与应用实践》,主要是针对Redis的实践应用，欢迎一起交流探讨^_^_