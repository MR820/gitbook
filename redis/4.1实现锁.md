### 利用watch+multi来实现乐观锁
乐观锁，顾名思义，乐观的认为数据不会被修改，只有当更新时才去判断数据是否被修改过，通常用版本号或时间戳来实现。
redis中通过watch和multi来实现，watch会监视给定的key是否发生更改，当exec的时候如果监视的key发生过改变，则整个事务会失败。
当然我们可以调用多次watch监视多个key。

```php
<?php

$redis = new Redis();
$redis->connect('127.0.0.1', 6379, 60);

//设置商品的库存数为100
$redis->set('goods_stock_nums', 100);
//监视该key
$redis->watch('goods_stock_nums');

//开启事务
$redis->multi();

//修改库存数,incr原子性递增,decr原子性递减
$redis->decr('goods_stock_nums');

//提交事务，如果在此期间有其他请求修改了该key，那么事务会失败
if ($redis->exec()) {
    echo '抢购成功';
} else {
    echo '数据错误，请重新再试';
}
```
### 使用setnx来实现悲观锁

```php
<?php

function getRedis()
{
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379, 60);
    return $redis;
}

function lock($key, $random)
{
    $redis = getRedis();
    return $redis->set($key, $random, ['nx', 'ex' => 3]);
}

function unlock($key, $random)
{
    $redis = getRedis();
    //使用lua脚本保证原子性
    $script = 'if redis.call("get",KEYS[1]) == ARGV[1] then return redis.call("del",KEYS[1]) else return 0 end';
    return $redis->eval($script, [$key, $random], 1);
}

function decrGoodsStockNums()
{
    $redis = getRedis();

    //获取商品库存数
    $ret = $redis->get('goods_stock_nums');

    if ($ret === false) {
        return false;
    }

    if ($ret <= 0) {
        return false;
    }
 
    $random = mt_rand();
    //先获取锁
    if (lock('goods_stock_nums_lock', $random)) {
        //修改库存数
        $redis->decr('goods_stock_nums');

        //释放锁
        unlock('goods_stock_nums_lock', $random);
        return true;
    } else {
        usleep(100);
        decrGoodsStockNums();
    }
}

decrGoodsStockNums();
```

### 分布式锁 
>***问题主要集中在如何获取分布式锁***
> setnx创建锁成功 或者 锁过期且getset时间为过期时间

```php
/**
     * 实现Redis分布锁
     */
    $key        = 'demo';       //要更新信息的缓存KEY
    $lockKey    = 'lock:'.$key; //设置锁KEY
    $lockExpire = 10;           //设置锁的有效期为10秒，防止死锁
     
    //获取缓存信息
    $result = $redis->get($key);
    //判断缓存中是否有数据
    if(empty($result))
    {
        $status = TRUE;
        while ($status)
        {
            //设置锁值为当前时间戳 + 有效期
            $lockValue = time() + $lockExpire;
            /**
             * 创建锁
             * 试图以$lockKey为key创建一个缓存,value值为当前时间戳
             * 由于setnx()函数只有在不存在当前key的缓存时才会创建成功
             * 所以，用此函数就可以判断当前执行的操作是否已经有其他进程在执行了
             * @var [type]
             */
            $lock = $redis->setnx($lockKey, $lockValue);
            /**
             * 满足两个条件中的一个即可进行操作
             * 1、上面一步创建锁成功;
             * 2、   1）判断锁的值（时间戳）是否小于当前时间    $redis->get()
             *      2）同时给锁设置新值成功    $redis->getset()
             */
            if(!empty($lock) || ($redis->get($lockKey) < time() && $redis->getSet($lockKey, $lockValue) < time() ))
            {
                //给锁设置生存时间
                $redis->expire($lockKey, $lockExpire);
                //******************************
                //此处执行插入、更新缓存操作...
                //******************************
     
                //以上程序走完删除锁
                //检测锁是否过期，过期锁没必要删除
                if($redis->ttl($lockKey))
                    $redis->del($lockKey);
                $status = FALSE;
            }else{
                /**
                 * 如果存在有效锁这里做相应处理
                 *      等待当前操作完成再执行此次请求
                 *      直接返回
                 */
                sleep(2);//等待2秒后再尝试执行操作
            }
        }
    }
```

>redis锁（setnx）有一定的安全风险，redis挂掉。第一个客户端没释放锁，第二个客户端就获取到锁
>持久化 rdb 镜像复制 aof 增量复制 1、每操作 2、每时间段 3、每cpubuffer 都有空窗期
>redis集群默认是弱一致性，开启强一致性 1、每操作 降低性能
>zookeeper是解决分布式锁的较好方案（zookeeper 安全性能非最优）
>大数据etcd 安全性能优

### 基于golang实现

```golang
package main

import (
    "fmt"
    "sync"
    "time"

    "github.com/go-redis/redis"
)

var client *redis.Client

func init() {
    c := redis.NewClient(&redis.Options{
        Addr:     "node01:6379",
        Password: "",
        DB:       0,
        PoolSize: 100,
    })
    client = c
}

func incr() {

    var lockKey = "counter_lock"
    var counterKey = "counter"

    resp := client.SetNX(lockKey, 1, time.Second*5)
    lockSuccess, err := resp.Result()
    if err != nil || !lockSuccess {
        fmt.Println(err, "lock result: ", lockSuccess)
        //incr()
        return
    }
    getResp := client.Get(counterKey)
    cntValue, err := getResp.Int64()
    if err == nil || err == redis.Nil {
        cntValue++
        resp := client.Set(counterKey, cntValue, 0)
        _, err := resp.Result()
        if err != nil {
            panic("set value error!")
        }
    }
    delResp := client.Del(lockKey)
    unlockSuccess, err := delResp.Result()
    if err == nil && unlockSuccess > 0 {
    } else {
        panic("unlock failed")

    }
}

func main() {
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            incr()
        }()
    }
    wg.Wait()
    getResp := client.Get("counter")
    cntValue, err := getResp.Int64()
    if err == nil || err == redis.Nil {
        println(cntValue)
    }
}
```

