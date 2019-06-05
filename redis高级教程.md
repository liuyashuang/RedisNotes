## Redis 数据备份与恢复
**Redis SAVE 命令用于创建当前数据库的备份**</br>
基本语法：`redis 127.0.0.2:6379>SAVE`</br>
该命令将在redis 安装目录下创建dump.rdb文件</br>

**恢复数据**</br>
如果需要恢复数据，只需将备份文件(dump.rdb)移动到redis 安装目录并启动服务即可</br>
获取redis 目录可以使用CONFIG命令</br>
```
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"
//CONFIG GET dir 输出的redis 安装目录
```
**Bgsave**</br>
创建redis备份文件也可以使用命令BGSAVE,该命令在后台执行

## Redis 安全
可以通过redis 的配置文件设置密码参数，客户端连接到redis服务就需要密码验证，可以使redis
服务更安全</br>
**查看是是否设置了密码验证**</br>
```
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
```
默认情况下 requirepass参数是空的，意味着你无需通过密码验证就可以连接到redis服务</br>
**修改参数，设置密码**</br>
```
127.0.0.1:6379> CONFIG set requirepass "runoob"
OK
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) "runoob"
```
设置密码狗，客户端连接redis 服务就需要密码验证，否则无法执行命令</br>
**密码验证**</br>
AUTH命令基本语法格式：`127.0.0.1:6379> AUTH password`
## Redis 性能测试
Redis 性能测试是通过同时执行多个命令实现的</br>
`redis-benchmark [option] [option value]`</br>
注意：改命令是在redis的目录下执行的，而不是redis客户端的内部指令
```
//实例
$ redis-benchmark -n 10000  -q

PING_INLINE: 141043.72 requests per second
PING_BULK: 142857.14 requests per second
SET: 141442.72 requests per second
GET: 145348.83 requests per second
INCR: 137362.64 requests per second
LPUSH: 145348.83 requests per second
LPOP: 146198.83 requests per second
SADD: 146198.83 requests per second
SPOP: 149253.73 requests per second
LPUSH (needed to benchmark LRANGE): 148588.42 requests per second
LRANGE_100 (first 100 elements): 58411.21 requests per second
LRANGE_300 (first 300 elements): 21195.42 requests per second
LRANGE_500 (first 450 elements): 14539.11 requests per second
LRANGE_600 (first 600 elements): 10504.20 requests per second
MSET (10 keys): 93283.58 requests per second
```
![redis性能测试工具可选参数](https://github.com/liuyashuang/RedisNotes/blob/master/img/redisParameter.png)
```
//实例
C:\Redis>redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 10000 -q
SET: 126582.27 requests per second
LPUSH: 140845.06 requests per second
```
## Redis客户端连接
Redis 通过监听一个TCP端口或者 Unix socket的方式来接受来自客户端的连接，当一个连接建立
后，redis内部会进行以下一些操作：
  - 首先，客户端socket 会被设置为非阻塞模式，因为redi在网络事件处理上采用的是非阻塞多路
  复用模型
  - 然后为这个socket 设置TCP_NODELAY 属性，禁用Nagle 算法
  - 然后创建一个可读的文件事件用于监听这个客户端socket 的数据发送

**最大连接数**</br>
redis2.6版本中最大连接数这个值是可设置的</br>
maxclients 的默认值是10000，可以在redis.conf 中对这个值进行修改
```
127.0.0.1:6379>config get maxclients
1> "maxclients"
2> "10000"
```
也可以在服务启动时设置最大连接数</br>
`redis-server --maxclients 100000`</br>

**客户端命令**</br>
![客户端命令](https://github.com/liuyashuang/RedisNotes/blob/master/img/redisClients.png)

## Redis管道技术
Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有
服务端的响应</br>

## Redis 分区
分区是分割数据到多个redis实例的处理过程，因此每个实例只保存key的一个子集</br>

**分区的优势：**
  - 通过利用多台计算机内存的和值，允许构造更大的数据库
  - 通过多核和多台计算机，允许扩展计算能力，通过多台计算机和网络适配器，允许扩展网络带宽

**分区的不足：**</br>
redis的一些特性在分区方面表现的不是很好：
    - 涉及多个key的操作通常是不被支持的，例如：当两个set映射到不同的redis实例上时，就不
    能对这两个set执行交集操作
    - 涉及多个key 的redis事务不能使用
    - 当使用分区时，数据处理较为复杂，比如需要处理多个rdb/aof文件，并且从多个实例和主机
      备份持久化文件
    - 增加或删除容量也比较复杂，redis集群大多数支持在运行时增加，删除节点的透明数据平衡
      的能力，但类似于客户端分区，代理等其他系统则不支持这项特性。
**分区类型**</br>
1. 范围分区</br>
    最简单的分区方式是按范围分区，就是映射一定范围的对象到特定的Redis实例</br>
    比如，ID从0到10000的用户会保存到实例R1,ID从10001到20000的用户会保存到R2，以此类推</br>
    不足就是要有一个区间范围到实例的映射表，这个表需要被管理，同时还需要各种对象的的映射表，</br>
    通常对redis来说并非是好的方法</br>
2. 哈希分区</br>
    另外一种分区方法是hash分区，这对任何key都适用，也无需是object_name这种形式，
    - 用一个hash函数将key转换为一个数字，比如使用crc32 hash函数，对key foobar执行
      crc32(foobar)会输出类似93024922的整数
    - 对这个整数取模，将其转换为0-3之间的数字，就可以将这个整数映射到4个redis实例中
      的一个了,93024922%4=2,就是说key foobar应该被存到R2实例中。注意：取模操作是取
      除的余数，通常在多种编程语言中用%操作符实现。

## Java 使用redis
**连接到redis服务**</br>
```
//实例
import redis.clients.jedis.Jedis;

public class RedisJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //查看服务是否运行
        System.out.println("服务正在运行: "+jedis.ping());
    }
}
```
**Redis Java String(字符串)实例**</br>
```
import redis.clients.jedis.Jedis;

public class RedisStringJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //设置 redis 字符串数据
        jedis.set("runoobkey", "www.runoob.com");
        // 获取存储的数据并输出
        System.out.println("redis 存储的字符串为: "+ jedis.get("runoobkey"));
    }
}
```
**Redis Java List(列表)实例**</br>
```
import java.util.List;
import redis.clients.jedis.Jedis;

public class RedisListJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //存储数据到列表中
        jedis.lpush("site-list", "Runoob");
        jedis.lpush("site-list", "Google");
        jedis.lpush("site-list", "Taobao");
        // 获取存储的数据并输出
        List<String> list = jedis.lrange("site-list", 0 ,2);
        for(int i=0; i<list.size(); i++) {
            System.out.println("列表项为: "+list.get(i));
        }
    }
}
```
**Redis Java Keys实例**
```
import java.util.Iterator;
import java.util.Set;
import redis.clients.jedis.Jedis;

public class RedisKeyJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");

        // 获取数据并输出
        Set<String> keys = jedis.keys("*");
        Iterator<String> it=keys.iterator() ;
        while(it.hasNext()){
            String key = it.next();
            System.out.println(key);
        }
    }
}
```
