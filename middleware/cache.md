## 缓存
### 缓存的淘汰算法?
FIFO（First In First Out）： 先进先出算法，即先放入缓存的先被移除。
LRU（Least Recently Used）： 最近最少使用算法，使用时间距离现在最久的那个被移除。
LFU（Least Frequently Used）： 最不常用算法，一定时间段内使用次数（频率）最少的那个被移除。

### 缓存更新的模式，以及会出现的问题和应对思路？
1. 读: 命中缓存,未命中则读数据库,再写入缓存;(同时缓存记失效时间,防止没有更新操作时导致一直读到脏数据;) 
2. 写: 写数据库,再删除/更新缓存；（很多业务情况下是直接删缓存）
3. 读的时候即使数据不存在，也要写一个短时间的缓存防止穿库。问题在于如果先来一波大量的null key，在nullkey过期时又来一大波用户，可能造成雪崩效应;解决办法是布隆过滤器;
4. 第二步的问题: 如果缓存刚好失效,A读库拿到旧值, B更新同时删除缓存,A再写入缓存,这样就一直是脏数据库了; 但是这个问题出现概率很小,因为读库写缓存速度很快,不可能写做完了读还没做完;而且缓存设置了失效时间,过几分钟又回拉新的数据出来;
5. 数据库和缓存双写一致性问题：先更新数据库再更新缓存，若更新完数据库了，还没有更新缓存，此时有请求过来了，访问到了缓存中的数据，怎么办？ 延时双删策略：先删缓存、再更新数据库、等待很短时间，再删除缓存。但延时双删并不是一个很好的策略，在更新完数据库等待的这段时间内，仍然会读到旧数据。能保持百分百一致性的方法就是加缓存了。

### 我们现在要做的系统是百万QPS级别，即使使用了Redis，还是不够快，不够可靠，有没有别的办法让系统更快更可靠？
> see https://www.imooc.com/read/63/article/1398
本地缓存
- 优先请求一级本地缓存，命中数据后直接返回;
- 如果本地缓存命中失败，请求二级缓存Redis，命中后直接返回;
- 如果Redis命中也失败了，再查mysql;

常用本地缓存框架：
- Google Guava;
- Ehcache;
- 语言自带的map;

问题: 基本每一个服务都做集群的，每次请求相同的数据不一定请求到同一台服务器的，那本地缓存命中的几率是不是不太高了? 

### 分布式锁有哪些主流实现方式?redis和 zk 锁有什么区别?
- redis的setnx, 需要注意几点:
    1. 过期时间和set需要是原子操作,使用`set <key> <value> EX <timeout> NX`同时设置;或者使用lua脚本;
    2. 解锁操作,需要判断该锁是否是该进程加的,再执行del操作;否则会出现误删其他进程锁的情况;但这又不是一个原子操作;同样建议使用脚本完成;
    3. 过期时间的长短是一个问题,程序执行时间过长,程序应该向redis延长过期时间;(但如果程序卡死就无法实现, 恢复之后锁已经超时被别人加上去了)
    4. 市面上有成熟的redisson;

或者使用zk:
- 获取zk的分布式锁,zk会创建一个znode;另外一个机器也尝试去创建那个znode，结果发现自己创建不了，因为被别人创建了，那只能等待;
- 使用ZooKeeper的顺序节点特性，假如我们在/lock/目录下创建3个节点，ZK集群会按照发起创建的顺序来创建节点，节点分别为/lock/0000000001、/lock/0000000002、/lock/0000000003，最后一位数是依次递增的，节点名由zk来完成;
- ZK中还有一种名为临时节点的节点，临时节点由某个客户端创建，当客户端与ZK集群断开连接，则该节点自动被删除。`EPHEMERAL_SEQUENTIAL`为临时顺序节点。

1. 客户端调用create()方法创建名为“/dlm-locks/lockname/lock-”的临时顺序节点。
2. 客户端调用getChildren(“lockname”)方法来获取所有已经创建的子节点。
3. 客户端获取到所有子节点path之后，如果发现自己在步骤1中创建的节点是所有节点中序号最小的，就是看自己创建的序列号是否排第一，如果是第一，那么就认为这个客户端获得了锁，在它前面没有别的客户端拿到锁。
4. 如果创建的节点不是所有节点中需要最小的，那么则监视比自己创建节点的序列号小的最大的节点，进入等待。直到下次监视的子节点变更的时候，再进行子节点的获取，判断是否获取锁。

### 介绍下 Fescar/了解 CAP 吗?redis里的 CAP 是怎样的?
- CAP是分布式系统能够满足的三种特性: Consistency一致性、Availability可用性、Partition tolerance分区容错性;
- 一致性表示所有节点在同一时间的数据完全一致,
- 可用性指服务一直可用，而且是正常响应时间;
- 容错性指在遇到某节点或网络分区故障的时候，仍然能够对外提供满足一致性和可用性的服务。
- 只能满足其中两点,例如出现网络故障,主从仍然可以访问,满足P;但是从库数据偏移;如果从库允许操作,就不满足一致性,如果从库不允许操作,就不满足可用性;
- 根据业务不同有不同的侧重,例如对用户服务通常牺牲一致性,满足AP,保证服务正常运行,只是数据可能有延迟;对库存或者交易订单通常牺牲可用性,必须保证数据一致.

- redis单节点满足CA: 因为不涉及网络分区;
- redis哨兵模式满足CA: 哨兵保证了可用性,如果slave只是复制,则满足强一致性;
- cluster满足AP,因为cluster是通过分片来实现的,不存在数据一致的可能;

### 经典三连: 雪崩，穿透，击穿
- 雪崩就是指缓存中大批量热点数据同时过期, 大流量涌入数据库;

    解决办法：
    - 将缓存失效时间分散开，比如每个key的过期时间是随机，防止同一时间大量数据过期现象发生，这样不会出现同一时间全部请求都落在数据库层，如果缓存数据库是分布式部署，将热点数据均匀分布在不同Redis和数据库中，有效分担压力。
    - (如果准许)让Redis数据永不过期 / 或者可以加超长过期时间,每次请求重设过期时间。比如中奖名单用户，每期用户开奖后，名单不可能会变了，无需更新。

- 穿透是指数据在redis与db都不存在, 大流量请求导致db压力极大;

    解决办法:
    - 分布式布隆过滤器(bf BloomFilter)：Redis自身支持BloomFilter。
    - 遇到数据库和Redis都查询不到的值，在Redis里set一个null value，过期时间很短，目的在于同一个key再次请求时直接返回null，避免穿透。

- 击穿和穿透概念类似，一般是指一个key被穿透，这个key是热点key，同一个key会被有成千上万次请求，比如微博热点排行榜，key是小时时间戳，value是个list的榜单。每个小时产生一个key，这个key会有百万QPS，如果这个key失效了，就像保险丝熔断，百万QPS直接压垮数据库。

    解决办法: 未命中缓存,先加分布式锁,再读db设置缓存,释放锁;

### 布隆过滤器？
- 在查业务缓存前设置一个bitmap，对于缓存key，通过若干hash函数，可以将其映射到bitmap的不同位置；
- 特点是如果某个key在bf里不存在那一定不存在；bf里存在则可能存在。
- 缺点是不能删除；而且在大数据量下bitmap里为1的数据越来越多，结果误判率越来越高，最后只能重建。
- 而且经过多个不同 Hash 函数后，得到的数组下标在内存中的跨度可能会非常的大，导致cpu缓存命中率降低。
- 并且可能出现大value的情况，需要切分。

> [这里](https://www.jasondavies.com/bloomfilter/ )有bf的动画演示，可以玩一玩

拆分的形式方法多种多样，但是本质是不要将 Hash(Key) 之后的请求分散在多个节点的多个小 bitmap 上，而是应该拆分成多个小 bitmap 之后，对一个 Key 的所有哈希函数都落在这一个小 bitmap 上。

典型应用：1. 防止数据库穿库；2. 判断用户是否有阅读记录；3. 解决缓存击穿；4. web拦截器，相同请求拦截；

### 布谷鸟过滤器

原理是准备两个hash表，两个hash函数。对于每个key都计算出在这两个hash表里的位置。

- 有空位，则占位；
- 没有空位，随机踢掉一个元素；
- 被踢掉的元素去找他另一个表的位置，如果有元素，继续踢掉；
- 又被踢掉的元素又去找另一个表的位置，一层一层套娃踢，直到有元素在另一个表里面有空位不用踢了，整个流程结束(数据量极大时需要设置套娃上限，防止无限循环)

> [这里玩一玩](http://www.lkozma.net/cuckoo_hashing_visualization/)

### 退群问题, 已读未读消息设计? (布隆过滤器不能删除已删除的key,如何处理?)
- 已读未读只是一个0/1标记,直接使用bitmap来实现;每个用户附带一个bitmapId的字段,用来当bitmap的key;
- 退群问题: 
    1. 退出群聊的成员只能标记删除，不能物理删除;退出的成员又重新加入,删除掉删除标记
    2. 再加个bitmap记录退群状态;

方案对比: 
- 一般的已读未读消息设计,对每个msg都保存两个列表,一个已读列表一个未读列表;此时一个用户id要占用4byte/8byte空间(int32 int64);
- 使用bitmap记录,只占2bit(key:1bit value: 1bit);

### 热点key解决方案?
> 相同的key会请求到同一台数据分片上，导致该分片负载较高成为瓶颈问题，导致雪崩等一系列问题。

问题: 
- 高访问量的Key，一个key访问的QPS超过1000就要高度关注了，比如热门商品，热门话题等。
- 大Value，有些key访问QPS虽然不高，但是由于value很大，造成网卡负载较大，网卡流量被打满，单台机器可能出现千兆 / 秒，IO 故障。
- 热点 Key + 大 Value 同时存在，服务器直接挂。

导致:
- 数据倾斜问题：大 Value 会导致集群不同节点数据分布不均匀，造成数据倾斜问题，大量读写比例非常高的请求都会落到同一个 redis server 上，该 redis 的负载就会严重升高，容易打挂。
- QPS 倾斜：分片上的 QPS 不均。
- 大 Value 会导致 Redis 服务器缓冲区不足，造成 get 超时。
- 由于 Value 过大，导致机房网卡流量不足。
- Redis 缓存失效导致数据库层被击穿的连锁反应。

如何发现问题:
- 提前获知: 根据业务，人肉统计 or 系统统计可能会成为热点的数据，如，促销活动商品，热门话题，节假日话题，纪念日活动等;
- Redis客户端通过计数的方式统计key的请求次数; 代码入侵性较强;
- 代理层统计: 多做一层代理,统一cache查询入口,做收集上报; 这个成本比较高,需要自己开发;
- 服务端收集: 监控集群QPS, 对高QPS的节点使用`monitor`来分析热点key;(monitor会打印所有的命令) 在高并发条件下，会存在内存暴涨和 Redis 性能的隐患，所以此种方法适合在短时间内使用;
- redis4.0有基于LFU的热点key发现机制;

热点key解决办法:
- key拆分: 二级结构如hash,可以进行拆分;
- 迁移热点key: 热点key单独跑个节点;
- 限流, 对于写命令我们可以单独针对这个热点key来限流;
- 增加本地缓存: 对于数据一致性不是那么高的业务，可以将热点 key 缓存到业务机器的本地缓存中.但是当数据更新时，可能会造成业务和 Redis 数据不一致。

### 大Value怎么拆分？
根据业务总结出value的大小:
1. 大：string类型 value > 10K，set、list、hash、zset 等集合数据类型中的元素个数 > 1000。
2. 超大：string类型 value > 100K，set、list、hash、zset 等集合数据类型中的元素个数 > 10000。

几个典型的分拆方案：
1. 一个较大的kv拆分成几个kv，将操作压力平摊到多个redis实例中，降低对单个redis的IO影响;
2. 将分拆后的几个kv存储在一个hash中，每个field代表一个具体的属性，使用hget,hmget来获取部分的value，使用hset，hmset来更新部分属性;
3. hash、set、zset、list 中存储过多的元素: 类似于场景一种的第一个做法，可以将这些元素分拆;

4. 固定一个桶的数量，比如10000，每次存取的时候，先在本地计算field的hash值，对10000取模，确定该field落在哪个key上，核心思想就是将value打散，每次只 get 你需要的;
    ```
    newHashKey = hashKey + (hash(field) % 10000); 
    hset(newHashKey, field, value); 
    hget(newHashKey, field)
    ```