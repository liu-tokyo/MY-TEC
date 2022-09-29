# Redis 远程字典服务 - 百度百科

Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用ANSI [C语言](https://baike.baidu.com/item/C语言)编写、支持网络、可基于内存亦可持久化的日志型、Key-Value[数据库](https://baike.baidu.com/item/数据库/103728)，并提供多种语言的API。

- 中文名

  远程字典服务

- 外文名

  Remote Dictionary Server

- 简  称

  Redis

- 分  类

  数据库

- 相  关

  NoSql 数据存储

- 开发语言

  ANSI[C语言](https://baike.baidu.com/item/C语言)

- 特  点

  速度快

## 定义

redis是一个key-value[存储系统](https://baike.baidu.com/item/存储系统)。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list([链表](https://baike.baidu.com/item/链表))、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些[数据类型](https://baike.baidu.com/item/数据类型)都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了[memcached](https://baike.baidu.com/item/memcached)这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。 [1] 

Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了[发布/订阅](https://baike.baidu.com/item/发布%2F订阅)机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。

redis的官网地址，非常好记，是redis.io。（域名后缀io属于国家域名，是british Indian Ocean territory，即英属印度洋领地），Vmware在资助着redis项目的开发和维护。

从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由[Pivotal](https://baike.baidu.com/item/Pivotal)赞助。

## 作者

redis [2] 的作者，叫Salvatore Sanfilippo，来自意大利的西西里岛，居住在卡塔尼亚。目前供职于Pivotal公司。他使用的网名是antirez。

## 性能

下面是官方的bench-mark数据： [1] 

测试完成了50个并发执行100000个请求。

设置和获取的值是一个256字节字符串。

Linux box是运行Linux 2.6,这是X3320 Xeon 2.5 ghz。

文本执行使用loopback接口(127.0.0.1)。

结果:读的速度是110000次/s,写的速度是81000次/s 。

## 支持语言

许多语言都包含Redis支持，包括： [1] 

| 授权协议：BSD                     | 开源组织：无         |
| --------------------------------- | -------------------- |
| 开发语言：C/C++ 查看源码 »        | 地区：不详           |
| 操作系统：Linux                   | 投 递 者：红薯       |
| 软件类型：开源软件                | 适用人群：未知       |
| 所属分类：服务器软件、 缓存服务器 | 收录时间：2009-04-14 |

## 常用命令

就DB来说，Redis成绩已经很惊人了，且不说[memcachedb](https://baike.baidu.com/item/memcachedb)和[Tokyo Cabinet](https://baike.baidu.com/item/Tokyo Cabinet)之流，就说原版的memcached，速度似乎也只能达到这个级别。Redis根本是使用内存[存储](https://baike.baidu.com/item/存储)，持久化的关键是这三条指令：SAVE BGSAVE LASTSAVE …

当接收到SAVE指令的时候，Redis就会dump数据到一个文件里面。

值得一说的是它的独家功能：存储列表和集合，这是它与mc之流相比更有竞争力的地方。

不介绍mc里面已经有的内容，只列出特殊的：

- TYPE key — 用来获取某key的类型

- KEYS pattern — 匹配所有符合模式的key，比如KEYS * 就列出所有的key了，当然，复杂度O(n)

- RANDOMKEY - 返回随机的一个key

- RENAME oldkey[newkey](https://baike.baidu.com/item/newkey)— key也可以改名

列表操作，精华

- RPUSH key string — 将某个值加入到一个key列表末尾

- LPUSH key string — 将某个值加入到一个key列表头部

- LLEN key — 列表长度

- LRANGE key start end — 返回列表中某个范围的值，相当于mysql里面的[分页](https://baike.baidu.com/item/分页)查询那样

- LTRIM key start end — 只保留列表中某个范围的值

- LINDEX key index — 获取列表中特定索引号的值，要注意是O(n)复杂度

- LSET key index value — 设置列表中某个位置的值

- LPOP key

- RPOP key — 和上面的LPOP一样，就是类似栈或队列的那种取头取尾指令，可以当成[消息队列](https://baike.baidu.com/item/消息队列)来使用了

集合操作

- SADD key member — 增加元素

- SREM key member — 删除元素

- SCARD key — 返回集合大小

- SISMEMBER key member — 判断某个值是否在集合中

- SINTER key1 key2 ... keyN — 获取多个集合的交集元素

- SMEMBERS key — 列出集合的所有元素

还有Multiple DB的命令，可以更换db，数据可以隔离开，默认是存放在DB 0。

## 数据模型

Redis的外围由一个键、值映射的[字典](https://baike.baidu.com/item/字典)构成。与其他[非关系型数据库](https://baike.baidu.com/item/非关系型数据库)主要不同在于：Redis中值的类型 [1] 不仅限于[字符串](https://baike.baidu.com/item/字符串)，还支持如下抽象数据类型：

- [字符串](https://baike.baidu.com/item/字符串)[列表](https://baike.baidu.com/item/列表)
- 无序不重复的[字符串](https://baike.baidu.com/item/字符串)[集合](https://baike.baidu.com/item/集合)
- 有序不重复的[字符串](https://baike.baidu.com/item/字符串)[集合](https://baike.baidu.com/item/集合)
- 键、值都为[字符串](https://baike.baidu.com/item/字符串)的[哈希表](https://baike.baidu.com/item/哈希表) [1] 

值的类型决定了值本身支持的操作。Redis支持不同无序、有序的[列表](https://baike.baidu.com/item/列表)，无序、有序的[集合](https://baike.baidu.com/item/集合)间的交集、并集等高级服务器端原子操作。

## 数据结构

redis提供五种数据类型：[string](https://baike.baidu.com/item/string)，hash，list，set及zset(sorted set)。

- **string（字符串）**

  string是最简单的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value，其上支持的操作与Memcached的操作类似。但它的功能更丰富。

  redis采用结构sdshdr和sds封装了字符串，字符串相关的操作实现在[源文件](https://baike.baidu.com/item/源文件)sds.h/sds.c中。

  [数据结构](https://baike.baidu.com/item/数据结构)定义如下：

  ```c++
  typedefchar*sds;
  structsdshdr{
  longlen;
  longfree;
  charbuf[];
  };
  ```

- **list(双向链表)**

  list是一个链表结构，主要功能是push、pop、获取一个范围的所有值等等。操作中key理解为链表的名字。

  对list的定义和实现在源文件adlist.h/adlist.c，相关的[数据结构](https://baike.baidu.com/item/数据结构)定义如下：

  ```c++
  //list迭代器
  typedefstructlistIter{
  listNode*next;
  intdirection;
  }listIter;
  //list数据结构
  typedefstructlist{
  listNode*head;
  listNode*tail;
  void*(*dup)(void*ptr);
  void(*free)(void*ptr);
  int(*match)(void*ptr,void*key);
  unsignedintlen;
  listIteriter;
  }list;
  ```

- **dict(hash表)**

  set是集合，和我们数学中的集合概念相似，对集合的操作有添加删除元素，有对多个集合求交并差等操作。操作中key理解为集合的名字。

  在源文件dict.h/dict.c中实现了hashtable的操作，[数据结构](https://baike.baidu.com/item/数据结构)的定义如下：

  ```c++
  //dict中的元素项
  typedefstructdictEntry{
  void*key;
  void*val;
  structdictEntry*next;
  }dictEntry;
  //dict相关配置函数
  typedefstructdictType{
  unsignedint(*hashFunction)(constvoid*key);
  void*(*keyDup)(void*privdata,constvoid*key);
  void*(*valDup)(void*privdata,constvoid*obj);
  int(*keyCompare)(void*privdata,constvoid*key1,constvoid*key2);
  void(*keyDestructor)(void*privdata,void*key);
  void(*valDestructor)(void*privdata,void*obj);
  }dictType;
  //dict定义
  typedefstructdict{
  dictEntry**table;
  dictType*type;
  unsignedlongsize;
  unsignedlongsizemask;
  unsignedlongused;
  void*privdata;
  }dict;
  //dict迭代器
  typedefstructdictIterator{
  dict*ht;
  intindex;
  dictEntry*entry,*nextEntry;
  }dictIterator;
  ```

  dict中table为dictEntry[指针](https://baike.baidu.com/item/指针)的[数组](https://baike.baidu.com/item/数组)，数组中每个成员为hash值相同元素的[单向链表](https://baike.baidu.com/item/单向链表)。set是在dict的基础上实现的，指定了key的比较函数为dictEncObjKeyCompare，若key相等则不再插入。

- **zset(排序set)**

  zset是set的一个升级版本，他在set的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，zset会自动重新按新的值调整顺序。可以理解了有两列的mysql表，一列存value，一列存顺序。操作中key理解为zset的名字。

  ```c++
  typedefstructzskiplistNode{
  structzskiplistNode**forward;
  structzskiplistNode*backward;
  doublescore;
  robj*obj;
  }zskiplistNode;
  typedefstructzskiplist{
  structzskiplistNode*header,*tail;
  unsignedlonglength;
  intlevel;
  }zskiplist;
  typedefstructzset{
  dict*dict;
  zskiplist*zsl;
  }zset;
  ```

  zset利用dict维护key -> value的映射关系，用zsl(zskiplist)保存value的有序关系。zsl实际是叉数

  不稳定的多叉树，每条链上的元素从根节点到[叶子节点](https://baike.baidu.com/item/叶子节点)保持升序排序。

## 存储

redis使用了两种[文件格式](https://baike.baidu.com/item/文件格式)：全量数据和增量请求。

全量数据格式是把内存中的数据写入磁盘，便于下次读取文件进行加载；

增量请求文件则是把内存中的数据序列化为操作请求，用于读取文件进行replay得到数据，序列化的操作包括SET、RPUSH、SADD、ZADD。

redis的存储分为内存存储、[磁盘存储](https://baike.baidu.com/item/磁盘存储)和log文件三部分，配置文件中有三个参数对其进行配置。

save seconds updates，save配置，指出在多长时间内，有多少次更新操作，就将[数据同步](https://baike.baidu.com/item/数据同步)到数据文件。这个可以多个条件配合，比如默认配置文件中的设置，就设置了三个条件。

appendonly yes/no ，appendonly配置，指出是否在每次更新操作后进行日志记录，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为redis本身同步数据文件是按上面的save条件来同步的，所以有的数据会在一段时间内只存在于内存中。

appendfsync no/always/everysec ，appendfsync配置，no表示等[操作系统](https://baike.baidu.com/item/操作系统)进行[数据缓存](https://baike.baidu.com/item/数据缓存)同步到磁盘，always表示每次更新操作后手动调用fsync()将数据写到磁盘，everysec表示每秒同步一次。

## 安装

- **获取源码、解压、进入源码目录**

  使用wget工具等下载：

  wget （百度不让用链接）

  tar xzf redis-1.2.6.tar.gz

  cd redis-1.2.6。

- **编译生成可执行文件**

  由于makefile文件已经写好，我们只需要直接在源码目录执行make命令进行编译即可：

  make

  make-test

  sudo make install

  make命令执行完成后，会在当前目录下生成本个[可执行文件](https://baike.baidu.com/item/可执行文件)，分别是redis-server、redis-cli、redis-benchmark、redis-stat，它们的作用如下：

  redis-server：Redis服务器的daemon启动程序

  redis-cli：Redis命令行操作工具。当然，你也可以用telnet根据其纯文本协议来操作

  redis-benchmark：Redis[性能测试](https://baike.baidu.com/item/性能测试)工具，测试Redis在你的系统及你的配置下的读写性能

  redis-stat：Redis状态检测工具，可以检测Redis当前状态参数及延迟状况。

- **建立Redis目录（非必须）**

  这个过程不是必须的，只是为了将Redis相关的[资源统一管理](https://baike.baidu.com/item/资源统一管理)而进行的操作。

  执行以下命令建立相关目录并拷贝相关文件至目录中：

  ```
  sudo -s
  mkdir -p /usr/local/redis/bin
  mkdir -p /usr/local/redis/etc
  mkdir -p /usr/local/redis/var
  cp redis-server redis-cli redis-benchmark redis-stat /usr/local/redis/bin/
  cp redis.conf /usr/local/redis/etc/
  ```

- **配置参数**

  在我们成功安装Redis后，我们直接执行redis-server即可运行Redis，此时它是按照默认配置来运行的（默认配置甚至不是[后台](https://baike.baidu.com/item/后台)运行）。我们希望Redis按我们的要求运行，则我们需要修改配置文件，Redis的配置文件就是我们上面第二个cp操作的redis.conf文件，它被我们拷贝到了/usr/local/redis/etc/目录下。修改它就可以配置我们的server了。如何修改？下面是redis.conf的主要配置参数的意义：

  daemonize：是否以[后台](https://baike.baidu.com/item/后台)daemon方式运行

  pidfile：pid文件位置

  port：监听的端口号

  timeout：请求超时时间

  loglevel：log信息级别

  logfile：log文件位置

  databases：开启数据库的数量

  save * *：保存[快照](https://baike.baidu.com/item/快照)的频率，第一个*表示多长时间，第二个*表示执行多少次写操作。在一定时间内执行一定数量的写操作时，自动保存[快照](https://baike.baidu.com/item/快照)。可设置多个条件。

  rdbcompression：是否使用压缩

  dbfilename：数据[快照](https://baike.baidu.com/item/快照)文件名（只是文件名，不包括目录）

  dir：数据[快照](https://baike.baidu.com/item/快照)的保存目录（这个是目录）

  appendonly：是否开启appendonlylog，开启的话每次写操作会记一条log，这会提高数据抗风险能力，但影响效率。

  appendfsync：appendonlylog如何同步到磁盘（三个选项，分别是每次写都强制调用fsync、每秒启用一次fsync、不调用fsync等待系统自己同步）

  下面是一个略做修改后的配置文件内容：

  ```C
  daemonizeyes
  pidfile/usr/local/redis/var/redis.pid
  port6379
  timeout300
  logleveldebug
  logfile/usr/local/redis/var/redis.log
  databases16
  save9001
  save30010
  save6010000
  rdbcompressionyes
  dbfilenamedump.rdb
  dir/usr/local/redis/var/
  appendonlyno
  appendfsyncalways
  glueoutputbufyes
  shareobjectsno
  shareobjectspoolsize1024
  ```

  将上面内容写为redis.conf并保存到/usr/local/redis/etc/目录下

  然后在命令行执行：

  ```
  /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
  ```

  即可在[后台](https://baike.baidu.com/item/后台)启动redis服务，这时你通过

  ```
  telnet[127.0.0.1](https://baike.baidu.com/item/127.0.0.1)6379
  ```

  即可连接到你的redis服务

- **Redis常用内存优化手段与参数**

  通过我们上面的一些实现上的分析可以看出redis实际上的内存管理成本非常高，即占用了过多的内存，作者对这点也非常清楚，所以提供了一系列的参数和手段来控制和节省内存，我们分别来讨论下。

  首先最重要的一点是不要开启Redis的VM选项，即虚拟内存功能，这个本来是作为Redis存储超出物理内存数据的一种数据在内存与磁盘换入换出的一个持久化策略，但是其内存管理成本也非常的高，并且我们后续会分析此种持久化策略并不成熟，所以要关闭VM功能，请检查你的redis.conf文件中 vm-enabled 为 no。

  其次最好设置下redis.conf中的maxmemory选项，该选项是告诉Redis当使用了多少物理内存后就开始拒绝后续的写入请求，该参数能很好的保护好你的Redis不会因为使用了过多的物理内存而导致swap,最终严重影响性能甚至崩溃。

  另外Redis为不同数据类型分别提供了一组参数来控制内存使用，我们在前面详细分析过Redis Hash是value内部为一个HashMap，如果该Map的成员数比较少，则会采用类似一维线性的紧凑格式来存储该Map, 即省去了大量指针的内存开销，这个参数控制对应在redis.conf配置文件中下面2项：

  1. hash-max-zipmap-entries 64
  2. hash-max-zipmap-value 512
  3. hash-max-zipmap-entries

  含义是当value这个Map内部不超过多少个成员时会采用线性紧凑格式存储，默认是64,即value内部有64个以下的成员就是使用线性紧凑存储，超过该值自动转成真正的HashMap。

  hash-max-zipmap-value 含义是当 value这个Map内部的每个成员值长度不超过多少字节就会采用线性紧凑存储来节省空间。

  以上2个条件任意一个条件超过设置值都会转换成真正的HashMap，也就不会再节省内存了，那么这个值是不是设置的越大越好呢，答案当然是否定的，HashMap的优势就是查找和操作的时间复杂度都是O(1)的，而放弃Hash采用一维存储则是O(n)的时间复杂度，如果

  成员数量很少，则影响不大，否则会严重影响性能，所以要权衡好这个值的设置，总体上还是最根本的时间成本和空间成本上的权衡。

- **同样类似的参数**

  - list-max-ziplist-entries 512
  
    说明：list数据类型多少节点以下会采用去指针的紧凑存储格式。
  
  - list-max-ziplist-value 64
  
    说明：list数据类型节点值大小小于多少字节会采用紧凑存储格式。
  
  - set-max-intset-entries 512
  
    说明：set数据类型内部数据如果全部是数值型，且包含多少节点以下会采用紧凑格式存储。

  最后想说的是Redis内部实现没有对内存分配方面做过多的优化，在一定程度上会存在内存碎片，不过大多数情况下这个不会成为Redis的性能瓶颈，不过如果在Redis内部存储的大部分数据是数值型的话，Redis内部采用了一个shared integer的方式来省去分配内存的开销，即在系统启动时先分配一个从1~n 那么多个数值对象放在一个池子中，如果存储的数据恰好是这个数值范围内的数据，则直接从池子里取出该对象，并且通过引用计数的方式来共享，这样在系统存储了大量数值下，也能一定程度上节省内存并且提高性能，这个参数值n的设置需要修改源代码中的一行宏定义REDIS_SHARED_INTEGERS，该值默认是10000，可以根据自己的需要进行修改，修改后重新编译就可以了。

  另外redis 的6种过期策略redis 中的默认的过期策略是volatile-lru 。设置方式

  ```
  config set maxmemory-policy volatile-lru
  ```

  maxmemory-policy 六种方式

  - volatile-lru：只对设置了过期时间的key进行LRU（默认值）

  - allkeys-lru ： 是从所有key里 删除 不经常使用的key

  - volatile-random：随机删除即将过期key

  - allkeys-random：随机删除

  - volatile-ttl ： 删除即将过期的

  - noeviction ： 永不过期，返回错误

  maxmemory-samples 3 是说每次进行淘汰的时候 会随机抽取3个key 从里面淘汰最不经常使用的（默认选项）

## 版本发布

2012年08月02日，Redis 2.4.16 小更新版本 NoSQL。 [3] 

2012年08月31日 ，Redis 2.4.17 小更新版本 NoSQL。 [4] 

2012年11月7日 Redis 2.6.3 发布，高性能K/V服务器

2013年4月30日Redis 2.6.13 发布，高性能K/V服务器 [5] 

2013年11月25日，Redis 2.8.1发布。 [6] 

2015年2月，Redis3.0.0发布. [7] 

2016年12月2日，Redis 4.0.0-RC1发布。 [8] 

2018年10月17日，Redis 5.0.0发布。

2019年9月25日，Redis 5.0.6发布。

2019年11月19日，Redis 5.0.7发布。

2020年3月12日，Redis 5.0.8发布。

2020年4月17日，Redis 5.0.9发布。

Redis 7.0.1 是最新的稳定版本。

## 参照URL

- https://baike.baidu.com/item/Redis/6549233?fr=aladdin
