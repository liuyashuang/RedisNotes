## Redis概念
#### redis特点：
    redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载使用
    redis不仅仅支持简单的key-value类型的数据，同时还支持list,set,zset,hash扥数据结构的存储
    redis支持数据的备份，即master-slave模式的数据备份
#### redis优势：
    性能极高
    丰富的数据类型，支持二进制案例的Strings,list,hashs,sets及ordered sets数据类型操作
    原子-redis的所有操作都是原子性的，
    丰富的特性

## Redis 数据类型
Redis支持五种数据类型：String（字符串）、hash（哈希）、list(列表)，set(集合)、zset（有序集合）
1. ##### String（字符串）</br>
    String 是 redis 最基本的的类型,你可以理解成与 Memcached 一模一样的类型，一个key对应一个value</br>
    (Memacached 是分布式内存对象缓存系统，本质上是一个简洁的key-value存储系统)</br>
    String 类型是二进制安全的，意思是redis的string可以包含任何数据</br>
    String 类型是Redis最基本的数据类型，string类型的值最大能存储512MB</br>
    ```
    redis 127.0.0.1:6379> SET name "runoob"
    OK
    redis 127.0.0.1:6379> GET name
    "runoob"
    ```
2. ##### Hash（哈希）
    Redis hash 是一个键值(key=>value)队集合
    Redis hash 是一个string类型的field 和 value 的映射表，hash 特备适合用于存储对象
    ```
    redis 127.0.0.1:6379> DEL runoob
    redis 127.0.0.1:6379> HMSET myhash field1 "Hello" field2 "World"
    "OK"
    redis 127.0.0.1:6379> HGET myhash field1
    "Hello"
    redis 127.0.0.1:6379> HGET myhash field2
    "World"
    ```
    每个hash可以存储 2^32-1键值对（40多亿）
3. ##### List(列表)
    Redis列表是简单的字符串列表，按照插入顺序排序，可以添加一个元素到列表的头部或尾部</br>
    ```
    redis 127.0.0.1:6379> DEL runoob
    redis 127.0.0.1:6379> lpush runoob redis
    (integer) 1
    redis 127.0.0.1:6379> lpush runoob mongodb
    (integer) 2
    redis 127.0.0.1:6379> lpush runoob rabitmq
    (integer) 3
    redis 127.0.0.1:6379> lrange runoob 0 10
    1) "rabitmq"
    2) "mongodb"
    3) "redis"
    redis 127.0.0.1:6379>
    ```
    列表最多可以存储2^32-1元素（每个列表可存储40多亿）
4. ##### Set(集合)
    Redis的Set是String类型的无序集合</br>
    集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)</br>
    sadd命令（添加一个string元素到key对应的set集合中，成功返回1.如果元素已经在集合中返回0，
    如果key对象的set集合不存在返回错误）
    ```
    redis 127.0.0.1:6379> DEL runoob
    redis 127.0.0.1:6379> sadd runoob redis
    (integer) 1
    redis 127.0.0.1:6379> sadd runoob mongodb
    (integer) 1
    redis 127.0.0.1:6379> sadd runoob rabitmq
    (integer) 1
    redis 127.0.0.1:6379> sadd runoob rabitmq
    (integer) 0
    redis 127.0.0.1:6379> smembers runoob

    1) "redis"
    2) "rabitmq"
    3) "mongodb"
    ```
    注意：上面实例中 rabitmq添加了两次，但根据集合内元素的唯一性，第二次插入的元素将被忽略</br>
    集合中最大的成员数为2^32-1（40亿）
5. ##### zset（sorted set：有序集合）
    zset和set一样也是string类型元素的集合，且不允许重复的成员</br>
    不同的是每个元素都会关联一个double类型的分数，redis通过分数来为集合中的成员进行从小到大的排序</br>
    zset的成员是唯一的，但分数却可以重复</br>
    zadd命令(添加元素到集合，元素在集合中存储则更新对应分数)
    ```
    redis 127.0.0.1:6379> DEL runoob
    redis 127.0.0.1:6379> zadd runoob 0 redis
    (integer) 1
    redis 127.0.0.1:6379> zadd runoob 0 mongodb
    (integer) 1
    redis 127.0.0.1:6379> zadd runoob 0 rabitmq
    (integer) 1
    redis 127.0.0.1:6379> zadd runoob 0 rabitmq
    (integer) 0
    redis 127.0.0.1:6379> > ZRANGEBYSCORE runoob 0 1000
    1) "mongodb"
    2) "rabitmq"
    3) "redis"
    ```
![各数据类型](https://github.com/liuyashuang/RedisNotes/blob/master/img/redis01.png)
