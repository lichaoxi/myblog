---
title: Redis性能优化和高级用法
tags: [Redis,性能优化]
categories: Redis
copyright: true
---

同关系型数据库一样，Redis也有自己的性能问题需要优化，在生产上可能会遭遇慢查询，弱事务，原子性等场景，这里总结几类Redis的常见问题并给出对应的解决方案。
<!--more-->
### 一、慢查询优化
***
从客户端发出的一个请求可以分为下图中4个步骤（`发送命令－〉命令排队－〉命令执行－〉返回结果`），由于Redis是单线程模型，那么很有可能一两个较慢的查询阻塞了多个业务的查询效率，这个时候需要对耗时较长的慢查询进行定位分析，首先需要设定Redis的日志，有两个关键参数可以记录第三个步骤执行命令的耗时记录。

![Redis查询](https://upload-images.jianshu.io/upload_images/8926909-aff84bd7a51a4625.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关于 Redis 慢查询的配置有两个，分别是 `slowlog-log-slower-than` 和 `slowlog-max-len`。 
- 1.`slowlog-log-slower-than`，用来控制慢查询的阈值，所有执行时间超过该值的命令都会被记录下来。该值的单位为微秒，默认值为 `10000`(10毫秒)，如果设置为 `0`，那么所有的记录都会被记录下来，如果设置为小于 `0` 的值，那么对于任何命令都不会记录，即关闭了慢查询。可以通过在配置文件中设置，或者用 `config set` 命令来设置：
```
config set slowlog-log-slower-than 10000
```
使用config set完后，若想将配置持久化保存到redis.conf，要执行`config rewrite `
- 2.`slowlog-max-len`，用来设置存储慢查询记录列表的大小，默认值为 `128`，当该列表满了时，如果有新的记录进来，那么 Redis 会把队最旧的记录清理掉，然后存储新的记录。在生产环境我们可以适当调大，比如调成 `1000`，这样就可以缓冲更多的记录。

慢查询配置好，那么在定位问题的时候可以采用`slowlog get`获取信息。运行Demo如下：
```java
127.0.0.1:6379> slowlog get
 1) 1) (integer) 456
    2) (integer) 1531632044
    3) (integer) 3
    4) 1) "get"
       2) "m"
    5) "127.0.0.1:50106"
    6) ""
 2) 1) (integer) 455
    2) (integer) 1531632037
    3) (integer) 14
    4) 1) "keys"
       2) "*"
    5) "127.0.0.1:50106"
    6) "
```
分别对应着：
1) 慢查询记录 id
2) 发起命令的时间戳
3) 命令耗时，单位为微秒
4) 该条记录的命令及参数
5) 客户端网络套接字（ip: port）

**单个慢查询的操作定位出来以后，一般可以按照拆解的思路，化大为小，尽量减少对其他业务请求的影响。**
>source：[Redis 慢查询分析](https://blog.csdn.net/qianghaohao/article/details/81052461)

### 二、Pipeline（管道）
***
上面已提到过Redis执行命令分为了4个步骤：`发送命令－〉命令排队－〉命令执行－〉返回结果`，这个过程称为`Round trip time`(简称RTT, 往返时间)，`mget mset`有效节约了RTT，但大部分命令（如hgetall，并没有mhgetall）不支持批量操作，需要消耗N次RTT 。有时我们会在短时间内发送大量互不依赖的命令(例如：后执行的命令不需要使用前面返回的结果)这个时候需要Pipeline来解决这个问题。

使用Pipeline在对Redis批量读写的时候，性能上有非常大的提升。因为管道可以在一次tcp的请求中同时发送多条命令，并且将响应结果一次性的返回给客户端。对于既定数量的命令请求，Redis管道通过减少客户端和服务器端的通信次数，来达到减少通信传输中往返时间的目的，提高效率。

Pipeline示例代码，用法比较简单：
```java
Pipeline pipelined = jedis.pipelined();
Pipeline p = jedis.pipelined();
/* 插入多条数据 */
for(Integer i = 0; i < 100000; i++) {
      p.set(i.toString(), i.toString());
}
             
Response<String> response = p.get("999");
/**
 * 执行报异常：redis.clients.jedis.exceptions.JedisDataException: 
 * Please close pipeline or multi block before calling this method.
 * System.out.println(response.get()); 
**/
p.sync();
```
- 和Redis的事务类似，管道能完成的操作也能够被更加灵活的Redis脚本实现，但是脚本的可读性不强、可维护性差，**如果批量处理的命令之间不存在依赖关系时，优先使用管道**；反之，则只能使用脚本了。
- 用Pipeline方式，Redis在处理完所有命令前先缓存起所有命令的处理结果，打包的命令越多，缓存消耗内存也越多，所以并不是打包的命令越多越好，**正确的做法是可以将大量命令的拆分多个小的Pipeline命令完成**。

***注意：由于pipeline的原理是收集需执行的命令，到最后才一次性执行。所以无法在中途立即查得数据的结果（需待pipelining完毕后才能查得结果），这样会使得无法立即查得数据进行条件判断。***
>source：[redis管道](https://www.cnblogs.com/xiaoxiongcanguan/p/9954254.html)

### 三、Redis事务和Lua脚本
***
Redis 事务可以一次执行多个命令，它的用法也比较简答，将一组需要一起执行的命令放到`multi`和`exec`两个命令之间，其中`multi`代表事务开始，`exec`代表事务结束。但是**Redis的事务并不保证原子性**，或者说是一种弱事务机制，Redis的事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。如果需要实现原子操作，**Redis可以结合Lua脚本来保证原子性**。

本人所在项目的红包模块在Redis上有很不错的应用，利用Redis的高性能缓存实现高并发秒抢，利用Lua实现原子操作，这部分的思路，可以在[【利用Redis和Lua的原子性实现抢红包功能】](https://www.jianshu.com/p/b58ed2fe6976?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)有进行参考，而具体的源码实现可以参照 [【Redis+Lua的红包源码】](https://github.com/ZhuBaker/redis-lua)来实现。

把其中的lua脚本和注释贴出来FYI：
```lua
/**
*      -- 函数：尝试获得红包，如果成功，则返回json字符串，如果不成功，则返回空
*      -- 参数：红包队列名， 已消费的队列名，去重的Map名，用户ID
*      -- 返回值：nil 或者 json字符串，包含用户ID：userId，红包ID：id，红包金额：money
*
*      -- 如果用户已抢过红包，则返回nil
*      if redis.call('hexists', KEYS[3], KEYS[4]) ~= 0 then
*        return nil
*      else
*        -- 先取出一个小红包
*        local hongBao = redis.call('rpop', KEYS[1]);
*        if hongBao then
*          local x = cjson.decode(hongBao);
*          -- 加入用户ID信息
*          x['userId'] = KEYS[4];
*          local re = cjson.encode(x);
*          -- 把用户ID放到去重的set里
*          redis.call('hset', KEYS[3], KEYS[4], KEYS[4]);
*          -- 把红包放到已消费队列里
*          redis.call('lpush', KEYS[2], re);
*          return re;
*        end
*      end
*      return nil
*/
```
而在Java里加载脚本的方法也比较easy：
```java
     //线程模拟抢红包
     public void run() {
        System.out.println("BEGIN " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
        Jedis jedis = RedisJava.getResource();
        for(int i = beginNum ; i <= endNum; i++){
            try{
                List<String> userInfo = new ArrayList<>(3);
                userInfo.add(""+i);
                userInfo.add("name" + i);
                userInfo.add("http://xxx" + i);
                // Lua脚本执行
                Object eval = jedis.evalsha(hashsha, 3, userInfo.toArray(new String[]{}));
                System.out.println(eval);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        jedis.close();
        System.out.println("END " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
    }				
```