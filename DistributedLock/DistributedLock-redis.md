### 分部锁之redis实现
#### 一、分布式锁
分布式锁，是一种思想，实现的方法有很多。比如讲沙滩当做分布式锁的组件，
- 加锁</br>
在沙滩上踩一脚，留下自己的脚印，对应加锁操作。其他进程或者线程，看到沙滩上有脚印，
证明锁已被人使用，则等待。
- 解锁</br>
把脚印从沙滩上抹去
- 锁超时</br>
为了避免死锁，可以设置一阵风，在单位时间后刮起，将脚印自动抹去</br>

分布式锁的实现有很多种，
比如基于数据库，memcached、redis、系统文件，zookeeper等，核心思想不变。

#### 二、redis
##### 1.加锁
加锁实际上就是在redis中，给key键设置一个值，为避免死锁，并给定一个过期时间</br>
`SET lock_key random_value NX PX 5000`</br>
值得注意的是：</br>
random_value 是客户端生成的唯一的字符串</br>
NX 代表只在键不存在时，才对键进行设置操作</br>
PX 5000 设置键的过期时间为5000毫秒</br>
##### 2.解锁
解锁的过程就是将key键删除，但也不能随意删除，不能说客户1的请求将客户2的请求删除掉</br>

为了保证解锁操作的原子性，使用LUA脚本完成这一操作。先判断当前锁的字符串是否与转入的值相等，
是的话就删除Key，解锁成功。

```
if redis.call('get',KEYS[1]) == ARGV[1] then
   return redis.call('del',KEYS[1])
else
   return 0
end
```
##### 3.实现
首先在pom文件中，引入Jedis
```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.0.1</version>
</dependency>
```
加锁过程就是通过set指令来设置值，成功则返回；否者就循环等待，在timeout时间内仍未获取到锁，
则获取失败
```
@Service
public class RedisLock {

    Logger logger = LoggerFactory.getLogger(this.getClass());

    private String lock_key = "redis_lock"; //锁键

    protected long internalLockLeaseTime = 30000;//锁过期时间

    private long timeout = 999999; //获取锁的超时时间


    //SET命令的参数
    SetParams params = SetParams.setParams().nx().px(internalLockLeaseTime);

    @Autowired
    JedisPool jedisPool;


    /**
     * 加锁
     * @param id
     * @return
     */
    public boolean lock(String id){
        Jedis jedis = jedisPool.getResource();
        Long start = System.currentTimeMillis();
        try{
            for(;;){
                //SET命令返回OK ，则证明获取锁成功
                String lock = jedis.set(lock_key, id, params);
                if("OK".equals(lock)){
                    return true;
                }
                //否则循环等待，在timeout时间内仍未获取到锁，则获取失败
                long l = System.currentTimeMillis() - start;
                if (l>=timeout) {
                    return false;
                }
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }finally {
            jedis.close();
        }
    }
}
```

解锁我们通过<font color="EE82EE">jedis.evel</font>来执行一段LUA就可以，将锁的Key键
和生成的字符串当做参数传进来
```
/**
  * 解锁
  * @param id
  * @return
  */
 public boolean unlock(String id){
     Jedis jedis = jedisPool.getResource();
     String script =
             "if redis.call('get',KEYS[1]) == ARGV[1] then" +
                     "   return redis.call('del',KEYS[1]) " +
                     "else" +
                     "   return 0 " +
                     "end";
     try {
         Object result = jedis.eval(script, Collections.singletonList(lock_key),
                                 Collections.singletonList(id));
         if("1".equals(result.toString())){
             return true;
         }
         return false;
     }finally {
         jedis.close();
     }
 }
```

开启1000个线程，对count进行累加，调用的时候，关键是唯一字符串的生成。设计到<font color="EE82EE">Snowflake</font>算法
```
@Controller
public class IndexController {

    @Autowired
    RedisLock redisLock;

    int count = 0;

    @RequestMapping("/index")
    @ResponseBody
    public String index() throws InterruptedException {

        int clientcount =1000;
        CountDownLatch countDownLatch = new CountDownLatch(clientcount);

        ExecutorService executorService = Executors.newFixedThreadPool(clientcount);
        long start = System.currentTimeMillis();
        for (int i = 0;i<clientcount;i++){
            executorService.execute(() -> {

                //通过Snowflake算法获取唯一的ID字符串
                String id = IdUtil.getId();
                try {
                    redisLock.lock(id);
                    count++;
                }finally {
                    redisLock.unlock(id);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        long end = System.currentTimeMillis();
        logger.info("执行线程数:{},总耗时:{},count数为:{}",clientcount,end-start,count);
        return "Hello";
    }
}
```
单节点Redis分布式锁的实现已经完成。比较简单，但是存在锁不可重入问题


#### 三、redisson
