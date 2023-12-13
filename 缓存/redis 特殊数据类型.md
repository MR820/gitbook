### Bitmap
- setbit
- getbit
- bitcount
```shell
setbit key offset value
```
其中 offset 必须是数字，value 只能是 0 或者 1
这个命令的返回值是修改前的值

```shell
setbit byte0 0 1;
setbit byte0 2 1;
setbit byte0 5 1;

setbit byte1 1 1;
setbit byte1 4 1;
```

对应内存中的值：
![](https://oss.wyxxt.org.cn/images/2021/09/18/e3478a7dd19a089ee005407aef8e963e.png)

可以看出 bit 的默认值是 0，那么 Bitmap 在实际开发的运用呢？

这里举一个例子：储存用户在线状态

这里只需要一个 key，然后把用户 ID 作为 offset，如果在线就设置为 1，不在线就设置为 0

```shell
//设置在线状态
$redis->setBit online 0 1;

//设置离线状态
$redis->setBit online 0 0;

//获取状态
$redis->getBit online 0;

//获取在线人数
$redis->bitCount online;
```

### Geo
- geoadd
- geopos
- geodist
- georadius
- georadiusbymember
- geohash

#### GEOADD
```shell
GEOADD key longitude latitude member [longitude latitude member ...]
```
将给定的空间元素（纬度、经度、名字）添加到指定的键里面
这些数据会以zset的结构被储存在键里面

比如设置

key为Sicily（意大利亚西西里岛屿）

member1为Palermo（意大利西西里自治区首府 巴勒莫）

member2为Catania（意大利西西里大区卡塔尼亚省首府 卡塔尼亚）

```shell
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
```

#### GEOPOS
```shell
GEOPOS key member [member ...]
```
返回经纬度

#### GEODIST
```shell
GEODIST key member1 member2 [unit]
```
返回两个位置之间的距离,unit必须为m km mi英里 或 ft英尺

#### GEORADIUS
```shell
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]
```
以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素

- WITHDIST

在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回

距离的单位和用户给定的范围单位保持一致

- WITHCOORD

将位置元素的经度和维度也一并返回

```shell
redis> GEORADIUS Sicily 15 37 200 km WITHDIST
1) 1) "Palermo"
 2) "190.4424"
2) 1) "Catania"
 2) "56.4413"
```

### HyperLogLog
这个结构可以非常省内存的去统计各种计数，比如注册 IP 数、每日访问 IP 数、页面实时UV、在线用户数等

但是它也有局限性，就是只能统计数量，而没办法去知道具体的内容是什么
- PFADD
- PFCOUNT
- PFMERGE
#### PFADD

```shell
redis> PFADD databases "Redis" "MongoDB" "MySQL"
(integer) 1

redis> PFADD databases "Redis"  # Redis 已经存在，不必对估计数量进行更新
(integer) 0
```

#### PFCOUNT

```shell
redis> PFCOUNT databases
(integer) 3
```

### PFMERGE
```shell
PFMERGE destkey sourcekey [sourcekey ...]
```

将多个 HyperLogLog 合并为一个 HyperLogLog

合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合的并集

合并得出的 HyperLogLog 会被储存在 destkey 键里面

如果该键并不存在，那么命令在执行之前， 会先为该键创建一个空的 HyperLogLog

```shell
redis> PFADD nosql "Redis" "MongoDB" "Memcached"
(integer) 1
redis> PFADD RDBMS "MySQL" "MSSQL" "PostgreSQL"
(integer) 1
redis> PFMERGE databases nosql RDBMS
OK
redis> PFCOUNT databases
(integer) 6
```