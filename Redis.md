
性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。

[参考](https://blog.csdn.net/mrleeapple/article/details/78620220)  
####  业务使用方式
hash sets: 关注列表, 粉丝列表, 双向关注列表(key-value(field), 排序)

string(counter): 微博数, 粉丝数, ...(避免了select count(*) from ...)

sort sets(自动排序): TopN, 热门微博等, 自动排序

lists(queue): push/sub提醒,...

#### 大数据表的存储
（eg：140字微博的存储）
一个库就存唯一性id和140个字；
另一个库存id和用户名，发布日期、点击数等信息，用来计算、排序等，等计算出最后需要展示的数据时再到第一个库中提取微博内容；

#### 经验总结
新浪微博响应时间超时目前设置为5s；（返回很慢的记录key，需记录下来分析，慢日志）；
redis不用读写分离，每个请求都是单线程，为什么要进行读写分离。

### [redis索引的设计](https://blog.csdn.net/qq_16414307/article/details/50505084)  
而范围检索，或者非唯一索引，则要使用redis 的 zset来实现。
9a,2

### Redis基础
[redis官方网站中文版](http://www.redis.cn/)  
#### [数据类型](http://www.redis.cn/topics/data-types-intro.html)
字符串：二进制安全，可以包含任意数据类型，如图片、串行化对象
列表：类似字符串数组（链表），按插入顺序排序
集合：无序字符串合集，重复元素自动合并
排序集合：集合上增加了一个排序字段score，是可重复的值，代表顺序
哈希：配成对儿的字符串(field/value)

### 自增计数器
[参考](https://blog.csdn.net/alexhendar/article/details/48315935)  
```
redis> SET page_view 20
OK

redis> INCR page_view
(integer) 21

redis> GET page_view    # 数字值在 Redis 中以字符串的形式保存
"21"
```

### Redis体验
#### 安装
ubuntu16.4(vm：u1601)：
```
sudo apt install redis-server
redis-server   (启动redis服务器)
redis-cli      (在另外的终端窗口中启动)
127.0.0.1:6379>
```
简单读写redis：
```
127.0.0.1:6379> set key1 v1
OK
127.0.0.1:6379> get key1
"v1"            (字符串)
127.0.0.1:6379> set key1 33
OK
127.0.0.1:6379> get key1
(integer) 33    (整数)
```
看上去Redis没有表的概念，只有键（key）；值的类型也是自动识别出来的。识别出整数后，可以用incr命令增加这个值，incr命令针对到字符串类型的值就报错了。

#### [哈希](http://www.runoob.com/redis/redis-hashes.html)
H开头的几个命令可以向同一个key中设置多个值，类似于结构体，如：
```
127.0.0.1:6379>  HMSET key2 name "redis tutorial" description "redis basic commands for caching" likes 20 visitors 23000
OK
127.0.0.1:6379>  HGETALL key2
1) "name"
2) "redis tutorial"
3) "description"
4) "redis basic commands for caching"
5) "likes"
6) "20"
7) "visitors"
8) "23000"
```
#### [列表](http://www.runoob.com/redis/redis-lists.html)
向同一个key中添加多个值，按插入顺序排序。
```
127.0.0.1:6379> lpush key4 ss
(integer) 1
127.0.0.1:6379> lpush key4 33
(integer) 2              （2是序号）
127.0.0.1:6379> lindex key4 1
"ss"            （根据序号取值）
127.0.0.1:6379> rpush key4 wbwang wang webb
(integer) 5                 (右侧插入多个值)
```
#### [集合](http://www.runoob.com/redis/redis-sets.html)
无序，值唯一，有点像枚举值。
```
redis 127.0.0.1:6379> SADD runoobkey redis
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mongodb
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 0
redis 127.0.0.1:6379> SMEMBERS runoobkey

1) "mysql"
2) "mongodb"
3) "redis"
```

#### [有序集合](http://www.runoob.com/redis/redis-sorted-sets.html)
在集合基础上增加了额外的排序字段
```
redis 127.0.0.1:6379> ZADD key5 1 redis
(integer) 1
redis 127.0.0.1:6379> ZADD key5 3 mysql
(integer) 0
redis 127.0.0.1:6379> ZADD key5 4 mysql
(integer) 0
127.0.0.1:6379> ZRANGE key5 0 10 WITHSCORES
1) "redis"
2) "1"
3) "mysql"
4) "4"
```
#### [基数估计(HyperLogLog)](http://www.runoob.com/redis/redis-hyperloglog.html)
构造一个可能出现重复元素的集合（如几亿元素），则估算基数（不重复的元素数）需要很大的内存和计算量，Redis提供了HyperLogLog命令来在误差可接受的范围内快速估算基数。
```
redis 127.0.0.1:6379> PFADD runoobkey "redis"

1) (integer) 1

redis 127.0.0.1:6379> PFADD runoobkey "mongodb"

1) (integer) 1

redis 127.0.0.1:6379> PFADD runoobkey "mysql"

1) (integer) 1

redis 127.0.0.1:6379> PFCOUNT runoobkey

(integer) 3
```
