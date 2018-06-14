
性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。

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
对于非范围唯一索引，我们可以简单的把索引也存成KV对，v保存主key即可，而范围检索，或者非唯一索引，则要使用redis的zset来实现。


### [Redis数据类型介绍(重点是有序集合)](http://www.redis.cn/topics/data-types-intro.html)
字符串：二进制安全，可以包含任意数据类型，如图片、串行化对象
列表：类似字符串数组（链表），按插入顺序排序
集合：无序字符串合集，重复元素自动合并
排序集合：集合上增加了一个排序字段score，是可重复的值，代表顺序
哈希：配成对儿的字符串(field/value)

原文中关于有序集合部分是英文的，下面是其中文翻译：

#### Redis有序集合

有序集合是一种类似于Set和Hash之间的混合的数据类型。与集合类似，有序集合由唯一的非重复字符串元素组成，因此在某种意义上，有序集合也是集合。

然而，虽然集合中的元素没有排序，但有序集合中的每个元素都与称为*分数（score）*的浮点数值相关联（这就是为什么这个类型也与哈希类似，因为每个元素都映射到一个值）。

而且，有序集合中的元素按*顺序排列*（因此它们不按请求排序，排序是用于表示有序集合的数据结构的特性）。他们根据以下规则进行排序：

- 如果A和B是两个具有不同分数的元素，如果A.score> B.score，则A> B。
- 如果A和B具有完全相同的分数，如果A字符串按字典顺序大于B字符串，则A> B。A和B字符串不能相等，因为已排序的集合仅允许唯一的元素。

让我们从一个简单的例子开始，添加一些黑客名称作为有序集合的元素，其出生年份为“分数”。

```
> zadd hackers 1940 "Alan Kay"
(integer) 1
> zadd hackers 1957 "Sophie Wilson"
(integer 1)
> zadd hackers 1953 "Richard Stallman"
(integer) 1
> zadd hackers 1949 "Anita Borg"
(integer) 1
> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
> zadd hackers 1916 "Claude Shannon"
(integer) 1
> zadd hackers 1969 "Linus Torvalds"
(integer) 1
> zadd hackers 1912 "Alan Turing"
(integer) 1
```

正如你所看到的`ZADD`是类似的`SADD`，但需要一个额外的参数（放置在要添加的元素之前），这是分数。`ZADD`也是可变的，所以你可以自由地指定多个分值对，即使这在上面的例子中没有使用。

对于有序集合，返回按出生年份排序的黑客列表是微不足道的，因为实际上*他们已经被排序*。

实现注意事项：有序集合是通过包含跳过列表和哈希列表的双重数据结构实现的，因此每次添加元素Redis都会执行O(log(N))操作。这很好，但是当我们要求排序元素时，Redis根本不需要做任何工作，它已经全部排序：

```
> zrange hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
6) "Richard Stallman"
7) "Sophie Wilson"
8) "Yukihiro Matsumoto"
9) "Linus Torvalds"
```

注意：0和-1意味着从元素索引0到最后一个元素（这里的-1的作用就像它在`LRANGE`命令中的作用一样）。

如果我想反序排列他们，从年轻到最老呢？使用[ZREVRANGE](http://www.redis.cn/commands/zrevrange)而不是[ZRANGE](http://www.redis.cn/commands/zrange)：

```
> zrevrange hackers 0 -1
1) "Linus Torvalds"
2) "Yukihiro Matsumoto"
3) "Sophie Wilson"
4) "Richard Stallman"
5) "Anita Borg"
6) "Alan Kay"
7) "Claude Shannon"
8) "Hedy Lamarr"
9) "Alan Turing"
```

使用`WITHSCORES`参数还可以返回分数：

```
> zrange hackers 0 -1 withscores
1) "Alan Turing"
2) "1912"
3) "Hedy Lamarr"
4) "1914"
5) "Claude Shannon"
6) "1916"
7) "Alan Kay"
8) "1940"
9) "Anita Borg"
10) "1949"
11) "Richard Stallman"
12) "1953"
13) "Sophie Wilson"
14) "1957"
15) "Yukihiro Matsumoto"
16) "1965"
17) "Linus Torvalds"
18) "1969"
```

#### 操控范围

有序集合比这更强大，它可以操控范围。让我们得到所有1950年以前出生的人。我们使用该`ZRANGEBYSCORE`命令来执行此操作：

```
> zrangebyscore hackers -inf 1950
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
```

我们要求Redis以负无穷大到1950年的分数返回所有元素（包括两个极值）。

也可以按范围删除元素。让我们从有序集合中删除1940年至1960年间出生的所有黑客：

```
> zremrangebyscore hackers 1940 1960
(integer) 4
```

`ZREMRANGEBYSCORE` 可能不是最好的命令名称，但它可能非常有用，并返回已删除元素的数量。

为有序集合元素定义的另一个非常有用的操作是get-rank。可以询问一组元素中元素的位置。

```
> zrank hackers "Anita Borg"
(integer) 4
```

`ZREVRANK`命令按降序获得排名。

#### 字典分数

使用Redis 2.8的最新版本，引入了一项新功能，允许按照字典顺序获取范围，假定有序集合中的元素都以相同的分数插入（元素与C语言库的 `memcmp`函数进行比较，因此可以保证没有排序，并且每个Redis实例将以相同的输出回复）。（C库函数memcmp是比较内存区域buf1和buf2的前count个字节。该函数是按字节比较的。）

字典化范围操作主命令是`ZRANGEBYLEX`、`ZREVRANGEBYLEX`、`ZREMRANGEBYLEX`和`ZLEXCOUNT`。

例如，让我们再次添加我们的著名黑客列表，但是这次对所有元素使用零分数：

```
> zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0
  "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon"
  0 "Linus Torvalds" 0 "Alan Turing"
```

根据有序集合排序规则，它们已经按字典顺序排序：

```
> zrange hackers 0 -1
1) "Alan Kay"
2) "Alan Turing"
3) "Anita Borg"
4) "Claude Shannon"
5) "Hedy Lamarr"
6) "Linus Torvalds"
7) "Richard Stallman"
8) "Sophie Wilson"
9) "Yukihiro Matsumoto"
```

使用`ZRANGEBYLEX`我们可以要求词典范围：

```
> zrangebylex hackers [B [P
1) "Claude Shannon"
2) "Hedy Lamarr"
3) "Linus Torvalds"
```

范围可以是包含的或不包含的（取决于第一个字符），也可以用字符串`+`和`-`字符串分别指定字符串无限大和负无限大。有关更多信息，请参阅文档。

此功能非常重要，因为它使我们能够使用有序集作为通用索引。例如，如果要通过128位无符号整数参数为元素编制索引，则只需将元素添加到具有相同分数（例如0）的有序集合中，但是将8个字节的前缀组成**高位优先的128位数字**。由于高位优先的数字按字典顺序排列（按原始字节顺序排列）实际上也是以数字顺序排列的，因此您可以在128位空间中请求范围，并获取元素值以舍弃前缀。

如果您想在更严格的演示环境中查看该功能，请查看[Redis自动填充演示](http://autocomplete.redis.io/)。

#### 更新分数：排行榜

在切换到下一个主题之前，最后一个关于有序集合的提示。有序集合的分数可以随时更新。对已包括在有序集合中的元素调用`ZADD`命令，将会以O(log(N )) 时间复杂度更新其分数（和位置）。因此，当有大量更新时，有序集合很适合。

由于这个特点，一个常见的用例就是排行榜。典型的应用程序是Facebook游戏，您可以将按照用户的最高分数和等级做组合，以显示前N位用户，以及用户在排行榜中的排名（例如“你是＃4932最好成绩“）。



### 自增计数器
[参考：Redis自增实现计数](https://blog.csdn.net/alexhendar/article/details/48315935)  
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
redis 127.0.0.1:6379> ZADD key5 4 mysql    (将分数由3改成了4)
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
#### 索引
增加了两个哈希，key是`usr:1`和`usr:2`：
```
hmset usr:1 uid 1 name aaa credit 10 type 0
hmset usr:2 uid 2 name bbb credit 20 type 1
hmset usr:3 uid 3 name ccc credit 25 type 1
```
如何快速检索出type是1的用户？办法是利用redis的有序集合（zset）结构，手工维护专门的索引：  
```
zadd usr.index.type 0 0:1
zadd usr.index.type 0 1:2
zadd usr.index.type 0 1:3
```
冒号前面的是type，后面的是uid。  
```
zrangebylex usr.index.type [1: (1;
1) "1:2"
2) "1:3"
```
方括号表示`包含`，括号表示`不包含`。而中间的空格隔开了两个范围参数，表示按字母顺序大于（可能包含）第一个参数，小于（可能包含）第二个参数。第一个参数用`-`表示字母顺序的无穷小，第二个参数用`+`表示字母顺序的无穷大。

分号（0x3B）在ascii码表中紧挨在冒号（0x3A）后面，所以上面使用了分号表示。也可以直接写成`[1: (1\0x3B`，只是不更好理解。

参考：[Redis 在新浪微博中的应用](https://blog.csdn.net/mrleeapple/article/details/78620220)  
