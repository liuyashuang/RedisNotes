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
