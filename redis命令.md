### Redis命令
###### 连接到本地redis服务器
```
$redis-cli
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING

PONG
```
###### 在远端服务器上执行命令
```
$ redis-cli -h host -p port -a password
//案例
$redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING

PONG
```
出现中文乱码解决：在redis-cli 后面加上--raw  redis-cli --raw

### Redis键（key）
Redis键命令的基本语法
`redis 127.0.0.1:6379>COMMAND KEY_NAME`
```
//实例
redis 127.0.0.1:6379> SET runoobkey redis
OK
redis 127.0.0.1:6379> DEL runoobkey
(integer) 1
```
Redis键相关的基本命令
  1. DEL key
          DEL命令用于删除已存在的键，不存在的key会被忽略
          返回值：被删除key 的数量
  2. Dump命令
          DUMP命令用于序列化给定key，并返回被序列化的值
          返回值：如果key不存在，那么返回nil.否则返回序列化之后的值
  3. Exists命令
          EXISTS命令用于检查给定key是否存在
          返回值：若key存在返回1，否则返回0
  4. Expire命令
          EXPIRE命令用于设置key的过期时间，key过期后将不再可用，单位以秒计
          返回值：设置成功返回1，当key不存在或者不能设置时（版本低）否则返回0
  5. Expireat命令
          Expireat命令用于以UNIX时间戳格式设置key的过期时间，key过期后不可再用
          基本语法：redis 127.0.0.1:6379> Expireat KEY_NAME TIME_IN_UNIX_TIMESTAMP
          返回值：成功返回1，当key不存在或者不能设置时（版本丢）（版本低）否则返回0
  6. PEXPIRE命令
          PEXPIRE命令和EXPIRE命令的作用类似，但是它是以毫秒为单位设置key的生存时间
          返回值：成功返回1，key不存在或设置失败返回0
          还有设置key过期时间的时间戳以毫秒计(PEXPIREAT)
  7. Keys命令
          KEYS命令用于查找所有符合给定模式pattern的key
          返回值：符合给定模式的key列表(Array)
    ```
    //实例，创建一些key并赋值
    redis 127.0.0.1:6379> SET runoob1 redis
    OK
    redis 127.0.0.1:6379> SET runoob2 mysql
    OK
    redis 127.0.0.1:6379> SET runoob3 mongodb
    OK
    //查找以runoob为开头的key
    redis 127.0.0.1:6379> KEYS runoob*
    1) "runoob3"
    2) "runoob1"
    3) "runoob2"
    //获取redis中所有的key 可使用 *
    redis 127.0.0.1:6379> KEYS *
    ```
  8. MOVE命令
          MOVE命令用于将当前数据库的key移动到给定的数据库db中
          基本语法：redis 127.0.0.1:6379> MOVE KEY_NAME DESTINATION_DATABASE
          返回值：移动成功返回1，失败返回0
          注意：当两个数据库有相同名字的数据时，移动会报错，移动目标数据库的值不会改变
  9. PERSIST命令
          PERSIST命令用于移除给定key的过期时间。使得key永不过期
          返回值：移除成功返回1，key不存在或者key没有设置过期时间返回0
  10. PTTL命令
          Pttl命令以毫秒为单位返回key的剩余过期时间
          返回值：当key不存在时返回-2，当key存在但没有剩余生存时间时返回-1，否则返回剩余时间（毫秒为单位的）
          在 Redis 2.8以前，当key不存在或者key没有剩余生存时间时都是返回-1
  11. TTL 命令
          TTL命令以秒为单位返回key的剩余过期时间
          返回值与PTTL 相似
  12. RANDOMKEY 命令
          RANDOMKEY 命令从当前数据库中随机返回一个key
          返回值：当数据库不为空时，返回一个key，当数据库为空时，返回nil(window系统返回null)
  13. RENAME 命令
          RENAME命令用于修改key的名称
          返回值：成功时提示OK,失败时候返回一个错误
          当老名字和新名字相同，或者老名字不存在时，返回一个错误，当新名字已经存在时，RENAME命令将覆盖旧值
  14. RENAMENX 命令
          Renamenx命令用于在新的key不存在时修改key的名称
          返回值：修改成功时，返回1，如果NEW_KEY_NAME已经存在，返回0
  15. Type命令
          Type命令用于返回key所储存的值的类型
          返回值：类型有 none(key不存在),string,list,set,zset,hash

### Redis 字符串(String)
1. SET key value
        SET命令用于设置给的key的值，如果key已经存储其他值，SET就覆写旧值
2. GET key
        GET命令用于获取指定key的值
3. GETRANGE key start end
        返回key中字符串值的子字串
4. GETSET key value
        将给定key的值设为value，并返回key 的旧值
        当key没有旧值，即key不存在时，返回nil
        当key存在但不是字符串类型时，返回一个错误
5. GETBIT key offset
        Getbit命令用于对key所储存的字符串值，获取指定偏移量上的位(bit)
        返回值：
          字符串值指定偏移量上的位
          当偏移量OFFSET比字符串值的长度大，或者key不存在时，返回0
6. MGET key1[key2]
        获取所有（一个或多个）将给定key的值
        返回一个或多个给定key的值，如果给定的key里面，有某个key不存在，那么这个key返回特殊值nil
7. SETBIT key offset value
        Setbit命令用于对key所存储的字符串值，设置或清除指定偏移量上的位
8. SETEX key seconds value
        将值value关联到key，并将key的过期时间设为seconds（以秒为单位）
        如果key已存在，SETEX命令将会替换旧的值
9. SETNX key value
        在指定的key不存在时，为key设置指定的值
        设置成功返回1，设置失败返回0
10. SETRANGE key offset value
        用value参数覆写给定key所存储的字符串值，从偏移量offset开始
11. STRLEN key
        返回key所储存的字符串值的长度
12. MSET key value [key value]
        同时设置一个或多个key-value对
13. MSETNX key value [key value]
        同时设置一个或多个key-value对，当且仅当所有给定key都不存在
14. PSETEX key milliseconds value
        以毫秒为单位设置key的生存时间
15. INCR key
        将key中储存的数字值增一
16. INCRBY key increment
        将key所储存的值加上给定的增量值（increment）
17. INCRBYFLOAT key increment
        将key所储存的值加上给定的浮点增量值（increment）
18. DECR key
        将key中储存的数字值减一
19. DECRBY key decrement
        key 所储存的值减去给定的减量值（decrement）
20. APPEND key value
        如果key已经存在并且是一个字符串,APPEND命令将指定的value追加到该key原来值（value）的末尾
### Redis哈希（Hash）
![哈希命令1](https://github.com/liuyashuang/RedisNotes/blob/master/img/hash01.png)</br>
![哈希命令2](https://github.com/liuyashuang/RedisNotes/blob/master/img/hash02.png)
### Redis列表(List)
Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）</br>
![列表命令1](https://github.com/liuyashuang/RedisNotes/blob/master/img/list01.png)</br>
![列表命令2](https://github.com/liuyashuang/RedisNotes/blob/master/img/list02.png)
### Redis集合(Set)
Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据</br>
Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)
![集合命令1](https://github.com/liuyashuang/RedisNotes/blob/master/img/set01.png)</br>
![集合命令2](https://github.com/liuyashuang/RedisNotes/blob/master/img/set02.png)
### Redis有序集合(ZSet)
Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员</br>
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序</br>
有序集合的成员是唯一的,但分数(score)却可以重复</br>
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)</br>
![有序集合命令1](https://github.com/liuyashuang/RedisNotes/blob/master/img/zset01.png)</br>
![有序集合命令2](https://github.com/liuyashuang/RedisNotes/blob/master/img/zset02.png)</br>
![有序集合命令3](https://github.com/liuyashuang/RedisNotes/blob/master/img/zset03.png)

### HyperLogLog
Redis在2.8.9版本添加了HyperLog结构</br>
Redis HyperLogLog是用来做基数统计的算法，优点是，在输入元素的数量或者体积非常非常大的，
计算基数所需的空间总是固定的，并且很小的</br>
在redis里面，每个HyperLogLog 键值需要花费12KB内存，就可以计算接近2^64个不同元素的基数</br>
HyperLogLog只会根据输入元素来计算基数，而不会储存输入元素本事，所以HyperLogLog不能像集合那样
返回输入的各个元素</br>

**基数的概念**</br>
比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8},
基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数
```
//实例
redis 127.0.0.1:6379> PFADD runoobkey "redis"
1) (integer) 1
redis 127.0.0.1:6379> PFADD runoobkey "mongodb"
1) (integer) 1
redis 127.0.0.1:6379> PFADD runoobkey "mysql"
1) (integer) 1
redis 127.0.0.1:6379> PFCOUNT runoobkey
(integer) 3
```
**基本命令**

    PFADD key element [element]
    添加指定元素到HyperLogLog 中
    PFCOUNT key [key]
    返回给定HyperLogLog 的基数估算值
    PFMERGE destkey sourcekey [sourcekey]
    将多个HyperLogLog 合并为一个HyperLogLog

### Redis发布订阅
Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接受消息</br>
Redis 客户端可以订阅任意数量的频道</br>
**常用命令**

    PSUBSCRIBE pattern [pattern]
    订阅一个或多个符合给定模式的频道
    PUBSUB subcommand [argument[argument···]]
    查看订阅与发布系统状态
    PUBLISH channel message
    将信息发送到指定的频道
    PUNSUBSCRIBE [pattern [pattern]]
    退订所有给定模式的频道
    SUBSCRIBE channel [channel]
    订阅给定的一个或多个频道的信息
    UNSUBSCRIBE [channel [channel]]
    退订给定的频道
```
//实例
//创建了订阅频道名为 redisChat
redis 127.0.0.1:6379> SUBSCRIBE redisChat
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
//重新开启个redis客户端，然后在同一个频道redisChat 发布两次消息
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"
(integer) 1
redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by runoob.com"
(integer) 1
# 订阅者的客户端会显示如下消息
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"
1) "message"
2) "redisChat"
3) "Learn redis by runoob.com"
```
### Redis 事务
Redis 事务可以一次执行多个命令， 并且带有以下两个重要的保证
  - 批量操作在发送EXEC 命令前被放入队列缓存
  - 收到EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行
  - 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中

一个事务从开始到执行会经历以下三个阶段
  - 开始事务
  - 命令入队
  - 执行事务

**事务命令**

    DISCARD
    取消事务，放弃执行事务块内的所有命令
    EXEC
    执行所有事务块内的命令
    MULTI
    标记一个事务块的开始
    UNWATCH
    取消WATCH 命令对所有key 的监视
    WATCH key [key]
    监视一个或多个key，如果在事务执行之前这些key被其他命令所改动，那么事务将被打断

**实例**</br>
先以MULTI开始一个事务，然后将多个命令入队到事务中，最后由EXEC命令触发事务，一并执行事务中的所有命令
```
redis 127.0.0.1:6379> MULTI
OK
redis 127.0.0.1:6379> SET book-name "Mastering C++ in 21 days"
QUEUED
redis 127.0.0.1:6379> GET book-name
QUEUED
redis 127.0.0.1:6379> SADD tag "C++" "Programming" "Mastering Series"
QUEUED
redis 127.0.0.1:6379> SMEMBERS tag
QUEUED
redis 127.0.0.1:6379> EXEC
1) OK
2) "Mastering C++ in 21 days"
3) (integer) 3
4) 1) "Mastering Series"
   2) "C++"
   3) "Programming"
```
单个 Redis命令的执行是原子性的，但Redis没有在事务上增加任何维持原子性的机制，所以Redis
事务的执行并不是原子性的</br>
事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会
导致前面已做指令的回滚，也不会造成后续的指令不做

### Redis 脚本
Redis 脚本使用Lua 解析器来执行脚本，执行脚本的常用命令为EVAL</br>
**Redis 脚本命令**

    EVAL script numkeys key [key] arg [arg]
    执行 Lua脚本
    EVALSHA sha1 numkeys key [key] arg [arg]
    执行 Lua脚本
    SCRIPT EXISTS script [script]
    查看置顶的脚本是否已经被保存在缓存当中
    SCRIPT FLUSH
    从脚本缓存中移除所有脚本
    SCRIPT KILL
    杀死当前正在运行的 Lua脚本
    SCRIPT LOAD script
    将脚本script 添加到脚本缓存中，但是并不立即执行这个脚本

**实例**</br>
```
redis 127.0.0.1:6379> EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```
### Redis 连接
主要是用于连接redis服务</br>
**连接命令**

    AUTH password   验证密码是否正确
    ECHO message    打印字符串
    PING            查看服务是否运行
    QUIT            关闭当前连接
    SELECT index    切换到指定的数据库
### Redis 服务器
主要是用于管理redis服务</br>
![服务器命令1](https://github.com/liuyashuang/RedisNotes/blob/master/img/server01.png)</br>
![服务器命令2](https://github.com/liuyashuang/RedisNotes/blob/master/img/server02.png)</br>
![服务器命令3](https://github.com/liuyashuang/RedisNotes/blob/master/img/server03.png)
