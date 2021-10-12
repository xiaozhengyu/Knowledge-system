# 基于 Redis 实现的限流

[Gitee代码](https://gitee.com/zhengyuxiao/redis-demo/tree/master/redis-demo/src/test/java/com/xzy/demo/limiting)

---

## 1. 基于set命令

### 思路：

基于接口信息和用户信息生成UUID并以此作为key，借助 set 命令记录用户在一段时间内访问接口的==次数==

```pseudocode
查询当前访问次数;
if( 初次访问 ){
    记录访问次数为1并设置过期时间;
    return true;
}

if( 访问次数 < 限制次数){
    累计访问次数;
    return true;
}

return false;

```

>   <font color = #1AA3FF>SET</font> key value [<font color = #1AA3FF>EX</font> seconds] [<font color = #1AA3FF>PX</font> milliseconds] [<font color = #1AA3FF>NX</font>|<font color = #1AA3FF>XX</font>]
>
>   -   `EX` *seconds* – 设置键key的过期时间，单位时秒
>   -   `PX` *milliseconds* – 设置键key的过期时间，单位时毫秒
>   -   `NX` – 只有键key不存在的时候才会设置key的值
>   -   `XX` – 只有键key存在的时候才会设置key的值
>
>   
>
>   <font color = #1AA3FF>INCR</font> key
>
>   **说明：**对存储在指定`key`的数值执行加1操作。如果指定的key不存在，那么在执行INCR操作之前，会先将它的值设定为`0`；如果指定的key中存储的值不是字符串类型，或者存储的字符串类型不能表示为一个整数，那么执行这个命令时服务器会返回错误信息。
>
>   <font color = #1AA3FF>GET</font> key
>
>   **说明：**返回指定的key的value。如果key不存在，返回特殊值nil；如果key的value不是string类型，返回错误信息（GET只支持处理string类型的values）。



### 评估：

1.   流量窗口固定

     假如限制10s内访问次数限制为20次，如果在第1秒就访问了20次，那么之后9秒就不能访问

2.   统计方式不灵活

     在共一个key的情况下，统计 1-10s 的流量信息时，无法统计 2-10s 的流量信息

     

### 实现：

```java
/**
 * 流量限制器_基于set命令
 *
 * @author xzy
 * @date 2021/10/12 11:13
 */
@Slf4j
@AllArgsConstructor
public class LimiterA implements Limiter {
    /**
     * 访问次数限制
     */
    public final int limit;

    /**
     * 访问时间限制
     */
    public final int expire;

    private final RedisTemplate<String, Object> redisTemplate;

    /**
     * 是否限制访问
     *
     * @param uuid - 基于接口、用户信息生成的唯一值
     * @return - false：放行访问 true：限制访问
     */
    @Override
    public boolean isLimit(String uuid) {
        Object value = redisTemplate.opsForValue().get(uuid);

        if (value == null) {
            log.debug("无需限制：初次访问    访问次数：0");
            redisTemplate.opsForValue().set(uuid, 1, expire, TimeUnit.SECONDS);
            return false;
        }

        int accTimes = (Integer) value;
        if (accTimes < limit) {
            log.debug("无需限制：未达上限    访问次数：{}", accTimes);
            redisTemplate.opsForValue().increment(uuid);
            return false;
        }

        log.debug("限制访问：已达上限    访问次数：{}", accTimes);
        return true;
    }
}
```

## 2. 基于ZSet数据结构

### 思路：

基于接口信息和用户信息生成UUID并以此作为key，借助 ZSet 记录用户访问指定接口的==日志==，并以时间戳作为score，方便后续统计任意时间范围内的接口访问次数

```pseudocode
统计当前访问日志的条数
if( 初次访问 || 访问次数 < 限制次数 ){
    新增1条访问日志
    return true;
}

return false;
```



### 评估：

1.   数据量庞大



### 实现：

```java
/**
 * 流量限制器_基于ZSet数据结构（不考虑并发、事务）
 *
 * @author xzy
 * @date 2021/10/12 13:28
 */
@Slf4j
@AllArgsConstructor
public class LimiterB implements Limiter {
    /**
     * 访问次数限制
     */
    public final long limit;

    /**
     * 访问时间限制（毫秒）
     */
    public final long expire;

    private final RedisTemplate<String, Object> redisTemplate;

    /**
     * 是否限制访问
     *
     * @param uuid - 基于接口、用户信息生成的唯一值
     * @return - false：放行访问 true：限制访问
     */
    @Override
    public boolean isLimit(String uuid) {
        long currentTimeMillis = System.currentTimeMillis();
        Long accTimes = redisTemplate.opsForZSet().count(uuid, currentTimeMillis - expire, currentTimeMillis);

        if (accTimes == null) {
            log.debug("无需限制：初次访问    访问次数：0");
            redisTemplate.opsForZSet().add(uuid, UUID.randomUUID().toString(), currentTimeMillis);
            return false;
        }

        if (accTimes < limit) {
            log.debug("无需限制：未达上限    访问次数：{}", accTimes);
            redisTemplate.opsForZSet().add(uuid, UUID.randomUUID().toString(), currentTimeMillis);
            return false;
        }

        log.debug("限制访问：已达上限    访问次数：{}", accTimes);
        return true;
    }
}
```



## 3. 基于令牌桶算法

### 思路：

1.   以List数据结构实现令牌桶
2.   创建定时任务向令牌桶定期添加令牌，直到令牌桶被填满
3.   每次访问接口前需要先从令牌桶获取令牌

>   令牌桶算法：
>
>   令牌桶算法是网络流量整形（Traffic Shaping）和速率限制（Rate Limiting）中最常使用的一种算法。典型情况下，令牌桶算法用来控制发送到网络上的数据的数目，并允许突发数据的发送。
>
>   大小固定的令牌桶可自行以恒定的速率源源不断地产生令牌。如果令牌不被消耗，或者被消耗的速度小于产生的速度，令牌就会不断地增多，直到把桶填满。后面再产生的令牌就会从桶中溢出。最后桶中可以保存的最大令牌数永远不会超过桶的大小。
>
>   传送到令牌桶的数据包需要消耗令牌。不同大小的数据包，消耗的令牌数量不一样。
>
>   令牌桶这种控制机制基于令牌桶中是否存在令牌来指示什么时候可以发送流量。令牌桶中的每一个令牌都代表一个字节。如果令牌桶中存在令牌，则允许发送流量；而如果令牌桶中不存在令牌，则不允许发送流量。因此，如果突发门限被合理地配置并且令牌桶中有足够的令牌，那么流量就可以以峰值速率发送。



### 实现：

```java
/**
 * 流量限制器_基于令牌桶算法（不考虑并发、事务）
 *
 * @author xzy
 * @date 2021/10/12 15:13
 */
@Slf4j
@AllArgsConstructor
public class LimiterC implements Limiter {
    /**
     * 访问次数限制
     */
    public final long limit;

    /**
     * 访问时间限制（毫秒）
     */
    public final long expire;

    private final RedisTemplate<String, Object> redisTemplate;

    /**
     * 添加令牌
     *
     * @param uuid - 标识接口的UUID
     */
    public void push(String uuid) throws InterruptedException {
        while (true) {
            Thread.sleep(expire / limit);
            String token = UUID.randomUUID().toString();
            redisTemplate.opsForList().leftPush(uuid, token);
            log.debug("新增令牌：{}", token);
        }
    }

    /**
     * 是否限制访问
     *
     * @param uuid - 基于接口、用户信息生成的唯一值
     * @return - false：放行访问 true：限制访问
     */
    @Override
    public boolean isLimit(String uuid) {
        Long stockCount = redisTemplate.opsForList().size(uuid);
        if (stockCount == null || stockCount == 0) {
            log.debug("限制访问：无法获取访问令牌");
            return true;
        }

        String token = (String) redisTemplate.opsForList().rightPop(uuid);
        log.debug("允许访问: 访问令牌剩余{}，获取访问令牌{}", stockCount, token);
        return false;
    }
}
```