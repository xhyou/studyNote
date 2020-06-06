# 安装redis

> [1.redis下载官网](https://github.com/antirez/redis)
>
> 2.在linux下解压下载的安装包
>
> 3.执行yum -install gcc-c++
>
> 4.执行make命令
>
> 5.执行mack install命令

# 排坑

> 在执行make命令是报错：
>
> make: *** 没有规则可以创建目标“makefile“等错误
>
> server.c:5118:176: 错误：‘struct redisServer’没有名为‘maxmemory’的成员

解决思路:

升级gcc版本：

> 查看gcc版本是否在5.3以上，centos7.6默认安装4.8.5
>
> `$ `gcc -v
>
> \# 升级gcc到5.3及以上,如下：
>
> 升级到gcc 9.3：
>
> `　$ `yum -y install centos-release-scl
>
> `$ `yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
>
> `$ `scl enable devtoolset-9 bash
>
> 需要注意的是scl命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本。如果要长期使用gcc 9.3的话
>
> `$ `echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
>
> 升级完之后再次编译即可

# redis的使用

1.redis的默认安装路径为

>  /usr/local/bin

![redis的安装路径](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200516114625945.png)

2.将安装redis文件夹下的redis.config配置文件拷贝到/usr/local/bin/自定义下的文件夹下

![copy配置文件](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200516114926117.png)

3.配置redis.config为后台启动

![配置redis.config](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200516115232686.png)

4.启动服务端

```shell
 redis-server  xconfig/redis.conf
```

5.启动客户端

```shell
redis-cli -p 6379
```

6.测试连接

![链接测试](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200516115806751.png)

# 五大数据类型

> 基本案例:

```bash
127.0.0.1:6379> keys *   # 查看所有的key
(empty array)
127.0.0.1:6379> set name demo # 设置key
OK
127.0.0.1:6379> get name
"demo"
127.0.0.1:6379> exists name #判断name是否存在,存在返回1
(integer) 1
127.0.0.1:6379> exists age
(integer) 0
127.0.0.1:6379> EXPIRE name 10  # 设置某个key的过期时间
(integer) 1
127.0.0.1:6379> ttl name  # 读取还有多久过期
(integer) 4
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> move name 1 #将当前库的key移到指定库中
127.0.0.1:6379> type name # 查看当前key的类型
127.0.0.1:6379> String
```

## String

```bash
127.0.0.1:6379> set name zhangsan #设置key的值
OK
127.0.0.1:6379> APPEND name ",nihao" #往这个key后追加值
(integer) 14
127.0.0.1:6379> STRLEN name   #查看某个key的长度
(integer) 14
127.0.0.1:6379> APPEND name1 jingdong #往key后追加值如果不存在,则为新增这个key
(integer) 8
#####################递增范围##################################
127.0.0.1:6379> set count 0  #设置初始化的数值
OK
127.0.0.1:6379> INCR count   #设置每次自增1
(integer) 1
127.0.0.1:6379> DECR count   #设置每次递减1
(integer) 1
127.0.0.1:6379> INCRBY count 10  #设置每次递增10
(integer) 11
127.0.0.1:6379> DECRBY count 10  #设置每次递减10
(integer) 1
#####################字符串范围##################################
127.0.0.1:6379> GETRANGE name 0 -1 #获取字符串的所有区间
"zhangsan,nihao"
127.0.0.1:6379> GETRANGE name 0 3  #获取字符串的指定取值范围
"zhan"
# 替换
127.0.0.1:6379> get name
"zhangsan,nihao"
127.0.0.1:6379> SETRANGE name 1 li #替换指定位置的字符串
(integer) 14
127.0.0.1:6379> get name
"zlingsan,nihao"
#####################字符串设置##################################
# setex (set with expire) 设置过期时间 
127.0.0.1:6379> setex name 10 nihao
OK
127.0.0.1:6379> get name
"nihao"
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> get name
(nil)
# setnx (set if not exist)(分布式锁中经常会使用) 不存在设置 如果存在不会覆盖还是使用之前存在的值
127.0.0.1:6379> keys *
1) "key1"
127.0.0.1:6379> get key1
"zhangsan"
127.0.0.1:6379> setnx key1 lisi
(integer) 0
127.0.0.1:6379> get key1
"zhangsan"
#####################批量设置##################################
#mset与mget:批量新增与批量获取
127.0.0.1:6379> set k1 v1 
OK
127.0.0.1:6379> mset k1 v11 k2 v2 k3 v3 #批量新增 如果存在了 覆盖
OK
127.0.0.1:6379> mget k1 k2 k3 #批量获取
1) "v11"
2) "v2"
3) "v3"
#msetnx 先判断是否存在,在批量设置值
127.0.0.1:6379> msetnx k4 v4 k5 v5 #是一组原子性操作,只有当所有的都不存在,才能执行成功
(integer) 1
127.0.0.1:6379> mget k4 k5
1) "v4"
2) "v5"
#复杂的key设置 user:{id}:{filed}
127.0.0.1:6379> msetnx user:1:name zhangsan user:1:age 12
(integer) 1
127.0.0.1:6379> mget user:1:name user:1:age
1) "zhangsan"
2) "12"
#####################先获取在设置##################################
127.0.0.1:6379> getset name zhangsan #先获取key=name的值,如果不存在返回nil
(nil)
127.0.0.1:6379> getset name lisi  #先获取key=name的值,如果存在打印
"zhangsan"
127.0.0.1:6379> get name
"lisi"

```

## List

```bash
#####################插入##################################
127.0.0.1:6379> lpush list one #将一个值或者多个值插入列表的头部(左)
(integer) 1
127.0.0.1:6379> lpush list two
(integer) 2
127.0.0.1:6379> lpush list three
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> rpush list four #将一个值或者多个值插入列表的尾部(右)
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "four"
#####################取出##################################
127.0.0.1:6379> LPOP list  #移除list中的第一个元素
"three"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
3) "four"
127.0.0.1:6379> RPOP list  #移除list中的最后一个元素
"four"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
#####################获取指定的下标的值##################################
127.0.0.1:6379> LRANGE list 0 -1 
1) "two"
2) "one"
127.0.0.1:6379> LINDEX list 0 #获取第一个下标的值
"two"
#####################获取指定list的长度##################################
127.0.0.1:6379> LLEN list 
(integer) 2
#####################移除list中指定的个数##################################
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "three"
3) "three"
4) "two"
5) "one"
127.0.0.1:6379> lrem list 3 three #移除list中key为three的三个
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
#####################list的截取操作##################################
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> LTRIM list 0 0 #截取list中的第0个位置的数据
OK
127.0.0.1:6379> lrange list 0 -1
1) "two"
#############移除列表的最后一个元素插入到新的列表#########################
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> RPOPLPUSH list list01 #将list中的元素尾部取出一个放入list01中
"one"
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
127.0.0.1:6379> lrange list01 0 -1
1) "one"
#####################替换操作##################################
127.0.0.1:6379> lrange list  0 -1
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> lset list 0 four #替换指定的值，如果存在的话
OK
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "two"
3) "one"
127.0.0.1:6379> LSET list 3 five #替换指定的值，不存在则异常
(error) ERR index out of range
#####################插入操作##################################
127.0.0.1:6379> LRANGE list 0 -1
1) "hello"
2) "word"
127.0.0.1:6379> LINSERT list before word nihao #前插入,往word前面插入一个nihao
(integer) 3
127.0.0.1:6379> Linsert list after word world #后插入,往word前面插入一个world
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "hello"
2) "nihao"
3) "word"
4) "world"
```

## Set

```bash
#####################基本操作##################################
127.0.0.1:6379> sadd mset hello #插入操作
(integer) 1
127.0.0.1:6379> sadd mset word
(integer) 1
127.0.0.1:6379> smembers mset #查询所有操作
1) "word"
2) "hello"
127.0.0.1:6379> sismember mset word #判断是否存在操作
(integer) 1
#####################查看元素的个数##################################
127.0.0.1:6379> scard mset 
(integer) 2
#####################移除元素##################################
127.0.0.1:6379> SREM mset hello #移除指定的key元素
(integer) 1
127.0.0.1:6379> SMEMBERS mset
1) "word"
#####################取出随机的元素##################################
127.0.0.1:6379> SMEMBERS mset
1) "word"
2) "go"
3) "hello"
127.0.0.1:6379> SRANDMEMBER mset # 随机取出一个数
"go"
127.0.0.1:6379> SRANDMEMBER mset
"hello"
127.0.0.1:6379> SRANDMEMBER mset 2 # 随机取出指定的数
1) "go"
2) "hello"
#####################随机移除的元素##################################
127.0.0.1:6379> SMEMBERS mset
1) "word"
2) "go"
3) "hello"
127.0.0.1:6379> SPOP mset #随机移除集合中的指定个数
"go"
127.0.0.1:6379> SMEMBERS mset
1) "word"
2) "hello"
####################移动指定的元素到另外一个集合中########################
127.0.0.1:6379> SMEMBERS mset
1) "word"
2) "hello"
127.0.0.1:6379> SMOVE mset mset1 word
(integer) 1
127.0.0.1:6379> SMEMBERS mset
1) "hello"
127.0.0.1:6379> SMEMBERS mset1
1) "word"
####################交集 差集 并集#######################################
127.0.0.1:6379> SMEMBERS mset1
1) "c"
2) "b"
3) "a"
127.0.0.1:6379> SMEMBERS mset2
1) "c"
2) "e"
3) "d"
127.0.0.1:6379> SDIFF mset1 mset2 #取mset1的差集
1) "a"
2) "b"
127.0.0.1:6379> SINTER mset1 mset2 #取mset1的交集
1) "c"
127.0.0.1:6379> SUNION mset1 mset2 #取mset1的并集
1) "c"
2) "a"
3) "b"
4) "e"
5) "d"
```

## Hash

```bash
#####################基本操作##################################
127.0.0.1:6379> hset mset01 name zhangsan #添加
(integer) 1
127.0.0.1:6379> hget mset01 name
"zhangsan"
127.0.0.1:6379> hmset mset01 name zhangsan age 14 # 添加多个
OK
127.0.0.1:6379> hmget mset01 name age #获取多个
1) "zhangsan"
2) "14"
127.0.0.1:6379> hgetall mset01 #获取全部
1) "name"
2) "zhangsan"
3) "age"
4) "14"
#####################删除操作##################################
127.0.0.1:6379> hgetall mset01
1) "name"
2) "zhangsan"
3) "age"
4) "14"
127.0.0.1:6379> HDEL mset01 name #删除指定的key 并且value也删除
(integer) 1
127.0.0.1:6379> HGETALL mset01
1) "age"
2) "14"
#####################获取长度##################################
127.0.0.1:6379> hgetall mset01
1) "age"
2) "14"
127.0.0.1:6379> hlen mset01 #获取指定的长度
(integer) 1
#####################判断是否存在##############################
127.0.0.1:6379> hgetall mset01
1) "age"
2) "14"
127.0.0.1:6379> HEXISTS mset01 age #判断是否存在 存在返回1
(integer) 1
127.0.0.1:6379> HEXISTS mset01 name
(integer) 0
#####################只获取value或者key的值########################
127.0.0.1:6379> HKEYS mset01 #获取所有的key
1) "age"
127.0.0.1:6379> HVALS mset01 #获取所有的value
1) "14"
#####################指定增量/批量新增########################
127.0.0.1:6379> hset mset01 count 0
(integer) 1
127.0.0.1:6379> HINCRBY mset01 count 10  #递增
(integer) 10 
127.0.0.1:6379> HINCRBY mset01 count -10 #递减
(integer) 0
127.0.0.1:6379> HSETNX mset01 count 01 #如果存在新增失败
(integer) 0
127.0.0.1:6379> HSETNX mset01 count1 01
(integer) 1
```

## Zset(有序集合)

```bash
#####################基本操作##################################
127.0.0.1:6379> zadd zset01 2500 xh #添加一个元素
(integer) 1
127.0.0.1:6379> zadd zset01 2500 xh 2600 xm  #添加多个元素
(integer) 1
127.0.0.1:6379> ZRANGE zset01 0 -1 #计算元素的区间
1) "xh"
2) "xm"
127.0.0.1:6379> ZRANGE zset01 0 -1 withscores #计算元素的带上分数
1) "xh"
2) "2500"
3) "xm"
4) "2600"
#####################排序操作##################################
127.0.0.1:6379> ZRANGE zset01 0 -1 withscores
1) "xh"
2) "2500"
3) "xm"
4) "2600"
127.0.0.1:6379> ZRANGEBYSCORE zset01 -inf +inf
1) "xh"
2) "xm"
127.0.0.1:6379> ZRANGEBYSCORE zset01 -inf +inf withscores #排序带上分数
1) "xh"
2) "2500"
3) "xm"
4) "2600"
127.0.0.1:6379> ZRANGEBYSCORE zset01 -inf 2500  withscores #查询指定的分数范围 
1) "xh"
2) "2500"
127.0.0.1:6379> ZREVRANGEBYSCORE zset01 +inf -inf withscores #降序排序
#####################移除操作##################################
127.0.0.1:6379> ZRANGE zset01 0 -1
1) "xh"
2) "xm"
127.0.0.1:6379> ZREM zset01 xm
(integer) 1
127.0.0.1:6379> ZRANGE zset01 0 -1
1) "xh"
#####################获取个数##################################
127.0.0.1:6379> ZCARD zset01
(integer) 1
#####################获取区间中的个数##################################
127.0.0.1:6379> zrange zset01 0 -1
1) "hello"
2) "world"
3) "people"
127.0.0.1:6379> zcount zset01 1 2
(integer) 2
```

# 三种特殊的数据类型

## geospatial(地理位置)

> 用途:朋友的定位,附件的人,打车的距离 
>
> 地址位置经纬度查询工具:http://www.jsons.cn/lngcode/

```bash
GEOADD：#添加地理位置
# 有效的地理位置:经度-180到180 纬度:-85.05112878度到85.05112878度
# 当地理位置超出的时候,会返回一个错误
  #127.0.0.1:6379>geoadd china:city 39.90 116.40 beijing
  # error ERR invalid longitude ,latitude pair 39.000000,116.4000000
  
#####################设置每个城市的经纬度###########################  
127.0.0.1:6379> GEOADD china:city 117.60 24.46 xiamen #设置指定城市的经纬度
(integer) 1
127.0.0.1:6379> geoadd china:city 119.2 26.04 fuzhou
(integer) 1
#####################获取每个城市的经纬度########################### 
127.0.0.1:6379> GEOPOS china:city xiamen chongqi #获取指定城市的经纬度
1) 1) "117.59999781847000122"
   2) "24.45999962167363861"
2) (nil)  #没有返回空
#####################返回两个给定位置之间的距离###########################
如果两个位置之间的其中一个不存在， 那么命令返回空值。
指定单位的参数 unit 必须是以下单位的其中一个：
    m 表示单位为米。
    km 表示单位为千米。
    mi 表示单位为英里。
    ft 表示单位为英尺。
127.0.0.1:6379> GEODIST china:city xiamen fuzhou km #查看两地的直线距离
"238.3033"
#####################查找指定位置附近距离###########################
127.0.0.1:6379> GEORADIUS china:city 117 24  1000 km  #查找当前精度范围1000km内的城市
1) "xiamen"
2) "fuzhou"
127.0.0.1:6379> GEORADIUS china:city 117 24  5000 km withdist #显示到中心距离的位置 
1) 1) "xiamen"
   2) "79.5062" #厦门到该中心位置79km
2) 1) "fuzhou"
   2) "317.2402"
127.0.0.1:6379> GEORADIUS china:city 117 24  5000 km withcoord #显示他人的定位信息
1) 1) "xiamen"
   2) 1) "117.59999781847000122"
      2) "24.45999962167363861"
2) 1) "fuzhou"
   2) 1) "119.19999986886978149"
      2) "26.04000031329945131"
127.0.0.1:6379> GEORADIUS china:city 117 24  5000 km  count 1 #筛选出指定数据
1) "xiamen"
#####################查找指定元素附近的其他元素###########################
 127.0.0.1:6379> GEORADIUSBYMEMBER china:city xiamen 1000 km  #指定元素附近1000km元素
1) "xiamen"
2) "fuzhou"
```

> geospatial:底层是基于zset实现的,所以我们可以使用zset的命令来操作geospatial

```bash
127.0.0.1:6379> ZRANGE china:city 0 -1
1) "xiamen"
2) "fuzhou"
127.0.0.1:6379> ZREM china:city xiamen
(integer) 1
127.0.0.1:6379> ZRANGE china:city 0 -1
1) "fuzhou"
```

## Hyperloglog

> 当一个人多次访问一个网站时候,还是算一个人(做一些访问统计)
>
> 主要做页面统计(前提:需要允许容错)

```bash
127.0.0.1:6379> PFADD mekey a b c d e f g h i j #mekey中插入10条数据
(integer) 1
127.0.0.1:6379> PFCOUNT mekey
(integer) 10
127.0.0.1:6379> PFADD mekey2 i x h f g e #mekey2插入5条数据
(integer) 1
127.0.0.1:6379> PFCOUNT mekey2
(integer) 6
127.0.0.1:6379> PFMERGE mykey3 mekey mekey2  #使用mykey3取mekey mekey2得并集
OK
127.0.0.1:6379> pfcount mykey3 
11 
```

## Bitmaps

> 使用场景:位存储 0 1 活跃不活跃 登录为登录。有两个状态得都可以使用bitmap

> 案例:使用bitmap统计一周的打卡记录 1为打卡 0为未打卡

```bash
127.0.0.1:6379> setbit sign 0 1  #设置bit 0代表星期日 1代表打卡
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 2 0
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> getbit sign 3  #获取指定的bit的值
(integer) 1
127.0.0.1:6379> BITCOUNT sign  #统计打卡(值为1)的个数
(integer) 2
```

# 事务

> Redis的单条命令是原子性的,但是事务不保证原子性
>
> Redis没有隔离级别
>
> Redis事务的本质：一组命令的集合,一个事务中所有的命令都会被序列化,在执行的过程中会按照顺序执行。
>
> 一次性、顺序性、排他性！执行完这些命令
>
> Redis的事务:
>
> * 开启事务(MULTI)
> * 命令入队(……)
> * 执行事务(EXEC)
>
> 

```bash
127.0.0.1:6379> MULTI  #开启事务
OK  
127.0.0.1:6379> set k1 v1  # 命令入队
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> EXEC  #执行事务
1) OK
2) OK
3) "v2"
4) OK
```

放弃事务

```bash
127.0.0.1:6379> MULTI #开启事务
OK
127.0.0.1:6379> set k1 v1 #命令入队
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> DISCARD  #取消事务
OK
127.0.0.1:6379> EXEC  
(error) ERR EXEC without MULTI
127.0.0.1:6379> get k1
(nil)
```

> 编译型异常(代码有问题,命令有错),事务中的所有命令都不会执行

```bash
127.0.0.1:6379> MULTI #开启事务
OK
127.0.0.1:6379> getset k1  #设置编译时异常
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> getset k1 v1
QUEUED
127.0.0.1:6379> exec #结束事务
(error) EXECABORT Transaction discarded because of previous errors. 
#提示错误 整个事务都会回滚

```



> 运行时异常（1/0）如果命令中存在语法错误,那么执行任务时候,其他命令是可以正常执行的,错误命令抛出异常

```bash
127.0.0.1:6379> MULTI #开启事务
OK
127.0.0.1:6379> set k1 "v1"  
QUEUED  
127.0.0.1:6379> INCR k1  #设置运行时异常(字符串不能递增)
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> EXEC  #执行事务
1) OK
2) (error) ERR value is not an integer or out of range #只有异常的抛错其他的可以正常执行
3) OK
4) "v2"
5) "v1"
```

# 乐观锁

> 认为什么时候都不会有问题,所有不会加锁。在更新数据的时候会去判断是否有人修改过这个数据
>
> 获取version
>
> 更新比较的version

```bash
#正常执行的场景
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> WATCH money
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> MULTI 
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY out 20
QUEUED
127.0.0.1:6379> EXEC
1) (integer) 80
2) (integer) 20
```

## 示例

* 客户端1

![image-20200518171241354](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200518171241354.png)

* 客户端2

![image-20200518171427158](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200518171427158.png)

# 实战-jedis

* 导入pom

```xml
<dependencies>
        <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.2.0</version>
        </dependency>
        <!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.62</version>
        </dependency>
    </dependencies>
```

* 测试连接

```java
public class TestJedis {
    //测试jedis
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        System.out.println(jedis.ping());//打印pong说明连接成功
    }
}
```

## 事务的实战

```java
public class TestTX {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        jedis.flushDB();
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("name","张三");
        jsonObject.put("age",14);
        String jsonStr = jsonObject.toString();
        //开启事务
        Transaction multi = jedis.multi();
        try {
            multi.set("user1",jsonStr);
            multi.set("user2",jsonStr);
//            int i=1/0; 设置抛出异常
            multi.exec();

        }catch (Exception e ){
            multi.discard();
            e.printStackTrace();
        }finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();
        }
    }
}
```

## 通用的实战

```java
public class TestKey {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        System.out.println("清空数据:"+jedis.flushDB());
        System.out.println("判断某个数值是否存在:"+jedis.exists("key01"));
        Set<String> keys = jedis.keys("*");
        System.out.println(keys);

        System.out.println("增加某个键:"+jedis.set("key01","value01"));
        System.out.println("增加某个键:"+jedis.set("username","xuehy"));
        System.out.println("***删除某个键***:"+jedis.del("key01"));
        System.out.println("查看username的存储类型:"+jedis.type("xuehy"));
        System.out.println("随机返回一个key:"+jedis.randomKey());
        System.out.println("重命名key:"+jedis.rename("username","name"));
        System.out.println("返回当前数据库中key的数目:"+jedis.dbSize());
    }
}
```

## String的实战

```java
public class TestString {
    public static void main(String[] args) throws InterruptedException {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        jedis.flushDB();
        System.out.println("===增加数据===");
        System.out.println(jedis.set("key1","value1"));//设置基本的数据类型
        System.out.println(jedis.set("key2","value2"));//设置基本的数据类型
        System.out.println(jedis.set("key3","value3"));//设置基本的数据类型
        System.out.println("删除指定的键:"+jedis.del("key1"));//删除指定的键
        System.out.println("获取指定的键:"+jedis.get("key2"));//获取指定的键
        System.out.println("修改指定的键:"+jedis.set("key3","value33"));//修改指定的键
        System.out.println("拼接指定的值:"+jedis.append("key3","hello"));//修改指定的键
        //多个操做
        System.out.println("增加多个键:"+jedis.mset("key01","value01","key02","value02"));
        System.out.println("获取多个值:"+jedis.mget("key01","key02"));
        System.out.println("删除多个键:"+jedis.del("key01","key02"));
        System.out.println("获取多个值:"+jedis.mget("key01","key02"));
        jedis.flushDB();
        //使用setnx 当key不存在时候新增 存在的时候不覆盖
        System.out.println(jedis.setnx("key01","value01"));
        System.out.println(jedis.setnx("key02","value02"));
        System.out.println(jedis.get("key01"));

        System.out.println("===========新增键值对并设置有效时间=============");
        System.out.println(jedis.setex("key03",2,"value03"));
        System.out.println(jedis.get("key03"));
        Thread.sleep(3);
        System.out.println(jedis.get("key03"));
        System.out.println("===========获取原值，更新为新值==========");
        System.out.println(jedis.getSet("key02","value002"));
        System.out.println(jedis.getrange("key02",0,4));
        System.out.println(jedis.keys("*"));
        jedis.flushDB();
        System.out.println("==="+jedis.setrange("key01",0,"li"));
        System.out.println(jedis.get("key01"));
    }
}
```

## List的实战

```java
public class TestList {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        jedis.flushDB();
        System.out.println("===========添加一个list===========");
        jedis.lpush("collections", "ArrayList", "Vector", "Stack", "HashMap", "WeakHashMap", "LinkedHashMap");
        jedis.lpush("collections", "HashSet");
        jedis.lpush("collections", "TreeSet");
        jedis.lpush("collections", "TreeMap");
        System.out.println("collections的值:"+jedis.lrange("collections",0,-1));
        System.out.println("出栈:"+jedis.lpop("collections"));
        System.out.println("获取长度:"+jedis.llen("collections"));
        System.out.println("获取指定位置的值:"+jedis.lindex("collections",0));
        System.out.println("移除指定个数:"+jedis.lrem("collections",2,"HashMap"));
        System.out.println("collections的值:"+jedis.lrange("collections",0,-1));
        System.out.println("截取操作:"+jedis.ltrim("collections",0,2));
        System.out.println("collections的值:"+jedis.lrange("collections",0,-1));
        System.out.println("移除表中的尾部元素插入到新表:"+jedis.rpoplpush("collections","collections1"));
        System.out.println("collections1的值:"+jedis.lrange("collections1",0,-1));
        System.out.println("替换操作:"+jedis.lset("collections1",0,"Nice"));
        System.out.println("collections1的值:"+jedis.lrange("collections1",0,-1));
        System.out.println(jedis.linsert("collections1", ListPosition.BEFORE,"Nice","Good"));
        System.out.println("collections1的值:"+jedis.lrange("collections1",0,-1));
        jedis.flushDB();
        System.out.println("===排序===");
        jedis.lpush("sortedList", "3","6","2","0","7","4");
        System.out.println("sortedList排序前："+jedis.lrange("sortedList", 0, -1));
        System.out.println("sortedList排序后："+jedis.sort("sortedList"));
    }
}
```

## Set的实战

```java
public class T_06_TestSet {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        jedis.flushDB();
        jedis.sadd("set01","e1","e2","e3","e4");
        System.out.println("查询所有的操作:"+jedis.smembers("set01"));
        System.out.println("查询key中的某个元素是否存在:"+jedis.sismember("set01","e1"));
        System.out.println("查看元素的个数:"+jedis.scard("set01"));
        System.out.println("删除指定元素:"+jedis.srem("set01","e1"));
        System.out.println("查询所有的操作:"+jedis.smembers("set01"));
        System.out.println("随机移除集合中的指定个数:"+jedis.spop("set01"));
        System.out.println("查询所有的操作:"+jedis.smembers("set01"));
        System.out.println("移动元素到另外一个Set中:"+jedis.smove("set01","set02","e4"));
        System.out.println("=================================");
        System.out.println(jedis.sadd("eleSet1", "e1","e2","e4","e3","e0","e8","e7","e5"));
        System.out.println(jedis.sadd("eleSet2", "e1","e2","e4","e3","e0","e8"));
        System.out.println("并集:"+jedis.sunion("eleSet1","eleSet2"));
        System.out.println("交集:"+jedis.sinter("eleSet1","eleSet2"));
        System.out.println("差集:"+jedis.sdiff("eleSet1","eleSet2"));
        //取eleSet1,eleSet2的交集放入到eleSet3中
        System.out.println(jedis.sinterstore("eleSet3", "eleSet1", "eleSet2"));
        System.out.println(jedis.smembers("eleSet3"));
    }
}
```

## Hash的实战

```java
public class T_07_TestHash {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        jedis.flushDB();
        HashMap<String,String> map = new HashMap<String, String>();
        map.put("key1","value1");
        map.put("key2","value2");
        map.put("key3","value3");
        map.put("key4","value4");
        jedis.hmset("hash",map);
        System.out.println(jedis.hgetAll("hash"));
        jedis.hset("hash","key5","value5");
        System.out.println("删除指定的key:"+jedis.hdel("hash","key1"));
        System.out.println("获取长度:"+jedis.hlen("hash"));
        System.out.println("是否存在:"+jedis.exists("hash","key1"));
        System.out.println("获取所有的key的值:"+jedis.hkeys("hash"));
        System.out.println("获取所有的value的值:"+jedis.hkeys("hash"));
        System.out.println("获取指定的值:"+jedis.hget("hash","key1"));
    }
}
```

## Zset的实战

```java
public class T_08_TestZset {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        jedis.flushDB();
        Map<String,Double> map = new HashMap();
        map.put("xh",2700.0);
        map.put("xl",2800.0);
        map.put("xz",2400.0);
        map.put("xf",3000.0);
        jedis.zadd("zset",map);
        System.out.println(jedis.zadd("zset",2600,"xm"));
        System.out.println(jedis.zrange("zset",0,-1));
        System.out.println(jedis.zrangeWithScores("zset",0,-1));
        System.out.println("移除操作:"+jedis.zrem("zset","xm"));
        System.out.println(jedis.zrangeWithScores("zset",0,-1));
        System.out.println("获取个数:"+jedis.zcard("zset"));
        System.out.println("获取指定区间的值:"+jedis.zrange("zset",0,2));
        System.out.println(jedis.keys("*"));
    }
}
```

# SpringBoot整合redis

## 原理剖析

> 说明： 在 SpringBoot2.x 之后，原来使用的jedis 被替换为了 lettuce
>
> jedis:采用直连,多个线程操作的情况下是不安全的,如果想要避免不安全的话,使用jedis pool连接池
>
> lettuce:采用netty,示例可以在多个线程中进行共享,不存在线程安全问题。

1.序列化规则

![image-20200519112230748](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200519112230748.png)

![image-20200519112313627](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200519112313627.png)

* 导入pom文件

```xml
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-redis</artifactId>
 </dependency>
```

* 测试类

```java
@SpringBootTest
class JedisApplicationTests {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads() {
        //获取redis的连接对象
        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
        connection.flushDb();
        //操作字符串
        redisTemplate.opsForValue().set("name","zhangsan");
        //操作list
        redisTemplate.opsForList().leftPush("list01","zhangsan");
        //操作事务
        redisTemplate.multi();//开启事务
        redisTemplate.discard();//停止事务
    }
}
```

## 自定义RedisTemplate

> 使用redis需要对pojo实现序列话,否则会报错

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String name;
    private int age;
}

@SpringBootTest
class JedisApplicationTests {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads() {
        //获取redis的连接对象
        User user = new User("张三",14);
        //操作字符串
        redisTemplate.opsForValue().set("user",user);
        System.out.println(redisTemplate.opsForValue().get("user"));
    }
}
```

![image-20200519224209152](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200519224209152.png)

## 自定义之前

```java
@SpringBootTest
class JedisApplicationTests {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads() {
        //获取redis的连接对象
        User user = new User("张三",14);
        //操作字符串
        redisTemplate.opsForValue().set("user",user);
        System.out.println(redisTemplate.opsForValue().get("user"));
    }
}
```

![image-20200519231029151](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200519231029151.png)

## 自定义之后

```java
@Configuration
public class RedisConfig {

    @Bean
    @SuppressWarnings("all")//压制所有的错误
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<String,Object>();
        template.setConnectionFactory(redisConnectionFactory);
        //json序列化配置
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        //String的序列话
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        //key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        //hash的key可以采用string的方式
        template.setHashKeySerializer(stringRedisSerializer);
        //value的序列化采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        //hash的value的序列话采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        return template;
    }
}
```

```java
@SpringBootTest
class JedisApplicationTests {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads() {
        //获取redis的连接对象
        User user = new User("张三",14);
        //操作字符串
        redisTemplate.opsForValue().set("user",user);
        System.out.println(redisTemplate.opsForValue().get("user"));
    }
}
```

![image-20200519230607253](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200519230607253.png)

## 通用工具类分装

```java
// 在我们真实的分发中，或者你们在公司，一般都可以看到一个公司自己封装RedisUtil
@Component
public final class RedisUtil {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // =============================common============================
    /**
     * 指定缓存失效时间
     * @param key  键
     * @param time 时间(秒)
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }


    /**
     * 判断key是否存在
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除缓存
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }


    // ============================String=============================

    /**
     * 普通缓存获取
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }
    
    /**
     * 普通缓存放入
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */

    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 普通缓存放入并设置时间
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */

    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 递增
     * @param key   键
     * @param delta 要增加几(大于0)
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }


    /**
     * 递减
     * @param key   键
     * @param delta 要减少几(小于0)
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }


    // ================================Map=================================

    /**
     * HashGet
     * @param key  键 不能为null
     * @param item 项 不能为null
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }
    
    /**
     * 获取hashKey对应的所有键值
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }
    
    /**
     * HashSet
     * @param key 键
     * @param map 对应多个键值
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * HashSet 并设置时间
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }


    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }


    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }


    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }


    // ============================set=============================

    /**
     * 根据key获取Set中的所有值
     * @param key 键
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 获取set缓存的长度
     *
     * @param key 键
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */

    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    // ===============================list=================================
    
    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 获取list缓存的长度
     *
     * @param key 键
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将list放入缓存
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */

    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */

    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }

    }

}
```

# Redis.cofig配置

```bash
# 配置文件 unit单位 对大小写不敏感！
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
```

> 包含

```bash
################################## INCLUDES ###################################

# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include .\path\to\local.conf
# include c:\path\to\other.conf
```

> 网络 NETWORK

```bash
bind 127.0.0.1 #绑定的ip
protected-mode yes #是否开启保护模式
port 6379 #端口
```

> 通用 GENERAL

```bash
daemonize no #是否以守护线程开启
pidfile /var/run/redis_6379.pid # 如果以后台的方式运行，我们就需要指定一个 pid 文件！
#日志
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice
logfile "" #日子的文件位置,默认当前下
databases 16 # 数据库的数量，默认是 16 个数据库
always-show-logo yes # 是否总是显示LOGO
```

> SNAPSHOTTING 快照 
>
> 持久化， 在规定的时间内，执行了多少次操作，则会持久化到文件 .rdb. aof

```bash
save 900 1 #900s内,如果至少有1个key进行了改变,我们即进行持久化
save 300 10 #900s内,如果至少有10个key进行了改变,我们即进行持久化
save 60 10000  #900s内,如果至少有10000key进行了改变,我们即进行持久化
stop-writes-on-bgsave-error yes #如果持久化出错,我们还继续执行
rdbcompression yes #是否压缩red文件
rdbchecksum yes #保存rdb的时候,进行错误的校验
dir ./ #rdb保存的文件路径
```

> REPLICATION 复制

> SECURITY 安全
>
> 可以在这里设置redis的密码，默认是没有密码！

```bash
# requirepass foobared 设置密码
127.0.0.1:6379> ping PONG 
127.0.0.1:6379> config get requirepass # 获取redis的密码 
1) "requirepass" 
2) "" 
127.0.0.1:6379> config set requirepass "123456" # 设置redis的密码 
OK
127.0.0.1:6379> config get requirepass # 发现所有的命令都没有权限了 (error)
NOAUTH Authentication required. 
127.0.0.1:6379> ping (error) NOAUTH Authentication required. 
127.0.0.1:6379> auth 123456 # 使用密码进行登录！
OK
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "123456"
```

> LIMITS 限制Client

```bash
 maxclients 10000  # 设置能连接上redis的最大连接数
 maxmemory <bytes> # 设置最大的内存
 maxmemory-policy noeviction # 内存到达上限之后的处理策略
     1、volatile-lru：只对设置了过期时间的key进行LRU（默认值） 
     2、allkeys-lru ： 删除lru算法的key 
     3、volatile-random：随机删除即将过期key
     4、allkeys-random：随机删除 
     5、volatile-ttl ： 删除即将过期的 
     6、noeviction ： 永不过期，返回错误
 
```

> APPEND ONLY MODE AOF的配置

```bash
appendonly no  # 默认是不开启aof模式的，默认是使用rdb方式持久化的
appendfilename "appendonly.aof" # 持久化的名字
appendfsync always # 每次修改都会 sync。消耗性能 
appendfsync everysec # 每秒执行一次 sync，可能会丢失这1s的数据！
appendfsync no # 不执行 sync，这个时候操作系统自己同步数据，速度最快！
```

# 持久化

## RDB

> 因为redis是内存数据库,如果不将内存数据库保存到磁盘中一旦服务器进程退出,服务器中的数据也就丢失了，所以需要提供持久化

rdb保存的文件是dump.rdb文件格式,文件名是由redis.conf得来的

![image-20200520105335246](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200520105335246.png)

![image-20200520105423241](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200520105423241.png)

> 触发机制：会生成dump.rdb文件

1. save满足的规则下会自动触发

2. 执行flushall命令

3. 退出redis

   会生成y一个dump.rdb文件

> 存在的位置

```bash
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/bin"
```

> 优点:
>
> ​	1.适合大规模的数据恢复！
>
> ​	2.对数据的完整性要不高！

> 缺点:
>
> ​    1.需要一定的时间间隔进程操作！如果redis意外宕机了，这个最后一次修改数据就没有的了！
>
> ​	2. fork进程的时候,需要占用一定的空间

## AOP

> 将所有的命令都记录下来,history，恢复的时候把这个文件的命令在重新执行一次

![image-20200520111832657](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200520111832657.png)

* 默认是不开启的 我们只需要将 appendonly 改为yes就开启了 aof！

* 重启，redis 就可以生效了！

> 注意：当问的aop文件被手动修改或者里面由错误的时候,会导致redis启动失效(使用aop备份程序),此时
>
> 我们需要修复这个文件,使用redis-check-aof --fix  appendonly.aof

> aof是无限追加的会越来越大,如果aof文件大于64m,fork一个新的进程来对文件进行追加写入

![image-20200520114435921](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200520114435921.png)

> 优点:
>
>  1.每次修改都同步,数据完整性好
>
>  2.每秒执行一次,可能会丢失1s的数据
>
>  3.不执行同步,操作系统会自己同步,效率最高

> 缺点
>
>  1.相对于数据文件来说,aof远大于rdb,修复的数据也比rdb慢
>
>  2.Aof的运行速率也比rdb的低

# 发布订阅

> 发布订阅是一种消息通信模式:发送者,订阅者,订阅者接受消息
>
> 例子:微信,微博,公众号

* 命令

> PUBSUB subcomand [argument [argument…]] 查看订阅与发布的系统状态
>
> SUBSCRIBE channel[channel…] 订阅一个或者多个的频道信息
>
> PSUBSCRIBE pattern [pattern…]  订阅一个或者符合多模式的频道信息
>
> PUBLISH channel message 将信息发送到指定的频道
>
> UNSUBSCRIBE[channel[channel…]] 指退订给定的频道
>
> PUNSUBSCRIBE[pattern[pattern]] 退订所有给定的频道

```bash
SUBSCRIBE xuehy #订阅一个频道
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "xuehy"
3) (integer) 1
1) "message"
2) "xuehy"
3) "nihao"
127.0.0.1:6379> PUBLISH xuehy nihao #往一个频道中发布信息
(integer) 1

127.0.0.1:6379> PUBSUB CHANNELS xuehy #查看订阅状态
1) "xuehy"
```

**使用场景：**

1、实时消息系统！

2、事实聊天！（频道当做聊天室，将信息回显给所有人即可！）

3、订阅，关注系统都是可以的！

​      稍微复杂的场景我们就会使用 消息中间件 MQ （）

# Redis的集群搭建

> 主从复制：是指将一台服务器的数据,复制到其他的redis服务器。前者称为主节点(master)，
>
> 后者成为从节点(slave).复制是单向的,只能由主节点到从节点

> 优点:
>
>    1.数据冗余:主从复制实现了数据的热备份
>
>    2.故障恢复:当主节点出现问题时,可以由从节点提供服务
>
>    3.负载均衡:实现读写分离
>
>    4.高可用

```bash
# 查看环境配置
127.0.0.1:6379> info replication
# Replication 
role:master  #默认就是主节点
connected_slaves:0 # 当前从节点的个数
master_replid:58339fcacffbfde3995d74758f2c5804f0c8d7d4
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
master_repl_meaningful_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

## 配置集群

> 修改redis.conf文件
>
>   1.pid端口
>
>   2.log文件名
>
>   3.dump文件名

配置主从复制

```bash
# 默认情况下，每台Redis服务器都是主节点； 我们一般情况下只用配置从机就好了！
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379
OK
127.0.0.1:6380> info replication
# Replication
role:slave #当前角色是从机
master_host:127.0.0.1 #主机的地址
master_port:6379  #主机的端口
master_link_status:up
master_last_io_seconds_ago:2
master_sync_in_progress:0
slave_repl_offset:28
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:4706b0faacbbb456f80e95e9a9a56496224ed0ae
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:28
master_repl_meaningful_offset:0
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:28
```

> 注意:真实的从主配置应该在配置文件中配置，这样的话是永久的，我们这里使用的是命令，暂时的!

![image-20200520155543270](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200520155543270.png)

> 复制原理：
>
> Slave 启动成功连接到 master 后会发送一个sync同步命令
>
> Master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行
>
> 完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。
>
> ==全量复制==：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
>
> ==增量复制==：Master 继续将新的所有收集到的修改命令依次传给slave，完成同步
>
> 但是只要是重新连接master，一次完全同步（全量复制）将被自动执行！ 我们的数据一定可以在从机中
>
> 看到！

当主机(Master)挂掉,我们可以手动让自己变成主机

> SLAVEOF no one

# 哨兵模式

> 描述:主从切换的技术是，当主服务宕机之后,需要==手动==从服务器中切换一台服务器作为主服务器，这就需要人工干预,费时费力。哨兵模式就是为了解决这种手动的方式

> 测试：

```bash
在配置文件中配置sentinel.conf
# sentinel monitor 被监控的名称 host port 1
sentinel monitor myredis 127.0.0.1 6379 1
# 后面的数字1表示当主机挂了,slave投票让谁接替成为主机，票数最多这成为新的主机
```

![image-20200520170634529](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200520170634529.png)

> 注意:当主机6379在连接上来,它自动变为6380的从机器

> 哨兵模式的优点:
>
>  1.哨兵集群,基于主从复制，实现高可用、热备份、故障恢复、负载均衡
>
>  2.哨兵模式是主动模式升级的、系统的可用性会更好
>
> 缺点:
>
>   1.Redis不好在线扩容,集群容量达到一定上限,在线扩容就很麻烦
>
>   2.实现哨兵模式的配置复杂

* 哨兵文件的配置

```bash
# Example sentinel.conf 
# 哨兵sentinel实例运行的端口 默认26379
port 26379 
# 哨兵sentinel的工作目录 
dir /tmp 
# 哨兵sentinel监控的redis主节点的 ip port
# master-name 可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
# quorum 配置多少个sentinel哨兵统一认为master主节点失联 那么这时客观上认为主节点失联了
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2
# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供
# 密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000
# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，
# 这个数字越小，完成failover所需的时间就越长，
# 但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。
# 可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1
# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那 里同步数据时。 
#3.当想要取消一个正在进行的failover所需要的时间。 
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时， slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
# 默认三分钟 
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000
# SCRIPTS EXECUTION
#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知 相关人员。
#对于脚本的运行结果有以下规则： 
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10 
#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。 
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等）， 将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信 息。调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述。如果sentinel.conf配 置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无 法正常启动成功。 
#通知脚本 
# shell编程
# sentinel notification-script <master-name> <script-path>
sentinel notification-script mymaster /var/redis/notify.sh
# 客户端重新配置主节点参数脚本 # 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已 经发生改变的信息。 # 以下参数将会在调用脚本时传给脚本: 
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port> 
# 目前<state>总是“failover”, 
# <role>是“leader”或者“observer”中的一个。
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通 信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。 
# sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh # 一般都是由运维来配置！
```

# 缓存穿透和雪崩

- 缓存穿透:缓存穿透的意思就是当用户去缓存中查某条数据没有,于是去持久层查询也没有,于是本次查询失败。当很多用户的时候,给持久层造成了很大的压力,这就叫缓存穿透。

> 解决方法：
>
> 1.布隆过滤器
>
> 2.缓存空对象
>
>    - 当存储层没有命中的时候,即将返回的空对象缓存起来，同时设置过期时间
>
>    - 存在的问题
>
>      1、如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键；
>
>      2、即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。

- 缓存击穿：是指一个key非常热点,在不停的扛着大并发，大并发在一个时间点对这个点进行访问,当key在失效的瞬间,持续的大并发就穿破缓存直接请求数据库，就像在屏障上开了一个洞。

> 解决办法：
>
> 1.设置热点数据不过期
>
> 2.加分布式锁

- 缓存雪崩:是指在某一段时间内,缓存集体失效。redis宕机

> 解决办法:
>
> 1.redis高可用，搭建多集群
>
> 2.限流降级这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
>
> 3.数据预热
>
> ​    在服务正式部署前,先把可能的数据访问一遍,对大量访问的数据手动设置不同的过期时间。让缓存失效 均匀点。
>
>    



