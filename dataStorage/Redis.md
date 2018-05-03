# JavaEE面试问题总结

##### @Author LucI_PhAN

## 数据存储和消息队列

### Redis

**1. Redis 有哪些数据类型**

Redis是一个开源的内存中的数据结构存储系统。它可以用作数据库,缓存和消息中间件。  
它支持多种类型的数据结构，如字符串String，散列Hashes，列表Lists，集合Sets，有序集合Sorted Sets或者ZSet，Bitmaps，Hyperloglogs和地理空间Geospatial索引半径查询。  
最常见的数据结构类型有String，List，Set，Hash，ZSet五种。

1.1 String  
Redis的String类型是一个由字节组成的序列。它和其他编程语言或者其他键值对存储提供的字符串操作非常相似。  
String是最常用的一种数据类型，普通的key/value存储都可以归为此类。value其实不仅是String，也可以是数字。

1.2 List  
Redis的List其实就是链表（Redis使用双端链表实现List）。  
使用List结构，可以轻松实现最新消息排行等功能。List的另一个应用就是消息队列。  
一个List结构可以有序存储多个字符串，并且是允许元素重复的。

1.3 Set  
Redis的集合和列表都可以存储多个字符串，列表可以存储多个相同的字符串，而集合通过使用散列表来保证自己存储的每个字符串都是各不相同的。   
Redis的集合使用的是无序的方式存储元素。  
应用场景：好友系统；利用唯一性，统计访问网站的所有独立IP。

1.4 Hash散列类型  
Redis的散列可以存储多个键值对之间的映射。和字符串一样，散列存储的值既可以是字符串又可以是数字值，并且用户同样可以对散列存储的数字执行自增操作或者自减操作。  
一个LIst散列类型的实例，是一个包含两个键值对的散列键。

1.5 有序集合ZSet  
有序集合和散列一样，用于存储键值对。有序集合的键被称为成员member，每一个成员都是独一无二的。而有序集合的值被称为分值score，分值必须是浮点数。  
有序集合是Redis里面唯一一个既可以根据成员访问元素，又可以根据分值以及分值的排序来访问元素的结构。  
一个有序集合类型的实例，zset-key是一个包含两个元素的有序集合键。

参考：  
[《Redis常见的5种不同的数据类型详解》](https://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247483987&idx=1&sn=5c5e4cd5bc73a7e6f84e5d6adfab0935&chksm=e9c5fbe2deb272f4b5b75bd2ac92bb27950452623ec83c0e1add7e30c773160421fab1571680&scene=21#wechat_redirect)

**2. Redis 内部结构**

Redis内部使用一个redisObject对象来表示所有的key和value。redusObject主要的信息包括数据类型type，编码方式encoding，数据指针ptr，虚拟内存vm等。  
type表示一个value对象具体是何种数据类型，encoding是不同数据类型在redis内部的编码方式。ptr指针指向对象的底层实现数据结构。

dict是一个用于维护key和value映射关系的数据结构。与很多语言中的map或dictionary类似。Redis的一个database中所有key到value的映射，就是使用一个dict来维护的。  
dict本质上是为了解决算法中的查找searching问题。一般查找问题的解法分为两大类：一个基于各种平衡树，一个基于哈希表。dict是一个基于哈希表的算法。它最显著的一个特点就在独特的rehashing。它采用了增量式重哈希incremental rehashing的方法。在需要扩展内存时避免一次性对所有key进行重哈希，而是将重哈希操作分散到对于dict的各个增删改查的操作中区。这种方法每次只对一小部分key进行rehash，而每次rehash之间不影响dict的操作。这避免了rehash期间单个请求的响应时间暴增。

Redis数据库是真正存储数据的地方。而数据库本身也是存储在内存中的。数据由dict和expires两个字典构成。其中dict保存键值对，而expires则保存键的过期时间。

参考：  
[《Redis的内部结构》](https://blog.csdn.net/tianshijianbing1989/article/details/50730572)   
[《redis内部数据结构深入浅出》](https://www.cnblogs.com/chenpingzhao/archive/2017/06/10/6965164.html)  

**3. Redis 使用场景**

Redis读写性能优异，数据类型丰富，而且Redis是单进程单线程工作的，所以增加/减少引用的操作不必保证原子性。其存储数据还可以自动过期。

3.1 Redis的高性能让其非常适合作为缓存  
缓存是Redis最常见的应用场景，因为其读写性能实在过于优异。且Redis内部是支持事务的，在使用时候能有效保证数据的一致性。

3.2 Redis支持多种数据格式，让其应用场景丰富。  
string作为最简单的k-v存储，短信验证码，配置信息等都适合用其存储。  
hash一般key作为id或者唯一标示，value存储详情。可以用于商品详情，新闻详情，个人信息等。  
list是有序的，比较适合存储有序且数据相对固定的数据。例如省市区表，字典等。因为其有序，适合根据写入时间来排序，例如最新的数据，消息队列等。  
set提供交集，并集，差集等操作。当其存储一个用户的好友时，可以非常便捷地找出几个用户的共同好友等信息，非常适合用于做推送等应用。  

3.3 Redis是单线程的，可作为分布式锁  
Redis是单线程，多路复用方式提高处理效率。Redis作为分布式锁，因为其性能的有事，不会成为瓶颈，一般会产生瓶颈的是真正的业务处理内容。  

3.4 Redis的自动过期能提高开发效率  
Redis的数据都可以设置过期时间，过期的数据清理无需使用方去关注，所以开发效率高。例如短信验证码的过期处理不需要像数据库一下还要查时间对比判断是否数据已经过期了。

参考：  
[《一起来聊聊最近很火的Redis常见应用场景解析》](http://baijiahao.baidu.com/s?id=1579614666308299862&wfr=spider&for=pc)  

**4. Redis 持久化机制**  

将redis内存服务器中的数据持久化到硬盘等介质中使得服务器再重启之后还可以重用以前得数据，也可以防止系统出现故障而导致数据无法恢复。  

Redis提供了两种不同方式的持久化方法：快照Snapshotting和之追加文件append-only-file

4.1 快照RDB  
快照就是俗称的备份，可以在定期内对数据进行备份，将redis服务器中的数据持久化到硬盘中。  
在创建快照之后，用户可以对快照进行备份。同行情况下，为了防止单台服务器出现故障而造成所有数据的丢失，还可以将快照复制到其他服务器，创建具有相同数据的数据副本。快照只适合数据不经常修改或者丢失部分数据影响不大的场景。因为快照只能恢复到最近一次生成快照的数据。如果这个时间间隔过大或者数据的修改非常频繁，会导致快照的恢复能力并不强大。

4.2 只追加文件AOF  
AOF在执行写命令的时候，将执行的写命令复制到硬盘里面。后期恢复的时候，只需要重新执行一下这个写命令就可以了。  
AOF的持久化会将被执行的写命令写到AOF文件的末尾，以此来记录数据繁盛的变化。这样，在恢复时只需要从头到尾执行一下AOF文件即可恢复数据。

参考：  
[《使用快照和AOF将Redis数据持久化到硬盘中》](https://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247483992&idx=1&sn=8f554bc490c4db1a78a30144f873e911&chksm=e9c5fbe9deb272fff47483c241e6d2a7aae99dc8f6fe9fee31f2dd214d0cf81b33d51f7a7dbe&scene=21#wechat_redirect)  

**5. Redis 集群方案与实现**

这题完全超过我的能力范围了。只给出具体链接，有人对此有深刻理解的话欢迎补充此题。

参考：  
[《Redis集群方案应该怎么做？》](https://www.zhihu.com/question/21419897)  
[《
redis集群主流架构方案分析》](https://blog.csdn.net/u011277123/article/details/55002024)  
[《这可能是最全的 Redis 集群方案介绍了》](http://www.sohu.com/a/79200151_354963)
