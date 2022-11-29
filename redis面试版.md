# 一、数据结构与数据类型

## 引言

### 键值对数据库如何实现？

Redis 的键值对中的 key 就是字符串对象，而 **value 可以是字符串对象，也可以是集合数据类型的对象**，比如 List 对象、Hash 对象、Set 对象和 Zset 对象。

**Redis 是使用了一个「哈希表」保存所有键值对**，哈希表的最大好处就是让我们可以用 O(1) 的时间复杂度来快速查找到键值对。哈希表其实就是一个数组，数组中的元素叫做哈希桶。

<img src="https://img-blog.csdnimg.cn/img_convert/f302fce6c92c0682024f47bf7579b44c.png" alt="img" style="zoom: 50%;" />

- **redisDb 结构，表示 Redis 数据库的结构**，结构体里存放了指向了 dict 结构的指针；
- dict 结构，结构体里存放了 2 个哈希表，**正常情况下都是用「哈希表1」，「哈希表2」只有在 rehash 的时候才用**
- ditctht 结构，表示哈希表的结构，结构里存放了哈希表数组，数组中的每个元素都是指向一个哈希表节点结构（dictEntry）的指针；
- dictEntry 结构，表示哈希表节点的结构，结构里存放了 **void * key 和 void * value 指针， *key 指向的是 xx 对象（这个key就被称为xx键，比如说指向String就叫字符串key，指向Hash对象就叫Hash key），value 则可以指向 String 对象，也可以指向集合类型的对象，比如 List 对象、Hash 对象、Set 对象和 Zset 对象**。

<u>简单来说，redis数据库是一个是一个个键值对对象构成的，每个键值对对象由Hash表来管理，每个对象存储在一个bucket中，redis中有两个HashTable结构，一个用来存放键值对对象，另一个在rehash时使用。每个键值对对象中包含两个指针（void* key,void* value）key指针指向一个XX类型的对象(被称为xx键)，value指向的就是redis中键值对的那几种基本数据类型。</u>

## 1、String（int 或 SDS实现）

### 介绍

String 是最基本的 key-value 结构，key 是唯一标识，value 是具体的值，value其实不仅是字符串， 也可以是数字（整数或浮点数），通常可以用来存储**反序列化后的json字符串**，value **最多可以容纳的数据长度是 `512M`。**

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/string.png" alt="img" style="zoom: 33%;" />

### 内部实现

String 类型的底层的数据结构实现主要是 **int 和 SDS（简单动态字符串）。**

<img src="https://img-blog.csdnimg.cn/img_convert/516738c4058cdf9109e40a7812ef4239.png" alt="img" style="zoom: 67%;" />

- **len，记录了字符串长度**。
- **alloc，分配给字符数组的空间长度**。这样在修改字符串的时候，可以通过 `alloc - len` 计算出剩余的空间大小，可以用来判断空间是否满足修改需求，如果不满足的话，就会自动将 SDS 的空间扩展至执行修改所需的大小，然后才执行实际的修改操作，所以使用 SDS 既不需要手动修改 SDS 的空间大小，也不会出现前面所说的缓冲区溢出的问题。
- **flags，用来表示不同类型的 SDS**。一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64，后面在说明区别之处。
- **buf[]，字符数组，用来保存实际数据**。**不仅可以保存字符串，也可以保存二进制数据。**



#### **SDS优势：**

- **SDS 不仅可以保存文本数据，还可以保存二进制数据，并且是二进制安全的**。因为 `SDS` 使用 `len` 属性的值而不是空字符来判断字符串是否结束，并且 SDS 的所有 API 都会以处理二进制的方式来处理 SDS 存放在 `buf[]` 数组里的数据。所以 SDS 不光能存放文本数据，而且能保存图片、音频、视频、压缩文件这样的二进制数据。
- **SDS 获取字符串长度的时间复杂度是 O(1)**。因为 C 语言的字符串并不记录自身长度，所以获取长度的复杂度为 O(n)；而 SDS 结构里用 `len` 属性记录了字符串长度，所以复杂度为 `O(1)`。
- **Redis 的 SDS API 是安全的，拼接字符串不会造成缓冲区溢出**。因为 SDS 在拼接字符串之前会检查 SDS 空间是否满足要求，如果空间不够会**自动扩容**，所以不会导致缓冲区溢出的问题。

字符串对象的内部编码（encoding）有 3 种 ：**int、raw 和 embstr**。

- 如果一个字符串对象保存的是**整数值**，并且这个整数值可以**用`long`类型来表示**，那么会将字符串对象中的value指针转换为long类型，并且将其编码类型设为int。
- 如果字符串对象保存的是一个字符串，并且这个字符串的长度**大于 32 字节**，那么字符串对象将使用一个简单动态字符串**（SDS）**来保存这个字符串，并将对象的编码设置为`raw`
- 如果字符串对象保存的是一个字符串，并且这个字符申的长度**小于等于 32 字节**（redis 2.+版本），那么字符串对象将使用一个简单动态字符串**（SDS）**来保存这个字符串，并将对象的编码设置为`embstr`， `embstr`编码是专门用于**保存短字符串的一种优化编码**方式

### 常用指令

```shell
# 设置 key-value 类型的值
> SET name lin
OK
# 根据 key 获得对应的 value
> GET name
"lin"
# 判断某个 key 是否存在
> EXISTS name
(integer) 1
# 返回 key 所储存的字符串值的长度
> STRLEN name
(integer) 3
# 删除某个 key 对应的值
> DEL name
(integer) 1
```

批量设置 :

```shell
# 批量设置 key-value 类型的值
> MSET key1 value1 key2 value2 
OK
# 批量获取多个 key 对应的 value
> MGET key1 key2 
1) "value1"
2) "value2"
```

计数器（字符串的内容为整数的时候可以使用）：

```shell
# 设置 key-value 类型的值
> SET number 0
OK
# 将 key 中储存的数字值增一
> INCR number
(integer) 1
# 将key中存储的数字值加 10
> INCRBY number 10
(integer) 11
# 将 key 中储存的数字值减一
> DECR number
(integer) 10
# 将key中存储的数字值键 10
> DECRBY number 10
(integer) 0
```

过期（默认为永不过期）：

```bash
# 设置 key 在 60 秒后过期（该方法是针对已经存在的key设置过期时间）
> EXPIRE name  60 
(integer) 1
# 查看数据还有多久过期
> TTL name 
(integer) 51

#设置 key-value 类型的值，并设置该key的过期时间为 60 秒
> SET key value EX 60
OK
> SETEX key  60 value
OK
```

不存在就插入：

```shell
# 不存在就插入（not exists）
>SETNX key value
(integer) 1
```

## 2、List（双向链表或压缩列表实现）

### ~说一下List类型底层数据结构

List 类型的底层数据结构是由**双向链表或压缩列表**实现的：

- 如果列表的元素个数**小于 `512` 个**（默认值，可由 `list-max-ziplist-entries` 配置），列表每个元素的值都小于 `64` 字节（默认值，可由 `list-max-ziplist-value` 配置），Redis 会使用**压缩列表**作为 List 类型的底层数据结构；
- 如果列表的元素不满足上面的条件，Redis 会使用**双向链表**作为 List 类型的底层数据结构；

但是**在 Redis 3.2 版本之后，List 数据类型底层数据结构就只由 quicklist 实现了，替代了双向链表和压缩列表**。

### ~Redis中实现链表有哪些特性？

```c
typedef struct list{
     //表头节点
     listNode *head;
     //表尾节点
     listNode *tail;
     //链表所包含的节点数量
     unsigned long len;
     //节点值复制函数
     void (*free) (void *ptr);
     //节点值释放函数
     void (*free) (void *ptr);
     //节点值对比函数
     int (*match) (void *ptr,void *key);
}list;

typedef  struct listNode{
       //前置节点
       struct listNode *prev;
       //后置节点
       struct listNode *next;
       //节点的值
       void *value;  
}listNode
```

redis在传统的双向链表上做了一层封装，特性如下：

1）双端：链表具有前置节点和后置节点的引用，获取这两个节点时间复杂度都为 O(1)。

2）无环：表**头节点的 prev 指针和表尾节点的 next 指针都指向 NULL**,对链表的访问都是以 NULL 结束。

3）带长度计数器：通过 len 属性获取链表长度的时间复杂度为 O(1)。

4）多态：链表节点使用指针来保存节点值，可以保存各种不同类型的值。

<img src="https://img-blog.csdnimg.cn/img_convert/cadf797496816eb343a19c2451437f1e.png" alt="img" style="zoom: 50%;" />

### ~双向链表的优点与缺陷？

**优点：**

- 获取长度的时间复杂度为O(1)；
- 获取某个节点的前置节点或后置节点的时间复杂度只需O(1)
- 链表节点可以保存各种不同类型的值；

**缺点：**

- 链表每个节点之间的内存都是不连续的，意味着**无法很好利用 CPU 缓存**。
- 保存一个链表节点的值都需要一个链表节点结构头的分配，**内存开销较大**。

### ~压缩列表的引入（注意List只是其中一种实现）

**Redis 对象（List 对象、Hash 对象、Zset 对象）包含的元素数量较少，或者元素值不大的情况才会使用压缩列表作为底层数据结构。**

压缩列表是 Redis 为了节约内存而开发的，它是**由连续内存块组成的顺序型数据结构**，一个压缩列表可以包含任意多的节点，每个节点可以保存一个**字节数组或整数值**。

![img](https://img-blog.csdnimg.cn/img_convert/ab0b44f557f8b5bc7acb3a53d43ebfcb.png)

- ***zlbytes***，类型为`uint32_t`，记录整个压缩列表占用对内存字节数；
- ***zltail***，类型为`uint32_t`，记录压缩列表「尾部」节点距离起始地址由多少字节，也就是列表尾的偏移量；
- ***zllen***，类型为`uint16_t`，记录压缩列表包含的节点数量；
- ***zlend***，类型为`uint8_t`标记压缩列表的结束点，固定值 0xFF（十进制255）。

<img src="https://img-blog.csdnimg.cn/img_convert/a3b1f6235cf0587115b21312fe60289c.png" alt="img" style="zoom:67%;" />

压缩列表节点包含三部分内容：

- ***prevlen***，记录了「前一个节点」的长度，目的是为了实现从后向前遍历，当前一个节点长度小于254字节时，该属性长度为1字节，当大于254字节时，该属性长度为5字节；
- ***encoding***，记录了当前节点实际数据的「类型`是字符串还是整数`和长度`占多少字节`」，类型主要有两种：**字符串和整数。**
- ***data***，记录了当前节点的实际数据，类型和长度都由 `encoding` 决定；

### ~压缩链表可能引发连锁更新问题

- 如果前一个**节点的长度小于 254 字节**，那么 prevlen 属性需要用 **1 字节的空间**来保存这个长度值；
- 如果前一个**节点的长度大于等于 254 字节**，那么 prevlen 属性需要用 **5 字节的空间**来保存这个长度值；
- <img src="https://img-blog.csdnimg.cn/img_convert/462c6a65531667f2bcf420953b0aded9.png" alt="img" style="zoom:67%;" />

**假设一个压缩列表中有多个连续的、长度在 250～253 之间的节点**，将一个**长度大于等于 254 字节**的新节点加入到压缩列表的表头节点，那么当前头节点的后一个节点的prevlen就要用5字节来保存这个长度值，所以该节点的内存空间就被重新分配，重新分配后该节点的大小也大于254了，那么就会导致之后的节点空间大小也被更新，**造成访问压缩列表性能的下降**。连锁更新在最坏情况下需要对压缩列表进行N次空间重新分配，每次空间分配最坏复杂度为O(N)，所以**连锁更新最坏复杂度为O(N^2)**。

### ~压缩列表的缺陷

空间扩展操作也就是重新分配内存，因此**连锁更新一旦发生，就会导致压缩列表占用的内存空间要多次重新分配，这就会直接影响到压缩列表的访问性能**。

所以说，**虽然压缩列表紧凑型的内存布局能节省内存开销，但是如果保存的元素数量增加了，或是元素变大了，会导致内存重新分配，最糟糕的是会有「连锁更新」的问题**。

因此，**压缩列表只会用于保存的节点数量不多的场景**，只要节点数量足够小，即使发生连锁更新，也是能接受的。

### 常用命令

```shell
# 将一个或多个值value插入到key列表的表头(最左边)，最后的值在最前面
LPUSH key value [value ...] 
# 将一个或多个值value插入到key列表的表尾(最右边)
RPUSH key value [value ...]
# 移除并返回key列表的头元素
LPOP key     
# 移除并返回key列表的尾元素
RPOP key 

# 返回列表key中指定区间内的元素，区间以偏移量start和stop指定，从0开始
LRANGE key start stop

# 从key列表表头弹出一个元素，没有就阻塞timeout秒，如果timeout=0则一直阻塞
BLPOP key [key ...] timeout
# 从key列表表尾弹出一个元素，没有就阻塞timeout秒，如果timeout=0则一直阻塞
BRPOP key [key ...] timeout
```

## 3、Hash（**压缩列表或哈希表**实现）

### 介绍

Hash 是一个键值对（key - value）集合，其中 value 的形式如： `value=[{field1，value1}，...{fieldN，valueN}]`**(field是key)**。Hash **特别适合用于存储对象。**

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221013165400499.png" alt="image-20221013165400499" style="zoom:67%;" />

### ~说一下Hash的底层数据结构

Hash 类型的底层数据结构是由**压缩列表或哈希表**实现的：

- 如果哈希类型元素个数**小于 `512` 个**（默认值，可由 `hash-max-ziplist-entries` 配置），所有值小于 `64` 字节（默认值，可由 `hash-max-ziplist-value` 配置）的话，Redis 会使用**压缩列表**作为 Hash 类型的底层数据结构；
- 如果哈希类型元素不满足上面条件，Redis 会使用**哈希表**作为 Hash 类型的 底层数据结构。

**在 Redis 7.0 中，压缩列表数据结构已经废弃了，交由 listpack 数据结构来实现了**。

**使用ZipList实现**

见Redis设计与实现 p72

**Hash数据结构分析（使用HashTable实现）：**

**底层是一个Entry数组并且记录了Entry的个数**

```c
typedef struct dictht{
     //哈希表数组
     dictEntry **table;
     //哈希表大小
     unsigned long size;
     //哈希表大小掩码，用于计算索引值
     //总是等于 size-1
     unsigned long sizemask;
     //该哈希表已有节点的数量
     unsigned long used;
 
} dictht
    
typedef struct dictEntry {
    //键值对中的键
    void *key;
  
    //键值对中的值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    //指向下一个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```



<img src="https://img-blog.csdnimg.cn/img_convert/dc495ffeaa3c3d8cb2e12129b3423118.png" alt="img" style="zoom:50%;" />

### ~Entry（hash节点）设计的妙处：

dictEntry 结构里键值对中的值是一个**「联合体 v」定义的**，因此，键值对中的值**可以是一个指向实际值的指针**，或者是一个**无符号的 64 位整数或有符号的 64 位整数或double 类的值**。这么做的好处是可以**节省内存空间**，因为当「值」是整数或浮点数时，就可以将值的数据内嵌在 dictEntry 结构里，无需再用一个指针指向实际的值，从而**节省了内存空间**。

### ~如何解决Hash冲突？

链地址法：**被分配到同一个哈希桶上的多个节点可以用这个单项链表连接起来**，这样就解决了哈希冲突。

### ~rehash

当链表长度过长是会发生rehash，步骤如下：

给hash表2分配空间，然后将每个Entry的hash值重新进行取模运算然后放入哈希表2中，哈希表2的大小一般是表1的两倍，迁移完成之后，将哈希表2设置为hash表1。

**如果「哈希表 1 」的数据量非常大，那么在迁移至「哈希表 2 」的时候，因为会涉及大量的数据拷贝，此时可能会对 Redis 造成阻塞，无法服务其他请求**。

### ~渐进式 rehash

为了避免 rehash 在数据迁移过程中，因拷贝数据的耗时，影响 Redis 性能的情况，所以 Redis 采用了**渐进式 rehash**，也就是将数据的迁移的工作不再是一次性迁移完成，而是**分多次迁移。**

- **在 rehash 进行期间，每次哈希表元素进行新增、删除、查找或者更新操作时，Redis 除了会执行对应的操作之外，还会顺序将「哈希表 1 」中索引位置上的所有 key-value 迁移到「哈希表 2」 上**；

在渐进式 rehash 进行期间，**新增一个 key-value 时，会被保存到「哈希表 2 」里面**，而「哈希表 1」 则不再进行任何添加操作，这样保证了「哈希表 1 」的 key-value 数量只会减少，随着 rehash 操作的完成，最终「哈希表 1 」就会变成空表。

**「哈希表 2 」只新增，「哈希表 1 」只减少。**

### ~rehash 触发条件

![img](https://img-blog.csdnimg.cn/img_convert/85f597f7851b90d6c78bb0d8e39690fc.png)

- **当负载因子大于等于 1 ，并且 Redis 没有在执行 bgsave 命令或者 bgrewiteaof 命令，也就是没有执行 RDB 快照或没有进行 AOF 重写的时候，就会进行 rehash 操作。**
- **当负载因子大于等于 5 时，此时说明哈希冲突非常严重了，不管有没有有在执行 RDB 快照或 AOF 重写，都会强制进行 rehash 操作。**



### 常用命令

```shell
# 存储一个哈希表key的键值
HSET key field value   
# 获取哈希表key对应的field键值
HGET key field

# 在一个哈希表key中存储多个键值对
HMSET key field value [field value...] 
# 批量获取哈希表key中多个field键值
HMGET key field [field ...]       
# 删除哈希表key中的field键值
HDEL key field [field ...]    

# 返回哈希表key中field的数量
HLEN key       
# 返回哈希表key中所有的键值
HGETALL key 

# 为哈希表key中field键的值加上增量n
HINCRBY key field n
#判断哈希表key中field键是否存在
hexists  key field
```

## 4、Set（哈希表或整数集合实现）

### 介绍

Set 类型是一个**无序并唯一的键值集合**，它的存储顺序不会按照插入的先后顺序进行存储。

一个集合最多可以存储 `2^32-1` 个元素。概念和数学中个的集合基本类似，可以交集，并集，差集等等，所以 Set 类型除了支持集合内的增删改查，同时还支持多个集合取交集、并集、差集。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/set.png" alt="img" style="zoom: 50%;" />

### 内部实现

Set 类型的底层数据结构是由**哈希表或整数集合**实现的：

- 如果集合中的元素**都是整数**且元素个数**小于 `512`** （默认值，`set-maxintset-entries`配置）个，Redis 会使用**整数集合**作为 Set 类型的底层数据结构；
- 如果集合中的元素不满足上面条件，则 Redis 使用**哈希表**作为 Set 类型的底层数据结构。

### ~什么是整数集合？

```c
typedef struct intset {
    //编码方式
    uint32_t encoding;
    //集合包含的元素数量
    uint32_t length;
    //保存元素的数组
    int8_t contents[];
} intset
```

保存元素的容器是一个 contents 数组，虽然 contents 被声明为 int8_t 类型的数组，但是实际上 contents 数组并不保存任何 int8_t 类型的元素，contents数组里的数组项按值的大小有序排列，**contents 数组的真正类型取决于 intset 结构体里的 encoding 属性的值**。

### ~整数集合升级操作

整数集合会有一个升级规则，就是当我们将一个新元素加入到整数集合里面，如果新元素的类型（int32_t）比整数集合现有所有元素的类型（int16_t）都要长时，整数集合需要先进行升级，**整数集合升级的过程不会重新分配一个新类型的数组，而是在原本的数组上扩展空间，这种升级操作节省了内存资源。**并且升级策略还可以提高整数集合的灵活性，因为它让一个集合中适应了不同的元素。

**注意，不会有降级操作！**

### ~常用命令

```shell
# 往集合key中存入元素，元素存在则忽略，若key不存在则新建
SADD key member [member ...]
# 从集合key中删除元素
SREM key member [member ...] 
# 获取集合key中所有元素
SMEMBERS key
# 获取集合key中的元素个数
SCARD key

# 判断member元素是否存在于集合key中
SISMEMBER key member

# 从集合key中随机选出count个元素，元素不从key中删除
SRANDMEMBER key [count]
# 从集合key中随机选出count个元素，元素从key中删除
SPOP key [count]
```

Set运算操作：

```shell
# 交集运算
SINTER key [key ...]
# 将交集结果存入新集合destination中
SINTERSTORE destination key [key ...]

# 并集运算
SUNION key [key ...]
# 将并集结果存入新集合destination中
SUNIONSTORE destination key [key ...]

# 差集运算
SDIFF key [key ...]
# 将差集结果存入新集合destination中
SDIFFSTORE destination key [key ...]
```

## 5、Zset（压缩列表或跳表实现）

### 介绍

**Zset （sorted set**）类型（有序集合类型）相比于 Set 类型多了一个排序属性 score（分值），对于有序集合 ZSet 来说，每个存储元素相当于有两个值组成的，一个是有序结合的元素值，一个是排序值。

有序集合保留了集合不能有重复成员的特性（**分值可以重复**），但不同的是，**有序集合中的元素可以排序**。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/zset.png" alt="img" style="zoom: 50%;" />

### 内部实现

Zset 类型的底层数据结构是由**压缩列表或跳表**实现的：

- 如果有序集合的元素个数小于 `128` 个，并且每个元素的值小于 `64` 字节时，Redis 会使用**压缩列表**作为 Zset 类型的底层数据结构；
- 如果有序集合的元素不满足上面的条件，Redis 会使用**跳表**作为 Zset 类型的底层数据结构；

**在 Redis 7.0 中，压缩列表数据结构已经废弃了，交由 listpack 数据结构来实现了。**

### ~跳表的设计

链表在查找元素的时候，因为需要逐一查找，所以查询效率非常低，时间复杂度是O(N)，于是就出现了跳表，**跳跃表支持平均O(logN)**。**跳表是在链表基础上改进过来的，实现了一种「多层」的有序链表**，这样的好处是能**快速定位数据。**跳表是升序排列的

<img src="https://img-blog.csdnimg.cn/img_convert/2ae0ed790c7e7403f215acb2bd82e884.png" alt="img" style="zoom:67%;" />

在传统的查找中，如果我们要查找4这个节点，那么需要四次，但基于跳表实现的查询就只需要两次，在大数据量情况下，查询效率大大提高了。

跳表节点的设计：

​																						**80对应score,tom对应ele**

![image-20221013200653179](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221013200653179.png)

```c
typedef struct zskiplistNode {
    //Zset 对象的元素值
    sds ele;
    //元素权重值
    double score;
    //后向指针，指向前一个元素
    struct zskiplistNode *backward;
  
    //节点的level数组，保存每层上的前向指针和跨度
    struct zskiplistLevel {
        struct zskiplistNode *forward;//指向后一个元素，由跨度来决定
        unsigned long span; //跨度
    } level[];
} zskiplistNode;
```

跳表是一个带有层级关系的链表，而且每一层级可以包含多个节点，每一个节点通过指针连接起来，实现这一特性就是靠跳表节点结构体中的**zskiplistLevel 结构体类型的 level 数组**。

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221013192329681.png" alt="image-20221013192329681" style="zoom: 80%;" />

level 数组中的每一个元素代表跳表的一层，也就是由 zskiplistLevel 结构体表示，**比如 leve[0] 就表示第一层**，zskiplistLevel 结构体里定义了「指向下一个跳表节点的指针」和「跨度」，跨度时用来记录两个节点之间的距离。

**查找过程中跨度的累加可以计算目标节点在跳表中的排位**。

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221113135612126.png" alt="image-20221113135612126" style="zoom:47%;" />

```c
//跳表结构体
typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```

- 跳表的头尾节点，便于在O(1)时间复杂度内**访问跳表的头节点和尾节点**；
- 跳表的长度，便于在O(1)时间复杂度获取跳表节点的数量；
- 跳表的最大层数，便于在O(1)时间复杂度获取跳表中层高**最大的那个节点的层数量**

### ~跳表的查找过程

查找一个跳表节点的过程时，跳表会从**头节点的最顶层（`level最大`）开始**，逐一遍历每一层。在遍历某一层的跳表节点时，会用跳表节点中的 **SDS 类型的元素和元素的权重来进行判断**，共有两个判断条件：

- 如果当前节点的权重「**小于**」要查找的权重时，跳表就会访问**该层上的下一个节点**。
- 如果当前节点的权重「**等于**」要查找的权重时，并且当前节点的 **SDS 类型数据「小于」**要查找的数据时，跳表就会访问**该层上的下一个节点。**

如果上面两个条件**都不满足**，或者下一个节点为空时，跳表就会使用**目前遍历到的节点的 level 数组**里的下一层指针（**更高一层**），然后沿着下一层指针继续查找，这就相当于**跳到了下一层**接着查找。

### ~跳表节点层数的设置

**跳表的相邻两层的节点数量最理想的比例是 2:1，查找复杂度可以降低到 O(logN)**。

怎样才能维持相邻两层的节点数量的比例为 2 : 1 呢？

Redis 则采用一种巧妙的方法是，**跳表在创建节点的时候，随机生成每个节点的层数**。在创建节点时为其生成一个[0-1]的随机数，若随机数<0.25，则层数增加一层，然后继续生成下一个随机数，直到随机数>=0.25，最终确定节点高度。**这样的做法，相当于每增加一层的概率不超过 25%，层数越高，概率越低，层高最大限制是 64。**

### ~为什么用跳表而不用平衡树？

- **从查询效率分析：**Zset 经常需要执行 ZRANGE 或 ZREVRANGE 的命令，即作为链表遍历跳表。**在做范围查找的时候，跳表比平衡树操作要简单**。在平衡树上，我们找到指定范围的小值之后，还需要以中序遍历的顺序继续寻找其它不超过大值的节点。而在跳表上进行范围查找就非常简单，**只需要在找到小值之后，对第 1 层链表进行若干步的遍历就可以实现。**

- **从实现难度分析：**从算法实现难度上来比较，跳表比平衡树要简单得多。平衡树的插入和删除操作可能引发子树的调整，逻辑复杂，而跳表的插入和删除只需要**修改相邻节点的指针**，操作简单又快速。

- **从内存占用上来分析：**跳表比平衡树更灵活一些**。**平衡树每个节点包含 2 个指针**（分别指向左右子树），而跳表每个节点包含的指针数目平均为 1/(1-p)，具体取决于参数 p 的大小。如果像 Redis里的实现一样，取 p=1/4，那么**平均每个节点包含 1.33 个指针，比平衡树更有优势。

### 常用命令

```shell
# 往有序集合key中加入带分值元素
ZADD key score member [[score member]...]   
# 往有序集合key中删除元素
ZREM key member [member...]                 
# 返回有序集合key中元素member的分值
ZSCORE key member
# 返回有序集合key中元素个数
ZCARD key 

# 为有序集合key中元素member的分值加上increment
ZINCRBY key increment member 

# 正序获取有序集合key从start下标到stop下标的元素
ZRANGE key start stop [WITHSCORES]
# 倒序获取有序集合key从start下标到stop下标的元素
ZREVRANGE key start stop [WITHSCORES]

# 返回有序集合中指定分数区间内的成员，分数由低到高排序。
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]

# 返回指定成员区间内的成员，按字典正序排列, 分数必须相同。
ZRANGEBYLEX key min max [LIMIT offset count]
# 返回指定成员区间内的成员，按字典倒序排列, 分数必须相同
ZREVRANGEBYLEX key max min [LIMIT offset count]
```

```shell
# 并集计算(相同元素分值相加)，numberkeys一共多少个key，WEIGHTS每个key对应的分值乘积
ZUNIONSTORE destkey numberkeys key [key...] 
# 交集计算(相同元素分值相加)，numberkeys一共多少个key，WEIGHTS每个key对应的分值乘积
ZINTERSTORE destkey numberkeys key [key...]
```

### quickList引入

quicklist 的结构体跟链表的结构体类似，都包含了表头和表尾，区别在于 quicklist 的节点是 quicklistNode。

```c
typedef struct quicklist {
    //quicklist的链表头
    quicklistNode *head;      //quicklist的链表头
    //quicklist的链表尾
    quicklistNode *tail; 
    //所有压缩列表中的总元素个数
    unsigned long count;
    //quicklistNodes的个数
    unsigned long len;       
    ...
} quicklist;

typedef struct quicklistNode {
    //前一个quicklistNode
    struct quicklistNode *prev;     //前一个quicklistNode
    //下一个quicklistNode
    struct quicklistNode *next;     //后一个quicklistNode
    //quicklistNode指向的压缩列表
    unsigned char *zl;              
    //压缩列表的的字节大小
    unsigned int sz;                
    //压缩列表的元素个数
    unsigned int count : 16;        //ziplist中的元素个数 
    ....
} quicklistNode;
```

**链表节点的元素不再是单纯保存元素值，而是保存了一个压缩列表，所以 quicklistNode 结构体里有个指向压缩列表的指针 *zl。**

<img src="https://img-blog.csdnimg.cn/img_convert/f46cbe347f65ded522f1cc3fd8dba549.png" alt="img" style="zoom:67%;" />

在向 quicklist 添加一个元素的时候，不会像普通的链表那样，直接新建一个链表节点。而是会检查插入位置的**压缩列表是否能容纳**该元素，如果能容纳就直接保存到 quicklistNode 结构里的压缩列表，**如果不能容纳，才会新建一个新的 quicklistNode 结构**。

### listpack 结构设计

listpack 采用了压缩列表的很多优秀的设计，比如还是用一块连续的内存空间来紧凑地保存数据，并且为了节省内存的开销，listpack 节点会采用不同的编码方式保存不同大小的数据。

<img src="https://img-blog.csdnimg.cn/img_convert/c5fb0a602d4caaca37ff0357f05b0abf.png" alt="img" style="zoom:67%;" />

每个 listpack 节点结构如下：

- encoding，定义该元素的编码类型，会对不同长度的整数和字符串进行编码；
- data，实际存放的数据；
- len，encoding+data的总长度；

，**listpack 没有压缩列表中记录前一个节点长度的字段了，listpack 只记录当前节点的长度，当我们向 listpack 加入一个新元素的时候，不会影响其他节点的长度字段的变化，从而避免了压缩列表的连锁更新问题**。

## 6、BitMap

### 介绍

一串连续的二进制数组（0和1），可以通过偏移量（offset）定位元素。BitMap通过最小的单位bit来进行`0|1`的设置，表示某个元素的值或者状态，时间复杂度为O(1)。

由于 bit 是计算机中最小的单位，使用它进行储存将非常节省空间，特别适合一些数据量大且使用**二值统计的场景**。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/bitmap.png)

### 内部实现

Bitmap 本身是用 String 类型作为底层数据结构实现的一种统计二值状态的数据类型。

**String 类型是会保存为二进制的字节数组**，所以，**Redis 就把字节数组的每个 bit 位利用起来，用来表示一个元素的二值状态**，你可以把 Bitmap 看作是一个 bit 数组。

### 常用命令

bitmap 基本操作：

```shell
# 设置值，其中value只能是 0 和 1
SETBIT key offset value

# 获取值
GETBIT key offset

# 获取指定范围内值为 1 的个数
# start 和 end 以字节为单位
BITCOUNT key start end
```

bitmap 运算操作：

```shell
# BitMap间的运算
# operations 位移操作符，枚举值
  AND 与运算 &
  OR 或运算 |
  XOR 异或 ^
  NOT 取反 ~
# result 计算的结果，会存储在该key中
# key1 … keyn 参与运算的key，可以有多个，空格分割，not运算只能一个key
# 当 BITOP 处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 0。返回值是保存到 destkey 的字符串的长度（以字节byte为单位），和输入 key 中最长的字符串长度相等。
BITOP [operations] [result] [key1] [keyn…]

# 返回指定key中第一次出现指定value(0/1)的位置
BITPOS [key] [value]
```

## 7、HyperLogLog

Redis HyperLogLog 是 Redis 2.8.9 版本新增的数据类型，是一种**用于「统计基数」的数据集合类型，基数统计就是指统计一个集合中不重复的元素个数。**但要注意，HyperLogLog 是统计规则是基于概率完成的，不是非常准确，标准误算率是 0.81%。

所以，简单来说 HyperLogLog **提供不精确的去重计数**。

在 Redis 里面，**每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 `2^64` 个不同元素的基数**，和元素越多就越耗费内存的 Set 和 Hash 类型相比，HyperLogLog 就非常节省空间。

假设我们要通过用户ID统计用户UV,假设每个用户id用long来存储，一个用户id占8byte字节，如果用set来统计那么 `1000`万个用户就要占去76M内存，但是用HyperLogLog却只需要占去一个字节，大大减少了内存的开销。

```shell
# 添加指定元素到 HyperLogLog 中
PFADD key element [element ...]

# 返回给定 HyperLogLog 的基数估算值。
PFCOUNT key [key ...]

# 将多个 HyperLogLog 合并为一个 HyperLogLog
PFMERGE destkey sourcekey [sourcekey ...]
```

## 8、GEO

Redis GEO 是 Redis 3.2 版本新增的数据类型，主要用于存储地理位置信息，并对存储的信息进行操作。

在日常生活中，我们越来越依赖搜索“附近的餐馆”、在打车软件上叫车，这些都离不开基于位置信息服务（Location-Based Service，LBS）的应用。LBS 应用访问的数据是和人或物关联的一组经纬度信息，而且要能查询相邻的经纬度范围，GEO 就非常适合应用在 LBS 服务的场景中。

GEO 本身并没有设计新的底层数据结构，而是**直接使用了 Sorted Set** 集合类型。

```shell
# 存储指定的地理空间位置，可以将一个或多个经度(longitude)、纬度(latitude)、位置名称(member)添加到指定的 key 中。
GEOADD key longitude latitude member [longitude latitude member ...]

# 从给定的 key 里返回所有指定名称(member)的位置（经度和纬度），不存在的返回 nil。
GEOPOS key member [member ...]

# 返回两个给定位置之间的距离。
GEODIST key member1 member2 [m|km|ft|mi]

# 根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
```

## 9、Stream

Redis Stream 是 Redis 5.0 版本新增加的数据类型，Redis 专门为消息队列设计的数据类型。

在 Redis 5.0 Stream 没出来之前，消息队列的实现方式都有着各自的缺陷，例如：

- **发布订阅模式，不能持久化也就无法可靠的保存消息，假如没有消费者，消息直接丢弃，并且对于离线重连的客户端不能读取历史消息的缺陷；**
- **List 实现消息队列的方式不能重复消费，一个消息消费完就会被删除，而且生产者需要自行实现全局唯一 I**D。

基于以上问题，Redis 5.0 便推出了 **Stream 类型**也是此版本最重要的功能，用于完美地实现消息队列，它支持消息的持久化、支持自动生成全局唯一 ID、支持 ack 确认消息的模式、支持消费组模式等，让消息队列更加的稳定和可靠。

### 常见命令

Stream 消息队列操作命令：

- XADD：插入消息，保证有序，**可以自动生成全局唯一 ID；**
- XLEN ：查询消息长度；
- XREAD：用于读取消息，**可以按 ID 读取数据**；
- XDEL ： 根据消息 ID 删除消息；
- DEL ：删除整个 Stream；
- XRANGE ：**读取区间消息**
- XREADGROUP：按消费组形式读取消息；
- XPENDING 和 XACK：
  - XPENDING 命令可以用来**查询每个消费组内所有消费者「已读取、但尚未确认」的消息**；
  - XACK 命令用于**向消息队列确认消息处理已完成；**

### 常用命令实战

```bash
# 向名为mystream的消息队列中插入一条key为name, value为VictorG的消息
# '*' 代表代表生成一条全局唯一ID
127.0.0.1:6379> xadd mystream * name VictorG
"1665805010384-0"

# 从 ID 号为 1665805010383-0 的消息开始，读取后续的所有消息（示例中一共 1 条）。
xread streams mystream  1665805010383-0
1) 1) "mystream"
   2) 1) 1) "1665805010384-0"
         2) 1) "name"
            2) "VictorG"
#实现阻塞读取
#5000代表阻塞时间，在5s内若有新数据进来将会被读取
xread block 5000 streams mystream $
(nil)
(5.06s)

```

**创建消费者组**

```bash
#0-0表示从第一条消息开始读
127.0.0.1:6379> xgroup create mystream group1 0-0
OK

# 将mymq这个消息队列加入到消费者组group1, 
# $表示从现在开始到达流中的新消息才会提供给组中的消费者。我们也可以指定一个有效ID，会提供给消费者大于指定ID的消息
# mkstream表示如果mymq不存在，则自动为我们创建
127.0.0.1:6379> xgroup create mymq group1 $ mkstream 
OK
#表示修改消费者组要获取的下一个ID，而无需再次删除和创建使用者组
127.0.0.1:6379> xgroup setid mymq group1 $
OK
#删除该消费者组
127.0.0.1:6379> xgroup destroy mymq group1
(integer) 1
```

**添加消费者**

```bash
#在消费者组group1中添加名为G1的消费者
127.0.0.1:6379> xgroup createconsumer mystream group1 G1
(integer) 1
#让消费者G1从消费者组group1中读取所有消息
127.0.0.1:6379> xreadgroup group group1 G1 streams mystream >
1) 1) "mystream"
   2) 1) 1) "1665805010384-0"
         2) 1) "name"
            2) "VictorG"
#再读一次，由于消息已被消费，所以读不到了            
127.0.0.1:6379> xreadgroup group group1 G1 streams mystream >
(nil)
#再创建一个消费者组
127.0.0.1:6379> xgroup create mystream group2 0-0
OK
#不同消费组的消费者可以消费同一条消息（但是有前提条件，创建消息组的时候，不同消费组指定了相同位置开始读取消息
127.0.0.1:6379> xreadgroup group group2 G1 streams mystream >
1) 1) "mystream"
   2) 1) 1) "1665805010384-0"
         2) 1) "name"
            2) "VictorG"
#让一个消费者组中的不同消费者依次读取一条消息
127.0.0.1:6379> xreadgroup group group3 G1 count 1 streams mystream >
1) 1) "mystream"
   2) 1) 1) "1665805010384-0"
         2) 1) "name"
            2) "VictorG"
127.0.0.1:6379> xreadgroup group group3 G2 count 1 streams mystream >
1) 1) "mystream"
   2) 1) 1) "1665806405059-0"
         2) 1) "name"
            2) "dasi"
127.0.0.1:6379> xreadgroup group group3 G3 count 1 streams mystream >
1) 1) "mystream"
   2) 1) 1) "1665806421465-0"
         2) 1) "name"
            2) "liusi"

```

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/%E6%B6%88%E6%81%AF%E7%A1%AE%E8%AE%A4.png)

**查看某个消费者具体读取了哪些数据**

```bash
#查看消费者组group3 G3消费者读取的消息ID
127.0.0.1:6379> xpending mystream group3 - + 10 G3 # [[IDLE min-idle-time] start end count [consumer]]

127.0.0.1:6379> xpending mystream group3 - + 10 G3
1) 1) "1665806421465-0"
   2) "G3"
   3) (integer) 140989
   4) (integer) 1
#一旦消息 1665806421465-0 被 G3 处理了，G3 就可以使用 XACK 命令通知 Streams，然后这条消息就会被删除。 
127.0.0.1:6379> xack mystream group3 1665806421465-0
(integer) 1
127.0.0.1:6379> xpending mystream group3 - + 10 G3
(empty array)

```

### 关于Stream作为消息队列的几个问题

#### **1、Redis Stream 消息会丢失吗？**

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E4%B8%89%E4%B8%AA%E9%98%B6%E6%AE%B5.png" alt="img" style="zoom: 50%;" />

要保证消息不丢失，实际上就是要保证**三个阶段**的消息不丢失：

- 生产者会不会丢消息，取决于生产者对于异常情况的处理是否合理。 从消息被生产出来，然后提交给 MQ 的过程中，**只要能正常收到 （ MQ 中间件） 的 ack 确认响应**，就表示发送成功，所以只要处理好返回值和异常，如果返回异常则进行消息重发，那么这个阶段是不会出现消息丢失的。再向Stream插入消息时，redis会发送ok给客户端，所以这阶段消息不会丢失。

-  Stream （ MQ 中间件）会自动使用**内部队列（也称为 PENDING List）**留存消费组里每个消费者读取的消息，但是未被确认的消息。消费者可以在重启后，**用 XPENDING 命令查看已读取、但尚未确认处理完成的消息。**等到消费者执行完业务逻辑后，再发送消费确认 XACK 命令，也能保证消息的不丢失。

- Redis 消息中间件会不会丢消息？

  会

  ，Redis 在以下 2 个场景下，都会导致数据丢失：

  - AOF 持久化配置为每秒写盘，但这个写盘过程是异步的，Redis 宕机时会存在数据丢失的可能
  - 主从复制也是异步的，主从切换时，也存在丢失数据的可能。 

#### 2、Redis Stream 消息可堆积吗？

**Redis 的数据都存储在内存中**，当消息堆积过多时，可能会引发OOM。

所以 Redis 的 Stream 提供了可以**指定队列最大长度的功能**，就是为了避免这种情况发生。

当指定队列最大长度时，队列长度**超过上限后，旧消息会被删除，只保留固定长度的新消息**。这么来看，Stream 在消息积压时，如果指定了最大长度，还是有可能**丢失消息**的。

因此，把 Redis 当作队列来使用时，会面临的 **2 个问题**：

- **Redis 本身可能会丢数据；**
- **面对消息挤压，内存资源会紧张；**

所以，能不能将 Redis 作为消息队列来使用，关键看你的业务场景：

- 如果你的业**务场景足够简单**，对于数据丢失不敏感，而且消息积压概率比较小的情况下，**把 Redis 当作队列是完全可以的**。
- 如果你的业务有**海量消息**，消息积压的概率比较大，并且不能接受数据丢失，那么还是用**专业的消息队列中间件**，因为向Kafka和RabbitMQ，他们的消息是存储在磁盘上的。

# 二、Redis中各种数据类型应用场景

## 1、String类型的应用场景

### I、缓存对象

- 直接缓存整个对象的 JSON，命令例子： `SET user:1 '{"name":"xiaolin", "age":18}'`。
- 采用将 key 进行分离为 user:ID:属性，采用 MSET 存储，用 MGET 获取各属性值，命令例子： `MSET user:1:name xiaolin user:1:age 18 user:2:name xiaomei user:2:age 20`。

### II、常规的计数器

当String对象的val是数字是，支持使用自增命令，并且redis处理命令是单线程的，所以适合并发计数场景，比如访问次数，点赞，转发次数，商品库存.....

```shell
# 初始化文章的阅读量
> SET aritcle:readcount:1001 0
OK
#阅读量+1
> INCR aritcle:readcount:1001
(integer) 1
#阅读量+1
> INCR aritcle:readcount:1001
(integer) 2
#阅读量+1
> INCR aritcle:readcount:1001
(integer) 3
# 获取对应文章的阅读量
> GET aritcle:readcount:1001
```

### III、分布式session共享（结合项目）

通常我们在开发后台管理系统时，会使用 Session 来保存用户的会话(登录)状态，这些 Session 信息会被保存在服务器端，但这只适用于单系统应用，如果是分布式系统此模式将不再适用。

例如用户一的 Session 信息被存储在服务器一，但第二次访问时用户一被分配到服务器二，这个时候服务器并没有用户一的 Session 信息，就会出现需要重复登录的问题，问题在于分布式系统每次会把请求随机分配到不同的服务器。

#### **~结合我的项目谈一谈**

**1.缓存token实现session共享**

当我们的项目部署在多台服务器上，用户在不同时间段内对我们项目资源的请求会打到不同的服务器上。

比如在我的项目中，我把项目部署在两台服务器上，用Nginx做了负载均衡，按照传统的单体项目，我们会使用**cookie和session（将用户信息放入session域中）**来保存用户的登录状态，并做登录时校验。

```java
session.setAttribute("user",user);
```



但在分布式模式下，由与session保存的用户状态只在一台服务器上起作用，当用户又访问了另一个页面，这个请求打到了另一台服务器，此时session状态不存在，用户被迫重新登录，影响用户体验。所以我在项目中引入redis,用String对象类型来保存用户的信息。随机生成遗传Token作为String对象的key，用户信息（user对象）作为value存储在redis中，并给token设置过期时间，然后把token返回给前端，那么无论如何用户下一次访问页面请求是打到那台服务器上，都是到redis中获取用户信息，实现了session共享。

```java
stringRedisTemplate.opsForValue().putAll(tokenKey, userMap);
  
    stringRedisTemplate.expire(tokenKey, LOGIN_USER_TTL, TimeUnit.MINUTES);
```

**2、缓存用户登录验证码做登录注册**

```java
String code = RandomUtil.randomNumbers(6);//生成验证码
        //before保存验证码到session
        //session.setAttribute("code",code);

        //保存到redis中并设置过期时间(2min)
        stringRedisTemplate.opsForValue().set(RedisConstants.LOGIN_CODE_KEY + phone,code,RedisConstants.LOGIN_CODE_TTL, TimeUnit.MINUTES);
        //发送验证码
        //todo 调用第三方短信发送平台
String cacheCode = stringRedisTemplate.opsForValue().get(RedisConstants.LOGIN_CODE_KEY + loginForm.getPhone());

```

### IV、分布式锁

SET 命令有个 NX 参数可以实现「key不存在才插入」，可以用它来实现分布式锁：

- 如果 key 不存在，则显示插入成功，可以用来表示加锁成功；
- 如果 key 存在，则会显示插入失败，可以用来表示加锁失败。

一般而言，还会对分布式锁加上过期时间，分布式锁的命令如下：

```shell
SET lock_key unique_value NX PX 10000
```

## 2、List类型应用场景

### 应用场景

#### 消息队列

消息队列在存取消息时，必须要满足三个需求，分别是**消息保序、处理重复的消息和保证消息可靠性**。

**~消息保序问题**

List本身就是按照先进先出的顺序进行存取的，List 可以使用 LPUSH + RPOP （或者反过来，RPUSH+LPOP）命令实现消息队列。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/list%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97.png" alt="img" style="zoom:50%;" />



在生产者往 List 中写入数据时，List 并不会主动地通知消费者有新消息写入，如果消费者想要及时处理消息，就需要在程序中不停地调用 `RPOP` 命令（比如使用一个while(1)循环）。如果有新消息写入，RPOP命令就会返回结果，否则，RPOP命令返回空值，再继续循环。

所以，即使没有新消息写入List，消费者也要不停地调用 RPOP 命令，这就会导致消费者程序的 CPU 一直消耗在执行 RPOP 命令上，带来不必要的性能损失。

为了解决这个问题，Redis提供了 BRPOP 命令。**BRPOP命令也称为阻塞式读取，客户端在没有读到队列数据时，自动阻塞，直到有新的数据写入队列，再开始读取新数据**。和消费者程序自己不停地调用RPOP命令相比，这种方式能**节省CPU开销。**

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97.png" alt="img" style="zoom: 50%;" />

***~如何处理重复的消息？***

- 每个消息都有一个**全局唯一的 ID。**
- 消**费者要记录已经处理过的消息的 ID**。当收到一条消息后，消费者程序就可以对比收到的消息 ID 和记录的已处理过的消息 ID，来判断当前收到的消息有没有经过处理。如果已经处理过，那么，消费者程序就**不再进行处理了**。

**List 并不会为每个消息生成 ID 号，所以我们需要自行为每个消息生成一个全局唯一ID**，生成之后，我们在用 LPUSH 命令把消息插入 List 时，需要在消息中包含这个全局唯一 ID。比如：

```shell
LPUSH mq "111000102:stock:99"
```

**~消息可靠性问题**

当消费者程序从 **List 中读取一条消息后，List 就不会再留存这条消息了。**所以，如果消费者程序在处理消息的过程出现了**故障或宕机**，就会导致消息没有处理完成，那么，消费者程序再次启动后，就没法再次从 List 中读取消息了，**消息丢失，可靠性得不到保证。**

为了留存消息，List 类型提供了 `BRPOPLPUSH` 命令，这个命令的**作用是让消费者程序从一个 List 中读取消息，同时，Redis 会把这个消息再插入到另一个 List（可以叫作备份 List）留存**。

**总结：**

- 消息保序：使用 LPUSH + RPOP；
- 阻塞读取：使用 BRPOP；
- 重复消息处理：生产者**自行实现**全局唯一 ID；
- 消息的可靠性：使用 BRPOPLPUSH

#### List 作为消息队列有什么缺陷？

**List 不支持多个消费者消费同一条消息**，因为一旦消费者拉取一条消息后，这条消息就从 List 中删除了，无法被其它消费者再次消费。

要实现一条消息可以被多个消费者消费，那么就要将多个消费者组成一个消费组，使得多个消费者可以消费同一条消息，但是 **List 类型并不支持消费组的实现**。

## 3、Hash类型对象应用场景

#### 缓存对象（结合项目）

Hash 类型的 （key，field， value） 的结构与对象的（对象id， 属性， 值）的结构相似，也可以用来**存储对象**。

我们可以使用如下命令，将用户对象的信息存储到 Hash 类型：

```shell
# 存储一个哈希表uid:1的键值
> HSET uid:1 name Tom age 15
2
# 存储一个哈希表uid:2的键值
> HSET uid:2 name Jerry age 13
2
# 获取哈希表用户id为1中所有的键值
> HGETALL uid:1
1) "name"
2) "Tom"
3) "age"
4) "15"
```

```java
stringRedisTemplate.opsForHash().putAll(tokenKey, userMap);//将对象放入Hash结构中，对象的每一个属性和值对应Hash对象中的一个Entry
```

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/hash%E5%AD%98%E5%82%A8%E7%BB%93%E6%9E%84.png" alt="img" style="zoom:50%;" />

#### 购物车

以**用户 id 为 key**，**商品 id 为 field**，**商品数量为 value**，恰好构成了购物车的3个要素，如下图所示。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/%E8%B4%AD%E7%89%A9%E8%BD%A6.png" alt="img" style="zoom: 33%;" />

涉及的命令如下：

- 添加商品：`HSET cart:{用户id} {商品id} 1`
- 添加数量：`HINCRBY cart:{用户id} {商品id} 1`
- 商品总数：`HLEN cart:{用户id}`
- 删除商品：`HDEL cart:{用户id} {商品id}`
- 获取购物车所有商品：`HGETALL cart:{用户id}`

当前仅仅是将商品ID存储到了Redis 中，在回显商品具体信息的时候，还需要拿着商品 id 查询一次数据库，获取完整的商品的信息

## 4、Set类型对象应用场景

集合的主要几个特性，**无序、不可重复、支持并交差**等操作。

这里有一个潜在的风险。**Set 的差集、并集和交集的计算复杂度较高，在数据量较大的情况下，如果直接执行这些计算，会导致 Redis 实例阻塞**。

#### 点赞

Set 类型可以保证一个用户只能点一个赞，这里举例子一个场景，**key 是文章id，value 是用户id**。

```shell
# 当第一个用户点赞该文章时创建该对象，key 是文章id，value 是用户id，

# uid:1 用户对文章 article:1 点赞
> SADD article:1 uid:1
(integer) 1
# uid:2 用户对文章 article:1 点赞
> SADD article:1 uid:2
(integer) 1
# uid:3 用户对文章 article:1 点赞
> SADD article:1 uid:3
(integer) 1

#当用户取消了对 article:1 文章点赞。
> SREM article:1 uid:1
(integer) 1

#返回给前端的数据
> SCARD article:1 #点赞数
> SMEMBERS article:1 #所有点赞用户的id
```

#### 共同关注（结合项目）

Set 类型支持交集运算，所以可以用来计算共同关注的好友、公众号等。

key 可以是用户id，value 则是已关注的公众号的id。

**`uid:1` 用户关注公众号 id 为 5、6、7、8、9，`uid:2` 用户关注公众号 id 为 7、8、9、10、11。**

```shell
# uid:1 用户关注公众号 id 为 5、6、7、8、9
> SADD uid:1 5 6 7 8 9
(integer) 5
# uid:2  用户关注公众号 id 为 7、8、9、10、11
> SADD uid:2 7 8 9 10 11
(integer) 5
```

**`uid:1` 和 `uid:2` 共同关注的公众号：**

```shell
# 获取共同关注
> SINTER uid:1 uid:2
1) "7"
2) "8"
3) "9"
```

**在我的项目中也实现了共同关注的功能：**

用户关注了某位用户后，需要将数据放入到set集合中，方便后续进行共同关注，同时当取消关注时，也需要从set集合中进行删除

**~关注与取关实现过程：**

当用户点击关注按钮的时候，前端传来两个信息：

1.是取关还是关注（true or false）

2.要关注的用户id(followUserId)

​	**如果是关注:**

 ```java
 Long userId = UserHolder.getUser().getId();
     String key = "follows:" + userId;
 
 // 把关注用户的id，放入redis的set集合 sadd userId followerUserId
             stringRedisTemplate.opsForSet().add(key, followUserId.toString());
 ```

​	**如果是取关：**

```java
// 把关注用户的id从Redis集合中移除
            stringRedisTemplate.opsForSet().remove(key, followUserId.toString());
```

**注意：关注与取关时操作redis前都必须先去操作mysql，mysql中有一张表记录了用户与其所关注的用户之间的关系**

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221014161250503.png" alt="image-20221014161250503" style="zoom: 80%;" />

**~共同关注实现细节**

当用户1进入用户2的主页，查看共同关注，前端会把用户2的id返回到后端，后端通过生成两个Set类型的key，来求交集

```java
// 1.获取当前用户
    Long userId = UserHolder.getUser().getId();
    String key = "follows:" + userId;
    // 2.求交集
    String key2 = "follows:" + id;
    Set<String> intersect = stringRedisTemplate.opsForSet().intersect(key, key2);//返回的是一串数字，即共同关注的用户id的集合
```

```shell
SINTER uid:1 uid:2
1) "7"
2) "8"
3) "9"
```

**然后处理数据通过这些用户id调用UserService去查询用户对象，返回给前端，最终实现了共同关注。**

#### 抽奖活动

存储某活动中中奖的用户名 ，Set 类型因为有去重功能，可以保证同一个用户不会中奖两次。

key为抽奖活动名，value为员工名称，把所有员工名称放入抽奖箱 ：

```shell
>SADD lucky Tom Jerry John Sean Marry Lindy Sary Mark
(integer) 5
```

如果允许重复中奖，可以使用 **SRANDMEMBER** 命令。

```shell
# 抽取 1 个一等奖：
> SRANDMEMBER lucky 1
1) "Tom"
# 抽取 2 个二等奖：
> SRANDMEMBER lucky 2
1) "Mark"
2) "Jerry"
# 抽取 3 个三等奖：
> SRANDMEMBER lucky 3
1) "Sary"
2) "Tom"
3) "Jerry"
```

如果不允许重复中奖，可以使用 **SPOP** 命令。

```shell
# 抽取一等奖1个
> SPOP lucky 1
1) "Sary"
# 抽取二等奖2个
> SPOP lucky 2
1) "Jerry"
2) "Mark"
# 抽取三等奖3个
> SPOP lucky 3
1) "John"
2) "Sean"
3) "Lindy"
```

## 5、Zset类型对象应用场景

Zset 类型（Sorted Set，有序集合） 可以根据元素的权重来排序，我们可以自己来决定每个元素的权重值。比如说，我们可以根据元素插入 Sorted Set 的时间确定权重值，先插入的元素权重小，后插入的元素权重大。

在面对需要展示**最新列表、排行榜**等场景时，如果数据更新频繁或者需要分页显示，可以优先考虑使用 Sorted Set。

#### 排行榜（结合项目）

有序集合比较典型的使用场景就是**排行榜**。例如学生成绩的排名榜、游戏积分排行榜、视频播放排名、电商系统中商品的销量排名等。

在探店笔记的详情页面，应该把给该笔记点赞的人显示出来，比如最早点赞的TOP5，形成点赞排行榜：

**点赞与显示TOP5（按照点赞时间排序）排行的的具体实现：**



**当前端进入一篇博客时，会将发布该博客的id传给后端，该篇博客的id可用于生成Zset的Key,将当前时间作为分数，点赞的用户Id作为value生成一个Zset对象，Zset会根据时间从小到大排序**

```java
public Result likeBlog(Long id) {
        Long userId = UserHolder.getUser().getId();
        String key =  RedisConstants.BLOG_LIKED_KEY + id;
        Double score = stringRedisTemplate.opsForZSet().score(key, String.valueOf(userId));
        if(score == null) {//说明未点赞
            boolean isSuccess = update().setSql("liked = liked + 1").eq("id", id).update();
            if (isSuccess) {
                //保存到redis中
                stringRedisTemplate.opsForZSet().add(key,String.valueOf(userId),System.currentTimeMillis());
            }
        } else {
            boolean isSuccess = update().setSql("liked = liked - 1").eq("id", id).update();
            if (isSuccess) {
                stringRedisTemplate.opsForZSet().remove(key,String.valueOf(userId));
            }
        }
        return Result.ok();
    }
```

抽象为redis命令：

```shell
ZADD blog:liked:[id] [currentTime] [UserId]
(integer) 1
```

**回显给用户的就是按照点赞时间排序的用户前五名，用ZRANGE实现**

```java
public Result queryBlogLikes(Long id) {
        String key = RedisConstants.BLOG_LIKED_KEY + id;
        //查询top5点赞用户,top5是用户id的字符串类型
        Set<String> top5 = stringRedisTemplate.opsForZSet().range(key, 0, 4);//返回的是点赞用户id的集合
        if(top5 == null || top5.isEmpty()) {
            return Result.ok(Collections.emptyList());
        }
        List<Long> ids = top5.stream().map(Long::valueOf).collect(Collectors.toList());
        //解析出用户id，根据用户id查询用户，返回用户信息
        String idStr = StrUtil.join(",", ids);
        List<UserDTO> users = userService.query().in("id",ids).last("ORDER BY FIELD(id," +idStr +")")
                .list()
                .stream().
                map(user -> BeanUtil.copyProperties(user,UserDTO.class)).collect(Collectors.toList());
        return Result.ok(users);
    }
```

抽象为redis命令：

```shell
ZREVRANGE blog:liked:[id] 0 4 #返回的是点赞用户id的集合
```

**tb_blog**

![image-20221014172047836](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221014172047836.png)

#### 电话、姓名排序

使用有序集合的 `ZRANGEBYLEX` 或 `ZREVRANGEBYLEX`（**通过正则表达式**） 可以帮助我们实现电话号码或姓名的排序，我们以 `ZRANGEBYLEX` （返回指定成员区间内的成员，**按 member正序排列**，**分数必须相同**）为例。

```shell
zadd names 0 Toumas 0 Jake 0 Bluetuo 0 Gaodeng 0 Aimini 0 Aidehua 
(integer) 6
```

获取所有人的名字:

```shell
> ZRANGEBYLEX names - +
1) "Aidehua"
2) "Aimini"
3) "Bluetuo"
4) "Gaodeng"
5) "Jake"
6) "Toumas"
```

获取名字中大写字母A开头的所有人：

```shell
> ZRANGEBYLEX names [A (B
1) "Aidehua"
2) "Aimini"
```

获取名字中大写字母 C 到 Z 的所有人：

```shell
> ZRANGEBYLEX names [C [Z
1) "Gaodeng"
2) "Jake"
3) "Toumas"
```

#### 实现Feed流（结合项目）

使用推模式，当user1发布一篇探店博客时，先查询关注了user1的所有用户，然后以**user1的followerId作为key,博客id作为value,当前时间作为score**,写入一个Zset对象中。

当user2点击“我的关注”图标时候，就会以自己的id为key去查询redis，返回的是按时间顺序由大到小排列好的blogId，这时就可以去查询出多条内容封装成博客对象返回给前端。

```java
public Result saveBlog(Blog blog) {
    // 1.获取登录用户
    UserDTO user = UserHolder.getUser();
    blog.setUserId(user.getId());
    // 2.保存探店笔记
    boolean isSuccess = save(blog);
    if(!isSuccess){
        return Result.fail("新增笔记失败!");
    }
    // 3.查询笔记作者的所有粉丝 select * from tb_follow where follow_user_id = ?
    List<Follow> follows = followService.query().eq("follow_user_id", user.getId()).list();
    // 4.推送笔记id给所有粉丝
    for (Follow follow : follows) {
        // 4.1.获取粉丝id
        Long userId = follow.getUserId();
        // 4.2.推送
        String key = FEED_KEY + userId;
        stringRedisTemplate.opsForZSet().add(key, blog.getId().toString(), System.currentTimeMillis());
    }
    // 5.返回id
    return Result.ok(blog.getId());
}
```

## 6、BitMap类型对象应用场景

#### 签到统计（结合项目）

在签到打卡的场景中，我们只用记录签到（1）或未签到（0），所以它就是非常典型的二值状态。

签到统计时，每个用户一天的签到用 1 个 bit 位就能表示，一个月（假设是 31 天）的签到情况用 31 个 bit 位就可以，而一年的签到也只需要用 365 个 bit 位，根本不用太复杂的集合类型。

假设我们要统计 ID 100 的用户在 2022 年 6 月份的签到情况，就可以按照下面的步骤进行操作。

第一步，执行下面的命令，记录该用户 6 月 3 号已签到。

```shell
SETBIT uid:sign:100:202206 2 1
```

第二步，检查该用户 6 月 3 日是否签到。

```shell
GETBIT uid:sign:100:202206 2 
```

第三步，统计该用户在 6 月份的签到次数。

```shell
BITCOUNT uid:sign:100:202206
```

**业务需求：当用户当天登录时，触发签到，将用户id和当前年月作为key，然后获取目前是第几月的第几天作为偏移量，将对应位置的bit位设为1。**

```java
//对应redis命令：set bit [sign: + userId + yyyyMM] [day - 1]
@Override
public Result sign() {
    // 1.获取当前登录用户
    Long userId = UserHolder.getUser().getId();
    // 2.获取日期
    LocalDateTime now = LocalDateTime.now();
    // 3.拼接key
    String keySuffix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
    String key = USER_SIGN_KEY + userId + keySuffix;
    // 4.获取今天是本月的第几天
    int dayOfMonth = now.getDayOfMonth();
    // 5.写入Redis SETBIT key offset 1
    stringRedisTemplate.opsForValue().setBit(key, dayOfMonth - 1, true);
    return Result.ok();
}
```

**统计目前签到次数：**

BITCOUNT  [sign: + userId + yyyyMM]

**统计这个月首次打卡时间**

Redis 提供了 `BITPOS key bitValue [start] [end]`指令，返回数据表示 Bitmap 中第一个值为 `bitValue` 的 offset 位置。

在默认情况下， 命令将检测整个位图， 用户可以通过可选的 `start` 参数和 `end` 参数指定要检测的范围。所以我们可以通过执行这条命令来获取 userID = 100 在 2022 年 6 月份**首次打卡**日期：

```text
BITPOS  [sign: + userId + yyyyMM] 1
```

#### 判断用户登陆态

Bitmap 提供了 `GETBIT、SETBIT` 操作，通过一个偏移值 offset 对 bit 数组的 offset 位置的 bit 位进行读写操作，需要注意的是 offset 从 0 开始。

只需要一个 key = login_status 表示存储用户登陆状态集合数据， 将用户 ID 作为 offset，在线就设置为 1，下线设置 0。通过 `GETBIT`判断对应的用户是否在线。 50000 万 用户只需要 6 MB 的空间。

假如我们要判断 ID = 10086 的用户的登陆情况：

第一步，执行以下指令，表示用户已登录。

```shell
SETBIT login_status 10086 1
```

第二步，检查该用户是否登陆，返回值 1 表示已登录。

```text
GETBIT login_status 10086
```

第三步，登出，将 offset 对应的 value 设置成 0。

```shell
SETBIT login_status 10086 0
```

#### 连续签到用户总数

我们把每天的日期作为 Bitmap 的 key，userId 作为 offset，若是打卡则将 offset 位置的 bit 设置成 1。

key 对应的集合的每个 bit 位的数据则是一个用户在该日期的打卡记录。

一共有 7 个这样的 Bitmap，如果我们能对这 7 个 Bitmap 的对应的 bit 位做 『与』运算。同样的 UserID offset 都是一样的，当一个 userID 在 7 个 Bitmap 对应对应的 offset 位置的 bit = 1 就说明该用户 7 天连续打卡。

假设要统计 3 天连续打卡的用户数，则是将三个 bitmap 进行 AND 操作，并将结果保存到 destmap 中，接着对 destmap 执行 BITCOUNT 统计，如下命令：

```shell
# 与操作
BITOP AND destmap bitmap:01 bitmap:02 bitmap:03
# 统计 bit 位 =  1 的个数
BITCOUNT destmap
```

即使一天有一个亿个用户签到，Bitmap 占用的内存也不大，大约占 12 MB 的内存（10^8/8/1024/1024），7 天的 Bitmap 的内存开销约为 84 MB。同时我们最好给 Bitmap 设置过期时间，**让 Redis 删除过期的打卡数据**，节省内存。

## 7、HyperLogLog类型对象应用场景

#### 百万级网页 UV 计数（结合项目）

Redis HyperLogLog 优势在于只需要花费 12 KB 内存，就可以计算接近 2^64 个元素的基数，和元素越多就越耗费内存的 Set 和 Hash 类型相比，HyperLogLog 就非常节省空间。

所以，非常适合统计百万级以上的网页 UV 的场景。

在统计 UV 时，你可以用 PFADD 命令（用于向 HyperLogLog 中添加新元素）把访问页面的每个用户都添加到 HyperLogLog 中。

```shell
PFADD page1:uv user1 user2 user3 user4 user5
```

接下来，就可以用 PFCOUNT 命令直接获得 page1 的 UV 值了，这个命令的作用就是返回 HyperLogLog 的统计结果。

```shell
PFCOUNT page1:uv
```

```java
stringRedisTemplate.opsForHyperLogLog().add(key, String.valueOf(userId));
Long UV = stringRedisTemplate.opsForHyperLogLog().size(key);
```

## 8、GEO应用场景

### 附近商户的实现（结合项目）

```shell
GEOADD shop1:locations 116.034579 39.030452 33 #把商铺类型为1的商家的经纬度插入
GEORADIUS shop1:locations 116.054579 39.030452 5 km ASC COUNT 10 #用户查找以这个经纬度为中心的 5 公里内的商铺信息，并返回给 LBS 应用
```

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221014195501529.png" alt="image-20221014195501529" style="zoom:50%;" />

**提前把商铺信息按商户类型放入redis中**

```java
// 3.3.写入redis GEOADD key 经度 纬度 member
for (Shop shop : value) {
            // stringRedisTemplate.opsForGeo().add(key, new Point(shop.getX(), shop.getY()), shop.getId().toString());
            locations.add(new RedisGeoCommands.GeoLocation<>(
                    shop.getId().toString(),
                    new Point(shop.getX(), shop.getY())
            ));
        }
		stringRedisTemplate.opsForGeo().add(key, locations);
    }
```

前端会传来自己所在的经纬度、商铺类型、分页号，之后我们就可以操作redis实现我们的业务逻辑

```java
@GetMapping("/of/type")
    public Result queryShopByType(
            @RequestParam("typeId") Integer typeId,
            @RequestParam(value = "current", defaultValue = "1") Integer current,
            @RequestParam(value = "x",required = false) Double x,
            @RequestParam(value = "y",required = false) Double y
    ) {
        return shopService.queryShopByType(typeId,current,x,y);
    }
```

## 9、Stream应用场景（结合项目）！！重点

## //todo

## 三、持久化篇

## 1、AOF 日志

Redis 每执行一条写操作命令，就把该命令以追加的方式写入到一个文件里，然后重启 Redis 的时候，先去读取这个文件里的命令，并且执行它，这就相当于恢复了缓存数据。

<img src="https://img-blog.csdnimg.cn/img_convert/6f0ab40396b7fc2c15e6f4487d3a0ad7.png" alt="img" style="zoom:67%;" />

这种保存写操作命令到日志的持久化方式，就是 Redis 里的 **AOF(\*Append Only File\*)** 持久化功能，**注意只会记录写操作命令，读操作命令是不会被记录的**，因为没意义。

在 Redis 中 AOF 持久化功能默认是不开启的，需要我们修改 `redis.conf` 配置文件中的以下参数：

```shell
appendonly yes # 表示打开aof文件，默认关闭

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"
```

AOF 日志文件其实就是普通的文本，我们可以通过 `cat` 命令查看里面的内容

```shell
*3 #表示当前命令有三个部分
$3 #每部分都是以「$+数字」开头
set #具体命令，「$3 set」表示这部分有 3 个字节，也就是「set」命令这个字符串的长度。
$3 #key长度为3
k11 #key
$3 #value长度为3
v11 #value
...
```

#### I.好处与风险

**Redis 是先执行写操作命令后，才将该命令记录到 AOF 日志里的，这么做其实有两个好处：**

1. 第一个好处，**避免额外的检查开销。**如果先写入aof但是该命令语法有问题，该条命令无法执行，这是就要将写入aof的错误命令擦除，增加了额外检查的开销。
2. 第二个好处，**不会阻塞当前写操作命令的执行**，因为当写操作命令执行成功后，才会将命令记录到 AOF 日志。

**AOF 持久化功能也有潜在风险：**

1. **数据丢失的风险**，如果在命令执行完成但还为写入aof文件的那瞬间服务挂了，那么刚才执行的那条数据就丢失了。
2. **可能会给「下一个」命令带来阻塞风险**，因为执行命令和写日志都是在主线程完成的，如果服务器的磁盘IO压力过大，那么就会导致写AOF文件时阻塞，使后面的命令无法进行。

<img src="https://img-blog.csdnimg.cn/img_convert/28afd536c57a46447ddab0a2062abe84.png" alt="img" style="zoom:67%;" />

#### II、AOF的三种写回策略

<img src="https://img-blog.csdnimg.cn/img_convert/4eeef4dd1bedd2ffe0b84d4eaa0dbdea.png" alt="img" style="zoom:67%;" />

AOF文件的写回流程：

1. 当redis命令执行完成后，会把该条命令写入**server.aof_buf缓冲区**
2. 陷入内核，执行write系统调用，将server.aof_buf缓冲区里的数据写入**aof文件的page cache**,
3. 根据参数来决定何时将**aof文件内核缓冲区(page cache)**里的数据写入硬盘

#### 写回策略相关参数说明

在 `redis.conf` 配置文件中的 `appendfsync` 配置项可以有以下 **3 种参数**可填：

- **Always**，这个单词的意思是「总是」，所以它的意思是每次写操作命令执行完后，同步将 AOF 日志数据写回硬盘；
- **Everysec**，这个单词的意思是「每秒」，所以它的意思是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区（page cache），然后**每隔一秒将缓冲区里的内容同步到硬盘，并且这个同步操作由一个线程专门负责执行**；
- **No**，意味着不由 Redis 控制写回硬盘的时机，转交给操作系统控制写回的时机，也就是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，再由操作系统决定何时将缓冲区内容写回硬盘。

```shell
# appendfsync always
appendfsync everysec	#默认为每秒写入
# appendfsync no
```

这 3 种写回策略都**无法能完美解决「主进程阻塞」和「减少数据丢失」的问题**，因为两个问题是对立的，偏向于一边的话，就会要牺牲另外一边

- 如果要高性能，就选择 No 策略；
- 如果要高可靠，就选择 Always 策略；
- 如果允许数据丢失一点，但又想性能高，就选择 Everysec 策略

<img src="https://img-blog.csdnimg.cn/img_convert/98987d9417b2bab43087f45fc959d32a.png" alt="img" style="zoom: 67%;" />

深入到源码后，你就会发现这三种策略只是在**控制 `fsync()` 函数的调用时机**。

**fsync函数：**

*fsync函数只对由文件描述符filedes指定的单一文件起作用，并且等待写磁盘操作结束，然后返回。*

*fsync可用于数据库这样的应用程序，这种应用程序需要确保将修改过的块立即写到磁盘上。*



当应用程序向文件写入数据时，内核通常先将数据复制到内核缓冲区中，然后排入队列，然后由内核决定何时写入硬盘。

- Always 策略就是每次写入 AOF 文件数据后，就执行 fsync() 函数；
- Everysec 策略就会创建一个异步任务来执行 fsync() 函数；
- No 策略就是永不执行 fsync() 函数

#### III.AOF重写机制（AOF文件变大时触发）

写入 AOF 日志的操作虽然是在主进程完成的，因为它写入的内容不多，所以一般不太影响命令的操作。

但当启用AOF重写机制，比如文件大于64M,此时会**重新读取所有缓存在内存中的键值对数据（无需对已有的AOF文件进行任何的读入，分析或写入）**，然后为他们生成命令，最后将这些命令写入一个新的AOF文件，此时旧的AOF文件就被替换**（替换操作其实是由父进程来完成的，因为他要将AOF重写缓冲区里的数据追加到子进程重写完的AOF文件中）**了。当然，这个**过程是十分耗时**的。

所以，Redis 的**重写 AOF 过程是由后台子进程 bgrewriteaof来完成的**，这么做可以达到两个好处：

1. 自进程在进行aof重写时不会阻塞父进程，所以父进程可以继续处理请求
2. 子进程自带父进程的数据副本，如果使用的是**子线程**，那么数据是共享的，这是就需要**加锁来保证数据的安全性，会降低性能**，但如果使用子进程，父子进程虽然共享内存，但这**个共享区域是只读**的，当父进程对共享内存区域的数据进行修改的时候，就会发生**写时复制**，于是复制进程就有了独立的数据区域，不需要加锁。

**子进程是怎么拥有主进程一样的数据副本的呢？**

主进程在通过 `fork` 系统调用生成 bgrewriteaof 子进程时，操作系统会把主进程的「**页表**」复制一份给子进程，这个页表记录着虚拟地址和物理地址映射关系，而不会复制物理内存，也就是说，两者的虚拟空间不同，但其对应的物理空间是同一个。

<img src="https://img-blog.csdnimg.cn/img_convert/5a1f2a90b5f3821c19bea3b7a5f27fa1.png" alt="img" style="zoom:67%;" />

不过，当父进程或者子进程在向这个内存发起写操作时，CPU 就会触发**缺页中断**，这个缺页中断是由于违反权限导致的，然后操作系统会在「缺页异常处理函数」里进行**物理内存的复制**，并重新设置其内存映射关系，将父子进程的内存读写权限设置为**可读写**，最后才会对内存进行写操作，这个过程被称为**写时复制(Copy On Write）**。

<img src="https://img-blog.csdnimg.cn/img_convert/d4cfac545377b54dd035c775603b4936.png" alt="img" style="zoom:67%;" />

写时复制顾名思义，**在发生写操作的时候，操作系统才会去复制物理内存**，这样是为了防止 fork 创建子进程时，由于物理内存数据的复制时间过长而导致父进程长时间阻塞的问题。

**有两个阶段会导致阻塞父进程：**

- 创建子进程的途中，由于要复制父进程的页表等数据结构，阻塞的时间跟页表的大小有关，**页表越大，阻塞的时间也越长**；
- 创建完子进程后，如果子进程或者父进程修改了共享数据，就会发生写时复制，这期间会拷贝物理内存，**如果内存越大，自然阻塞的时间也越长**；

**还有一个问题：**

如果关于一个key-value的数据生成命令已经被子进程写入了aof重写缓冲区，但此时父进程修改了这个key-value数据，那么父子进程的数据就不一致了。

为了解决这种数据不一致问题，Redis 设置了一个 **AOF 重写缓冲区**，这个缓冲区在创建 bgrewriteaof 子进程之后开始使用。

在重写 AOF 期间，当 Redis 执行完一个写命令之后，它会**同时将这个写命令写入到 「AOF 缓冲区」和 「AOF 重写缓冲区」**。

<img src="https://img-blog.csdnimg.cn/202105270918298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODI3Njc0,size_16,color_FFFFFF,t_70#pic_center" alt="在这里插入图片描述" style="zoom: 50%;" />

也就是说，在 bgrewriteaof 子进程执行 AOF 重写期间，主进程需要执行以下**三个工作**:

- 执行客户端发来的命令；
- 将执行后的写命令追加到 「AOF 缓冲区」；
- 将执行后的写命令追加到 「AOF 重写缓冲区」；

这样一来可以保证：

- AOF缓冲区的内容会定期被写入和同步到AOF文件，**对现有的AOF文件的处理工作照常进行**，也就是说，即使后台子进程在重写AOF，父进程不管，照样继续写自己的AOF文件，你写你的，我写我的。
- 从子进程创建开始，服务器执行所有写命令都会被记录到AOF缓冲区

当子进程完成 AOF 重写工作后，会**向主进程发送一条信号**，信号是进程间通讯的一种方式，且是异步的。

**主进程收到该信号后，会调用一个信号处理函数**，该函数主要做以下工作：

- 将 AOF **重写缓冲区**中的所有内容追加到**新的 AOF 的文件中**，**使得新旧两个 AOF 文件所保存的数据库状态一致**；
- 新的 AOF 的文件进行改名，覆盖现有的 AOF 文件。

在整个 AOF 后台重写过程中，除了发生写时复制会对主进程造成阻塞，还有信号处理函数执行时也会对主进程造成阻塞，在其他时候，AOF 后台重写都不会阻塞主进程。

## 2、RDB快照的实现

RDB方式是将内存中的数据的快照以**二进制的方式写入名字为 dump.rdb的文件中**。我们对 Redis 进行设置， 让它根据设置周期性自动保存数据集。**因为是二进制文件，所以rdb恢复数据的速度要高于aof**

#### I、RDB快照怎么用？

Redis 提供了两个命令来生成 RDB 文件，分别是 `save` 和 `bgsave`，他们的区别就在于是否在「主线程」里执行：

- 执行了 save 命令，就会在主线程生成 RDB 文件，由于和执行操作命令在同一个线程，所以如果写入 RDB 文件的时间太长，**会阻塞主线程**；
- 执行了 bgsave 命令，会创建一个子进程来生成 RDB 文件，这样可以**避免主线程的阻塞**；

Redis 还可以通**过配置文件的选项**来实现每隔一段时间自动执行一次 **bgsave** 命令，默认会提供以下配置：

```text
# You can set these explicitly by uncommenting the three following lines.
#
# save 3600 1
# save 300 100
# save 60 10000
```

只要满足上面条件的任意一个，就会执行 bgsave，它们的意思分别是：

- 3600 秒之内，对数据库进行了至少 1 次修改；
- 300 秒之内，对数据库进行了至少 10 次修改；
- 60 秒之内，对数据库进行了至少 10000 次修改。

Redis 的快照是**全量快照**，也就是说每次执行快照，**都是把内存中的「所有数据」都记录到磁盘中。**

所以可以认为，执行快照是一个比较重的操作，如果频率太频繁，可能会对 Redis 性能产生影响。如果频率太低，服务器故障时，丢失的数据会更多。

通常可能设置**至少 5 分钟才保存一次快照**，这时如果 Redis 出现宕机等情况，则意味着最多可能丢失 5 分钟数据。

#### II.执行快照时，数据能被修改吗？

可以的。在执行bgsave后，redis主进程会创建子进程来写rdb文件，Redis 依然**可以继续处理操作命令**的，与aof重写一样，也是利用了**写时复制技术**来实现内存的隔离。

但是，如果主线程（父进程）要**修改共享数据里的某一块数据**（比如键值对 `A`）时，就会发生写时复制，于是这块数据的**物理内存就会被复制一份（键值对 `A'`）**，然后**主线程在这个数据副本（键值对 `A'`）进行修改操作**。与此同时，**bgsave 子进程可以继续把原来的数据（键值对 `A`）写入到 RDB 文件**。

**一句话：父进程修改的是数据副本，子进程写的是原数据**，所以 Redis 在使用 bgsave 快照过程中，如果主线程修改了内存数据，不管是否是共享的内存数据，RDB 快照都**无法写入主线程刚修改的数据**

**！注意极端情况：**

如果在快照过程中，父进程修改了所有共享内存数据，那么内存占用量就会变成原来的两倍，所以在快照过程中要**时刻注意内存占用情况。**

#### III、AOF和RDB混用

尽管 RDB 比 AOF 的数据恢复速度快，但是快照的频率不好把握：

- 如果频率太低，两次快照间一旦服务器发生宕机，就可能会比较多的数据丢失；
- 如果频率太高，频繁写入磁盘和创建子进程会带来额外的性能开销。

Redis 4.0 提出**混合使用 AOF 日志和内存快照**，也叫混合持久化。**混合持久化可以对中和RDB和AOF的优缺点。**

如果想要开启混合持久化功能，可以在 Redis 配置文件将下面这个配置项设置成 yes：

```text
aof-use-rdb-preamble yes
```

当开启了混合持久化时，在 AOF 重写日志时，`fork` 出来的重写子进程会先将与主线程**共享的内存**数据**以 RDB 方式写入到 AOF 文件（此时AOF重写文件中的数据是二进制形式）**，然后主线程处理的操作命令会被记录在重写缓冲区里，重写缓冲区里的增量命令会以 AOF 方式写入到 AOF 文件，写入完成后通知主进程将新的含有 RDB 格式和 AOF 格式的 AOF 文件替换旧的的 AOF 文件。

一句话概括:AOF 文件的**前半部分是 RDB 格式的全量数据，后半部分是 AOF 格式的增量数据**。

**RDB段的内容是在重写过程开始时的所有共享数据，AOF中的数据是重写过程中新产生的数据和修改过的数据。**

<img src="https://img-blog.csdnimg.cn/img_convert/f67379b60d151262753fec3b817b8617.png" alt="图片" style="zoom:67%;" />

**这种方式，数据恢复速度快，数据丢失少。**

## 3、Redis大key在持久化过程中的影响

### I、对AOF日志的影响

**根据三种写回策略来分析：**

- Always 策略就是每次写入 AOF 文件数据后，就执行 fsync() 函数；
- Everysec 策略就会创建一个异步任务来执行 fsync() 函数；
- No 策略就是永不执行 fsync() 函数;

**always：**主线程在执行 fsync() 函数的时候，阻塞的时间会比较久，因为当写入的数据量很大的时候，数据同步到硬盘这个过程是很耗时的。

**Eversec:** 该模式是用异步任务来执行 fsync() 函数,所以不影响主线程的；

**No:**永不执行fsync(),何时写入由os来决定，不会阻塞主线程。

**总结：是否会对AOF产生影响，就是看主线程是否会主动调用fsync()。**

### II、对AOF重写和RDB的影响

#### **~fork()时页表过大带来阻塞阻塞**

当 AOF 日志写入了很多的大 Key，AOF 日志文件的大小会很大，那么很快就会触发 **AOF 重写机制**。

AOF 重写机制和 RDB 快照（bgsave 命令）的过程，**都会分别通过 `fork()` 函数创建一个子进程来处理任务**。

在fork()的过程中内核会把父进程的页表复制一份给子进程，**如果页表很大，那么这个复制过程是会很耗时的，那么在执行 fork 函数的时候就会发生阻塞现象**。fork()是由**redis主线程调用**的，如果主线程被阻塞，那么redis主线程无法接收客户端的请求，主线程被阻塞。



**我们可以执行 `info` 命令获取到 latest_fork_usec 指标，表示 Redis 最近一次 fork 操作耗时。**

```sql
# 最近一次 fork 操作耗时
latest_fork_usec:1330
```

如果 fork 耗时很大，比如超过1秒，则需要做出优化调整：

- **单个实例**的内存占用控制在 **10 GB 以下**，这样 fork 函数就能很快返回。
- 如果 Redis **只是当作纯缓存**使用，不关心 Redis 数据安全性问题，可以考虑**关闭 AOF 和 RDB 重写**，这样就不会调用 fork 函数了。
- 在主从架构中，要适当调大 repl-backlog-size，避免因为 repl_backlog_buffer 不够大，导致主节点频繁地使用全量同步的方式，全量同步的时候，是会创建 RDB 文件的，也就是会调用 fork 函数。

#### ~写时复制带来阻塞

如果创建完子进程后，**父进程对共享内存中的大 Key 进行了修改，那么内核就会发生写时复制，会把物理内存复制一份，由于大 Key 占用的物理内存是比较大的，那么在复制物理内存这一过程中，也是比较耗时的，于是父进程（主线程）就会发生阻塞**。

#### 总结：

- 创建子进程的途中，由于要复制父进程的页表等数据结构，阻塞的时间跟页表的大小有关，页表越大，阻塞的时间也越长；
- 创建完子进程后，如果子进程或者父进程修改了共享数据，就会发生写时复制，这期间会拷贝物理内存，如果内存越大，自然阻塞的时间也越长；

### III、其他影响

大 key 除了会影响持久化之外，还会有以下的影响：

- 客户端超时阻塞。由于 Redis 执行命令是单线程处理，然后在操作大 key 时会比较耗时，那么就会阻塞 Redis，从客户端这一视角看，就是很久很久都没有响应。
- 引发网络阻塞。每次获取大 key 产生的网络流量较大，如果一个 key 的大小是 1 MB，每秒访问量为 1000，那么每秒会产生 1000MB 的流量，这对于普通千兆网卡的服务器来说是灾难性的。
- 阻塞工作线程。如果使用 del 删除大 key 时，会阻塞工作线程，这样就没办法处理后续的命令。
- 内存分布不均。集群模型在 slot 分片均匀情况下，会出现数据和查询倾斜情况，部分有大 key 的 Redis 节点占用内存多，QPS 也会比较大。

**如何避免大 Key 呢？**

最好在设计阶段，就把**大 key 拆分成一个一个小 key**。或者，定时检查 Redis 是否存在大 key ，如果该大 key 是可以删除的，不要使用 DEL 命令删除，因为该命令删除过程会阻塞主线程，而是用 **unlink 命令**（Redis 4.0+）删除大 key，因为该命令的删除过程是异步的，不会阻塞主线程。

# 四、功能篇

## 1、过期删除策略

Redis 是可以对 key 设置过期时间的，因此需要有相应的机制将已过期的键值对删除，而做这个工作的就是过期**键值删除策略。**

### 如何设置过期时间？

先说一下对 key 设置过期时间的命令。 设置 key 过期时间的命令一共有 4 个：

- `expire <key> <n>`：设置 key 在 n 秒后过期，比如 expire key 100 表示设置 key 在 100 秒后过期；
- `pexpire <key> <n>`：设置 key 在 n 毫秒后过期，比如 pexpire key2 100000 表示设置 key2 在 100000 毫秒（100 秒）后过期。
- `expireat <key> <n>`：设置 key 在某个时间戳（精确到秒）之后过期，比如 expireat key3 1655654400 表示 key3 在时间戳 1655654400 后过期（精确到秒）；
- `pexpireat <key> <n>`：设置 key 在某个时间戳（精确到毫秒）之后过期，比如 pexpireat key4 1655654400000 表示 key4 在时间戳 1655654400000 后过期（精确到毫秒）

当然，在设置字符串时，也可以同时对 key 设置过期时间，共有 3 种命令：

- `set <key> <value> ex <n>` ：设置键值对的时候，同时指定过期时间（精确到秒）；
- `set <key> <value> px <n>` ：设置键值对的时候，同时指定过期时间（精确到毫秒）；
- `setex <key> <n> <valule>` ：设置键值对的时候，同时指定过期时间（精确到秒）。

如果你想查看某个 key 剩余的存活时间，可以使用 `TTL <key>` 命令。

```bash
# 设置键值对的时候，同时指定过期时间位 60 秒
> setex key1 60 value1
OK

# 查看 key1 过期时间还剩多少
> ttl key1
(integer) 56
> ttl key1
(integer) 52
```

如果突然反悔，取消 key 的过期时间，则可以使用 `PERSIST <key>` 命令。

```bash
# 取消 key1 的过期时间
> persist key1
(integer) 1

# 使用完 persist 命令之后，
# 查下 key1 的存活时间结果是 -1，表明 key1 永不过期 
> ttl key1 
(integer) -1
```

### 如何判定 key 已过期了？

**当我们对一个键值对设置了过期时间，那么Redis就会把该key带上过期时间存储到过期字典中**。过期字典保存了数据库中所有的key和其过期时间。

```c
typedef struct redisDb {
    dict *dict;    /* 数据库键空间，存放着所有的键值对 */
    dict *expires; /* 键的过期时间 */
    ....
} redisDb;
```

**过期字典的数据结构图如下：**

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E8%BF%87%E6%9C%9F%E7%AD%96%E7%95%A5/%E8%BF%87%E6%9C%9F%E5%AD%97%E5%85%B8%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png" alt="img" style="zoom: 80%;" />

当我们查询一个 key 时，Redis 首先检查该 key 是否存在于过期字典中：

- 如果不在，则正常读取键值；
- 如果存在，则会获取该 key 的过期时间，然后与当前**系统时间进行比对**，如果比系统时间大，那就没有过期，否则判定该 key 已过期。

### 过期删除策略有哪些？

常见的三种过期删除策略：

- **定时删除**: 在设置 key 的过期时间时，同时创建一个定时事件，当时间到达时，由事件处理器自动执行 key 的删除操作
  
  - 优点：
    - 内存友好，保证过期的key尽快被删除
  - 缺点：
    - 若在同时时间，过期的key比较多，删除过期的key会占去相当一部分CPU时间，会对服务器的响应时间和吞吐量造成影响。
- **惰性删除**: 不主动删除，当客户端访问该键的时候检查其过期时间，如果过期则删除
  
  - 优点：
    - 对CPU时间友好
  - 缺点：
    - 如果一个 key 已经过期，而这个 key 又仍然保留在数据库中，那么只要这个过期 key 一直没有被访问，它所占用的内存就不会释放，造成了一定的内存空间浪费。所以，惰性删除策略**对内存不友好。**
- **定期删除**：每隔一段时间「随机」从数据库中取出一定数量的 key 进行检查，并删除其中的过期key。
  
  - 优点：
    - 通过限制删除操作执行的时长和频率，来**减少删除操作对 CPU 的影响**，同时也能删除一部分过期的数据减**少了过期键对空间的无效占用。**
  - 缺点：
    - **由于删除时间是随机的**，难以确定删除操作执行的时长和频率。如果执行的太频繁，定期删除策略变得和定时删除策略一样，对CPU不友好；如果执行的太少，那又和惰性删除一样了，过期 key 占用的内存不会及时得到释放。
  
  因此，如果采用定期删除策略，服务器必须根据实际情况，合理地设置删除操作的**执行时长和执行频率**。

### Redis 过期删除策略是什么（惰性+定期）？

**Redis 选择「惰性删除+定期删除」这两种策略配和使用**，以求在合理使用 CPU 时间和避免内存浪费之间取得平衡。

#### ~Redis 是怎么实现惰性删除的？

**通过expireIfNeeded（）这个函数来实现惰性删除**

```c
int expireIfNeeded(redisDb *db, robj *key) {
    if (!keyIsExpired(db,key)) return 0;

if (server.masterhost != NULL) return 1;

if (checkClientPauseTimeoutAndReturnIfPaused()) return 1;

/* Delete the key */
server.stat_expiredkeys++;
propagateExpire(db,key,server.lazyfree_lazy_expire);
notifyKeyspaceEvent(NOTIFY_EXPIRED,
        "expired",key,db->id);
    int retval = server.lazyfree_lazy_expire ? dbAsyncDelete(db,key) ://可以在redis.conf文件中修改该参数
                                               dbSyncDelete(db,key);
    if (retval) signalModifiedKey(NULL,db,key);
    return retval;

```

**Redis 在访问或者修改 key 之前，都会调用 expireIfNeeded 函数对其进行检查**，检查 key 是否过期：

- 如果过期，则删除该 key，至于选择异步删除，还是选择同步删除，根据 **`lazyfree_lazy_expire`** 参数配置决定（Redis 4.0版本开始提供参数），然后返回 null 客户端；
- 如果没有过期，不做任何处理，然后返回正常的键值对给客户端；

#### ~Redis 是怎么实现定期删除的？

定期删除策略的做法：**每隔一段时间「随机」从一定数量的数据库中取出一定数量的 key 进行检查，并删除其中的过期key。**

**1、这个间隔检查的时间是多长呢？**

在 Redis 中，默认**每秒进行十次过期扫描**，此配置可通过 Redis 的配置文件 redis.conf 进行配置，配置键为 hz 它的默认值是 hz 10。

```shell
# By default "hz" is set to 10. Raising the value will use more CPU when
# Redis is idle, but at the same time will make Redis more responsive when
# there are many keys expiring at the same time, and timeouts may be
# handled with more precision.
#
# The range is between 1 and 500, however a value over 100 is usually not
# a good idea. Most users should use the default of 10 and raise this up to
# 100 only in environments where very low latency is required.
hz 10 	#建议10~100之间

```

**2、随机抽查的数量是多少呢？**

我查了下源码，定期删除的实现在 expire.c 文件下的 `activeExpireCycle` 函数中，其中随机抽查的数量由**ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP**定义的，它是写死在代码中的，数值是 20

也就是说，数据库每轮抽查时，**会随机选择 20 个 key 判断是否过期**。

**综上：在默认情况下，Redis每秒进行十次检查，每次检查从所有的key中随机选出20个，判断是否过期。**

## 2、内存淘汰策略（内存达到上限时触发）

当 Redis 的内存超过最大允许的内存之后，Redis 会触发内存淘汰策略。内存淘汰是指内存超过最大值时的保护策略，**内存达到上限时触发。**

### 如何设置 Redis 最大运行内存？

在配置文件 **redis.conf** 中，可以通过参数 `maxmemory <bytes>` 来设定最大运行内存，只有在 Redis 的运行内存达到了我们设置的最大运行内存，才会触发内存淘汰策略。 不同位数的操作系统，maxmemory 的默认值是不同的：

- 在 **64 位操作系统中**，**maxmemory 的默认值是 0**，表示没有内存大小限制，那么不管用户存放多少数据到 Redis 中，Redis 也不会对可用内存进行检查，直到 Redis 实例因内存不足而崩溃也无作为。
- 在 **32 位操作系统中**，**maxmemory 的默认值是 3G**，因为 32 位的机器最大只支持 4GB 的内存，而系统本身就需要一定的内存资源来支持运行，所以 32 位操作系统限制最大 3 GB 的可用内存是非常合理的，这样可以避免因为内存不足而导致 Redis 实例崩溃。

### Redis内存淘汰策略有哪些？

### *I.不进行数据淘汰的策略*

**noeviction**（Redis3.0之后，默认的内存淘汰策略） ：它表示当运行内存超过最大设置内存时，不淘汰任何数据，这时如果有新的数据写入，则会触发 OOM，但是如果没用数据写入的话，只是单纯的查询或者删除操作的话，还是可以正常工作。

```bash
# The default is:
#
# maxmemory-policy noeviction

127.0.0.1:6379> config get maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"

```

### *II.进行数据淘汰的策略*

针对「进行数据淘汰」这一类策略，又可以细分为**「在设置了过期时间的数据中进行淘汰」和「在所有数据范围内进行淘汰」**这两类策略。

在**设置了过期时间**的数据中进行淘汰：

- **volatile-random**：随机淘汰设置了过期时间的任意键值；
- **volatile-ttl**：优先淘汰更早过期的键值。
- **volatile-lru**（Redis3.0 之前，默认的内存淘汰策略）：淘汰所有设置了过期时间的键值中，最久未使用的键值；
- **volatile-lfu**（Redis 4.0 后新增的内存淘汰策略）：淘汰所有设置了过期时间的键值中，最少使用的键值；

在**所有数据**范围内进行淘汰：

- **allkeys-random**：随机淘汰任意键值;
- **allkeys-lru**：淘汰整个键值中最久未使用的键值；
- **allkeys-lfu**（Redis 4.0 后新增的内存淘汰策略）：淘汰整个键值中最少使用的键值。

> **如何修改 Redis 内存淘汰策略？**

设置内存淘汰策略有两种方法：

- 方式一：通过“`config set maxmemory-policy <策略>`”命令设置。它的优点是设置之后立即生效，不需要重启 Redis 服务，缺点是重启 Redis 之后，设置就会失效。
- 方式二：通过修改 Redis 配置文件修改，设置“`maxmemory-policy <策略>`”，它的优点是重启 Redis 服务后配置不会丢失，缺点是必须重启 Redis 服务，设置才能生效。

### LRU 算法和 LFU 算法有什么区别？

**LFU 内存淘汰算法是 Redis 4.0 之后新增内存淘汰策略**，那为什么要新增这个算法？那肯定是为了解决 LRU 算法的问题。

**传统的LRU算法：**

将数据用链表组织起来，按操作从前往后排列，最近被访问的数据会被移到链表头部，当需要内存淘汰的时候，只需将链表尾部的数据淘汰即可。

**Redis 并没有使用这样的方式实现 LRU 算法，因为传统的 LRU 算法存在两个问题**：

- 需要用链表管理所有的缓存数据，这会带来**额外的空间开销**；
- 当有数据被访问时，需要在链表上把该数据移动到头端，如果有大量数据被访问，就会带来**很多链表移动操作，会很耗时**，进而会降低 Redis 缓存性能。

#### Redis是如何实现LRU（按最近的访问时间来淘汰）的？

Redis 实现的是一种**近似 LRU 算法**，目的是为了更好的节约内存，它的**实现方式是在 Redis 的对象结构体中添加一个额外的字段，用于记录此数据的最后一次访问时间**。

当 Redis 进行内存淘汰时，会使用**随机采样的方式来淘汰数据**，它是随机取 5 个值（此值可配置），然后**淘汰最久没有使用的那个**。

**优点：节省了内存空间，提高了redis的缓存性能**

但是 LRU 算法有一个问题，**无法解决缓存污染问题**，比如应用一次读取了大量的数据，而这些数据只会被读取这一次，那么这些数据会留存在 Redis 缓存中很长一段时间，造成缓存污染。

因此，在 **Redis 4.0 之后引入了 LFU 算法来解决缓存污染问题。**

#### 什么是LFU算法（按照最近的访问次数来淘汰）

LFU 全称是 Least Frequently Used 翻译为**最近最不常用的，**LFU 算法是根据数据访问次数来淘汰数据的，它的核心思想是“如果数据过去被访问多次，那么将来被访问的频率也更高”。

所以， LFU 算法会记录每个数据的访问次数。当一个数据被再次访问时，就会增加该数据的访问次数。这样就解决了偶尔被访问一次之后，数据留存在缓存中很长一段时间的问题，相比于 LRU 算法也更合理一些。

#### Redis是如何实现LFU的？

LFU 算法相比于 LRU 算法的实现，多记录了**「数据的访问频次」**的信息。Redis 对象的结构如下：

```c
typedef struct redisObject {
    ...
    // 24 bits，用于记录对象的访问信息
    unsigned lru:24;  
    ...
} robj;
```

**在 LRU 算法中**，Redis 对象头的 **24 bits 的 lru 字段是用来记录 key 的访问时间戳**，因此在 LRU 模式下，Redis可以根据对象头中的 lru 字段记录的值，来比较最后一次 key 的访问时间长，从而淘汰最久未被使用的 key。

**在 LFU 算法中**，Redis对象头的 24 bits 的 lru 字段被分成两段来存储，**高 16bit 存储 ldt**(Last Decrement Time)，**低 8bit 存储 logc(**Logistic Counter)。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E8%BF%87%E6%9C%9F%E7%AD%96%E7%95%A5/lru%E5%AD%97%E6%AE%B5.png)

- ldt 是用来记录 key 的**访问时间戳**；
- logc 是用来记录 key 的**访问频次**，它的值越小表示使用频率越低，越容易淘汰，每个新加入的 key 的logc 初始值为 5。

**注意：logc是访问频次，并非访问次数，它会随着时间的增加而衰减**

在每次 key 被访问时，会先对 logc 做一个衰减操作，衰减的值跟前后访问时间的差距有关系，如果前后两次访问时间间隔较长，衰减的值就大，这样实现的 LFU 算法是根据**访问频率**来淘汰数据的，而不只是访问次数。

对 logc 做完衰减操作后，就开始对 logc 进行增加操作，增加操作并不是单纯的 + 1，而是根据概率增加， **logc 越大的 key，它的 logc 就越难再增加。**

redis.conf 提供了两个配置项，用于调整 LFU 算法从而控制 logc 的增长和衰减：

- `lfu-decay-time` 用于调整 logc 的衰减速度，它是一个以分钟为单位的数值，默认值为1，lfu-decay-time 值越大，衰减越慢；
- `lfu-log-factor` 用于调整 logc 的增长速度，lfu-log-factor 值越大，logc 增长越慢。

```bash
# The default value for the lfu-decay-time is 1. A special value of 0 means to
# decay the counter every time it happens to be scanned.
#
# lfu-log-factor 10
# lfu-decay-time 1
```

## 3、内存回收

Redis在自己的对象系统中构建了一个**引用计数**技术实现内存的回收机制，通过这一机制，程序可以通过对象的引用计数信息，在适当的时候自动释放对象并进行内存回收。

```c
typydef struct redisObject {
    //……
    int refcount;
    //……
} robj;
```

- 当创建一个新对象时，引用计数为1
- 当对象被一个程序使用时，引用计数值加1
- 当对象不再被对象使用时，引用计数减1
- 当计数变为0时，对象所占的内存将会被释放

# 五、高可用

## 1、主从复制

### 主从复制解决什么问题？

前面我们说了AOF和RDB两种日志来实现了缓存数据的持久化，可用于Redis服务挂掉或系统宕机时恢复数据

但是，在服务宕机的情况下，该台机器是无法处理请求的，这回影响用户的请求访问，并且如果这台服务器的硬盘出现了故障，那么这台机器上已经持久化了的数据就无法恢复，这是相当恐怖的。

所以，要避免这种单点故障，最好的办法是将数据备份到其他服务器上，让这些服务器也可以对外提供服务，这样即使有一台服务器出现了故障，其他服务器依然可以继续提供服务。

这些服务器之间的数据如何保持一致性呢？数据的读写操作是否每台服务器都可以处理？

Redis 提供了**主从复制模式**，来避免上述的问题。**一句话，主从复制解决的是多机部署下的数据一致性问题。**

**主从复制模式：**

这个模式可以保证多台服务器的数据一致性，且主从服务器之间采用的是**「读写分离」的方式**。

**主服务器可以进行读写操作**，当发生**写操作时自动将写操作同步给从服务器**，而**从服务器一般是只读**，并接受主服务器同步过来写操作命令，然后执行这条命令。

<img src="https://img-blog.csdnimg.cn/img_convert/2b7231b6aabb9a9a2e2390ab3a280b2d.png" alt="图片" style="zoom:67%;" />

### 第一次同步

多台服务器之间要通过什么方式来确定谁是主服务器，或者谁是从服务器呢？

我们可以使用 `replicaof`（Redis 5.0 之前使用 slaveof）命令**形成主服务器和从服务器的关系**。

比如，现在有服务器 A 和 服务器 B，我们在服务器 B 上执行下面这条命令：

```text
# 服务器 B 执行这条命令
replicaof <服务器 A 的 IP 地址> <服务器 A 的 Redis 端口号>
```

接着，**服务器 B 就会变成服务器 A 的「从服务器」**，然后与主服务器进行第一次同步。

主从服务器间的第一次同步的三个阶段：

- 第一阶段是建立链接、协商同步；
- 第二阶段是主服务器同步数据给从服务器；
- 第三阶段是主服务器发送新写操作命令给从服务器。

<img src="https://img-blog.csdnimg.cn/img_convert/ea4f7e86baf2435af3999e5cd38b6a26.png" alt="图片" style="zoom:67%;" />

**第二阶段：**

协商完成之后，主服务器会执行 bgsave 命令来**生成 RDB 文件**，然后把文件发送给从服务器。从服务器收到 RDB 文件后，会先清空当前的数据，然后载入 RDB 文件。由于主服务器生成RDB文件的过程是非阻塞的，这是由主服务器的主进程产生的子进程来完成的，所以在RDB复制过程中新增或修改的数据并没有同步到从服务器，**这就造成了数据的不一致**

那么为了保证主从服务器的数据一致性，主服务器在下面这三个时间间隙中将收到的写操作命令，**写入到 replication buffer 缓冲区里，replication buffer`（复制缓冲区）`写入的时机为：**

- 主服务器生成 RDB 文件期间；
- 主服务器发送 RDB 文件给从服务器期间；
- 「从服务器」加载 RDB 文件期间；

​										**（主节点会给每个新连接的从节点，分配一个 replication buffer）**

**第三阶段**：

**完成 RDB 的载入**后，会回复一个确认消息给主服务器。

接着，主服务器将 replication buffer 缓冲区里所记录的写操作命令发送给从服务器，从服务器执**行来自主服务器 replication buffer 缓冲区里发来的命令，这时主从服务器的数据就一致了。**

### 命令传播

主从服务器在完成第一次同步后，双方之间就会维护一个 TCP 连接。

后续主服务器可以通过这个连接继续将**写操作命令传播给从服务器**，然后从服务器执行该命令，使得与主服务器的数据库状态相同。

而且这个连接是长连接的，目的是**避免频繁的 TCP 连接和断开带来的性能开销。**

上面的这个过程被称为**基于长连接的命令传播**，通过这种方式来保证第一次同步后的主从服务器的数据一致性

### 分摊主服务器的压力

有两个问题会影响主服务器的性能：

- 在进行RDB复制的时候会fork()子进程,如果内存数据非常大并且频繁的fork()，那么主线程被阻塞，无法处理客户端请求。
- 在进行RDB传输的时候，主线程会占用网络带宽，这会对客户端与服务器之间的连接与数据传输带来影响。

为了分担主服务器压力，采用下面这种模型来

<img src="https://img-blog.csdnimg.cn/img_convert/4d850bfe8d712d3d67ff13e59b919452.png" alt="图片" style="zoom:67%;" />

**主服务器生成 RDB 和传输 RDB 的压力可以分摊到充当经理角色的从服务器**。

那具体怎么做到的呢？

其实很简单，我们在「从服务器」上执行下面这条命令，使其作为目标服务器的从服务器：

```text
replicaof <目标服务器的IP> 6379
```

此时如果**目标服务器本身也是「从服务器」**，那么该目标服务器就会成为「经理」的角色，不仅可以接受主服务器同步的数据，也会把数据同步给自己旗下的从服务器，从而减轻主服务器的负担。

### 增量复制（主从网络断开恢复时使用）

在 Redis 2.8 之前，如果主从服务器在命令同步时出现了网络断开又恢复的情况，从服务器就会和主服务器**重新进行一次全量复制**，很明显这样的开销太大了。

所以，从 Redis 2.8 开始，网络断开又恢复后，从主从服务器会采用**增量复制**的方式继续同步，也就是只会把网络断开期间主服务器接收到的写操作命令，同步给从服务器。

那么关键的问题来了，**主服务器怎么知道要将哪些增量数据发送给从服务器呢？**

答案藏在这两个东西里：

- **repl_backlog_buffer**，是一个「**环形**」缓冲区，用于主从服务器断连后，从中找到差异的数据；
- **replication offset**，标记上面那个缓冲区的同步进度，**主从服务器都维护一个复制偏移量**，主服务器使用 master_repl_offset 来记录自己「*写*」到的位置，从服务器使用 slave_repl_offset 来记录自己「*读*」到的位置。**通过判断这两个偏移量是否相等程序就很容易知道主从服务器是否处于一致状态**。如果偏移量不同，则说明中途连接短暂断开过，主从不一致，这时当从服务器重新连接以后就可以通过判断自己的offset是否仍然在缓冲区内，如果仍在缓冲区内，就可通过二者偏移量之差计算出需要增量复制的数据，否则就要进行全量同步。

**当主服务器进行命令传播时，不仅会将写命令发送给从服务器，还会将写命令写入到 repl_backlog_buffer 缓冲区里，因此 这个缓冲区里会保存着最近传播的写命令。**

<img src="https://img-blog.csdnimg.cn/img_convert/2db4831516b9a8b79f833cf0593c1f12.png" alt="图片" style="zoom:67%;" />

因此，**为了避免在网络恢复时，主服务器频繁地使用全量同步的方式，我们应该调整下 repl_backlog_buffer 缓冲区大小（从redis.conf中调整），尽可能的大一些**，减少出现从服务器要读取的数据被覆盖的概率，从而使得主服务器采用增量同步的方式。

**repl_backlog_buffer 最小的大小可以根据这面这个公式估算：**

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221015223318052.png" alt="image-20221015223318052" style="zoom:67%;" />

- second 为从服务器断线后重新连接上主服务器所需的平均时间(以秒计算)。

- write_size_per_second 则是主服务器平均每秒产生的写命令数据量大小。

### 总结

主从复制共有三种阶段：**全量复制、基于长连接的命令传播、增量复制**。

主从服务器**第一次同步**的时候，会采用全量复制的方式进行同步，此时主服务器会有两个耗时的时间，一个是生成RDB文件和传输RDB文件，为了避免过多的数据从主服务器向多台从服务器发送，可以把一部分从服务器升级为经理角色（从服务器再带几个从服务器）以分担主服务器压力。

第一次同步之后，主从服务器之间就会维护着一个长连接，每当主服务器有写入操作的时候，就会通过这个连接将命令传播给从服务器，从而保证了数据的一致性。

如果遇到网络断开，增量复制就可以上场了，不过这个还跟 repl_backlog_size 这个大小有关系。如果过小，可能导致未同步的数据被新数据覆盖，导致数据丢失，此时主从服务器之间又会开启全量复制，加大了系统开销。所以要调大这个参数的值，以降低主从服务器断开后全量同步的概率。

## 2、哨兵

### 为什么要有哨兵机制？

在 Redis 的主从架构中，由于主从模式是读写分离的，如果**主节点（master）挂了**，那么将没有主节点来服务客户端的写操作请求，也**没有主节点给从节点（slave）进行数据同步了**。如果这是要恢复的话，就需要人工介入重新设置主节点，但这种方法不够智能，于是Redis 在 2.8 版本以后提供的**哨兵（Sentinel）机制**，它的作用是实现**主从节点故障转移**，实现了 Redis 的主从架构下的高可用。

### 哨兵是如何工作的？

哨兵其实是一个**运行在特殊模式下的 Redis 进程**，所以它也是一个节点。观察的对象是**主从节点**。

当然，它不仅仅是观察那么简单，在它观察到有异常的状况下，会做出一些“动作”，来**修复异常状态**。

哨兵节点主要负责三件事情：**监控、选主、通知**。

### 如何判断主节点出现故障？

哨兵会每隔 1 秒给所有主从节点发送 PING 命令，当主从节点收到 PING 命令后，会发送一个响应命令给哨兵，这样就可以判断它们是否在正常运行。

<img src="https://img-blog.csdnimg.cn/26f88373d8454682b9e0c1d4fd1611b4.png" alt="哨兵监控主从节点" style="zoom:67%;" />

如果主节点或者从节点没有在规定的时间内响应哨兵的 PING 命令，哨兵就会将它们标记为「**主观下线**」。这个「规定的时间」是配置项 `down-after-milliseconds` 参数设定的，单位是毫秒。

#### 主观下线与客观下线

**主观下线：**

之所以针对「主节点」设计「主观下线」和「客观下线」两个状态，是因为有可能「主节点」其实并没有故障，可能只是因为主节点的系统压力比较大或者网络发送了拥塞，导致主节点没有在规定时间内响应哨兵的 PING 命令。一句话：**主观下线并非真正下线，可能只是网络拥塞带来的响应延迟，因此主观下线是会误判的。**

**客观下线：**

**客观下线只适用于主节点。**为了减少误判的情况，哨兵在部署的时候不会只部署一个节点，而是用多个节点部署成**哨兵集群**（*最少需要三台机器来部署哨兵集群*），**通过多个哨兵节点一起判断，就可以就可以避免单个哨兵因为自身网络状况不好，而误判主节点下线的情况**。

#### 如何判断客观下线？

当一个哨兵判断主节点为「主观下线」后，就会向其他哨兵发起命令，其他哨兵收到这个命令后，就会根据自身和主节点的网络状况，做出赞成投票或者拒绝投票的响应。

<img src="https://img-blog.csdnimg.cn/13e4361407ba46979e802eaa654dcf67.png" alt="img" style="zoom:67%;" />

当这个哨兵的赞同票数达到哨兵配置文件中的 **quorum 配置项**设定的值后，这时主节点就会被该哨兵标记为「客观下线」。

quorum 的值一般设置为哨兵个数的二分之一加1，例如 3 个哨兵就设置 2。

哨兵判断完主节点客观下线后，哨兵就要开始在多个「从节点」中，选出一个从节点来做新主节点。

### 由哪个哨兵进行主从故障转移（哨兵中的leader）？

前面说过，为了更加“客观”的判断主节点故障了，一般不会只由单个哨兵的检测结果来判断，而是多个哨兵一起判断，这样可以减少误判概率，所以**哨兵是以哨兵集群的方式存在的**。

所以这时候，**如果主服务器被判断为客观下线的时候**，就需要在哨兵集群中选出一个 leader，**让 leader 来执行主从切换，实现故障转移。**

选举 leader 的过程其实是一个投票的过程，在投票开始前，肯定得有个**「候选者」**。

**哪个哨兵节点判断主节点为「客观下线」（在判断客观下线时发起协商）**，这个哨兵节点就是**候选者**，所谓的候选者就是想当 Leader 的哨兵。**（只是候选人，还没有真正成为leader）**

#### 如何由候选人变为leader？

候选者会向其他哨兵发送命令，表明希望成为 Leader 来执行主从切换，并让所有其他哨兵对它进行投票。

每个哨兵只有一次投票机会，如果用完后就不能参与投票了，可以投给自己或投给别人，但是**只有候选者才能把票投给自己**。

任何一个「候选者」，要满足两个条件：

- **第一，拿到半数以上的赞成票；**
- **第二，拿到的票数同时还需要大于等于哨兵配置文件中的 quorum 值。**

### 故障转移过程

#### Step1：选出新主节点

故障转移操作第一步要做的就是在已下线主节点属下的所有「从节点」中，挑选出一个状态良好、数据完整的从节点，然后向这个「从节点」发送 **SLAVEOF no one** 命令，将这个**「从节点」转换为「主节点」**。

选取节点**不能是随机的**，我们首先必须将网络状况不好的从节点先过滤掉，Redis 有个叫 **down-after-milliseconds * 10** 配置项，如果发生断连的次数超过了 10 次，就说明这个从节点的网络状况不好，不适合作为新主节点。

在过滤掉网络不好的从节点后，接下来要对所有从节点进行三轮考察：**优先级、复制进度、ID 号**。在进行每一轮考察的时候，哪个从节点优先胜出，就选择其作为新主节点。

- 第一轮考察：哨兵首先会根据从节点的优先级来进行排序，优先级越小排名越靠前**（slave-priority 配置项）**
- 第二轮考察：如果优先级相同，则查看复制的下标，哪个从「主节点」接收的复制数据多（复制进度最靠前），哪个就靠前。
- 第三轮考察：如果优先级和下标都相同，就选择从节点 ID 较小的那个。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%93%A8%E5%85%B5/%E9%80%89%E4%B8%BB%E8%BF%87%E7%A8%8B.webp" alt="img" style="zoom:67%;" />

选举出从节点后，哨兵 leader 向被选中的从节点发送 `SLAVEOF no one` 命令，让这个从节点解除从节点的身份，将其变为新主节点。哨兵 leader 向被选中的从节点 发送 `SLAVEOF no one` 命令，将该从节点升级为新主节点。

在发送 `SLAVEOF no one` 命令之后，哨兵 leader 会以每秒一次的频率向被升级的从节点发送 `INFO` 命令（没进行故障转移之前，`INFO` 命令的频率是每十秒一次）,观察恢复信息就可判断它的晋升之路是否顺利。

#### step2:让其余从节点指向新的主节点

当新主节点出现之后，哨兵 leader 下一步要做的就是，让已下线主节点属下的所有「从节点」指向「新主节点」，这一动作可以通过向「从节点」发送 `SLAVEOF` 命令来实现。

#### step3:通知客户端主节点已更换

这主要**通过 Redis 的发布者/订阅者机制来实现**的。每个哨兵节点提供发布者/订阅者机制，客户端可以从哨兵订阅消息。

哨兵提供的消息订阅频道有很多，不同频道包含了主从节点切换过程中的不同关键事件，几个常见的事件如下：

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%93%A8%E5%85%B5/%E5%93%A8%E5%85%B5%E9%A2%91%E9%81%93.webp" alt="img" style="zoom: 23%;" />

**主从切换完成后，哨兵就会向 `+switch-master` 频道发布新主节点的 IP 地址和端口的消息，这个时候客户端就可以收到这条信息，然后用这里面的新主节点的 IP 地址和端口进行通信了**。

**即客户端收到通知后，之后写数据就通过该新主节点进行。**

#### Step4:将旧主节点变为从节点

故障转移操作最后要做的是，继续监视旧主节点，当旧主节点重新上线时，哨兵集群就会向它发送 `SLAVEOF` 命令，让它成为新主节点的从节点

### 哨兵集群如何组成？

**哨兵节点之间是通过 Redis 的发布者/订阅者机制来相互发现的**。

在主从集群中，**主节点**上有一个名为`__sentinel__:hello`的频道，不同哨兵就是通过它来相互发现，实现互相通信的。

哨兵 A 把自己的 IP 地址和端口的信息发布到`__sentinel__:hello` 频道上，哨兵 B 和 C 订阅了该频道。那么此时，哨兵 B 和 C 就可以从这个频道直接获取哨兵 A 的 IP 地址和端口号。然后，哨兵 B、C 可以和哨兵 A 建立网络连接。

<img src="https://img-blog.csdnimg.cn/a6286053c6884cf58bf397d01674fe80.png" alt="img" style="zoom:50%;" />

### 哨兵集群如何知道「从节点」的信息？

**主节点知道所有「从节点」的信息**，所以哨兵会每 10 秒一次的频率向主节点发送 **INFO 命令**来获取所有「从节点」的信息。

正式通过 Redis 的发布者/订阅者机制，哨兵之间可以相互感知，然后组成集群，同时，哨兵又通过 INFO 命令，在主节点里获得了所有从节点连接信息，于是就能和从节点建立连接，并进行监控了。

<img src="https://img-blog.csdnimg.cn/fdd5f695bb3643258662886f9fba0aab.png" alt="img" style="zoom50%;" />

## 3、集群

### Redis有几种集群模式？

三种。分别是：主从模式、哨兵模式、cluster集群模式。

#### **I.主从模式：**

至少需要两台redis服务器，一台主节点（master）、一台从节点（slave），组成主从模式的Redis集群。通常来说，master主要负责写，slave主要负责读，主从模式实现了读写分离。

**主从复制的作用：**

- 数据冗余：实现了数据的热备份，是持久化之外的另一种数据冗余方式
- 故障恢复：master故障时，slave可以提供服务，实现故障快速恢复
- 负载均衡：master负责写，slave负责读。在写少读多的场景下可以极大提高redis吞吐量
- 高可用基石：**主从复制是redis哨兵模式和cluster集群模式的基础**。

**不足之处：**

主节点宕机之后，需要**手动拉起从节点**来提供业务，不能达到高可用。

#### II.哨兵模式：

哨兵模式实现了redis的高可用。但是有一个问题，这两种模式都没有解决，这两种模式都**只能有一个master节点负责写操作**，在**高并发的写**操作场景，master节点就会成为性能瓶颈。

Redis较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。

#### III、Cluster集群模式（官方推荐）：

Redis集群是Redis提供的分布式数据库方案，**集群通过分片（sharding）**来进行数据共享，并提供复制和故障转移功能。

Redis 的**哨兵模式**基本已经可以实现高可用，读写分离 ，但是在这种模式下**每台 Redis 服务器都存储相同的数据，很浪费内存**，所以在redis3.0上加入了 Cluster 集群模式，实现了 **Redis 的分布式存储**，也就是说**每台 Redis 节点上存储不同的内容**。

<img src="https://img-blog.csdnimg.cn/dc91a19c03d14267ab9203fa337f25da.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAc3VuX2xt,size_20,color_FFFFFF,t_70,g_se,x_16" alt="img" style="zoom:70%;" />

### Cluster集群模式详解：

#### 什么是节点？

一个Redis集群有多个节点，每个节点在一台独立的机器上，在节点之间握手通信之前，各个节点是相互独立的，当他们通过命令进行相互连接后，他们就成为一个集群。

可以根据修改 **cluster-enabled配置项**来决定是否开启集群模式（默认为no）

连接各个节点的工作可以使用**CLUSTER MEET命令**来完成， 格式如下：

**CLUSTER MEET　＜ip＞ <port\>**

#### 节点间的内部通信机制？

Redis 维护集群元数据采用另一个方式， **gossip 协议**，所有节点都持有一份元数据，**不同的节点如果出现了元数据的变更，就不断将元数据发送给其它的节点，让其它节点也进行元数据的变更。**

gossip 好处在于，元数据的更新比较分散，不是集中在一个地方，更新请求会陆陆续续打到所有节点上去更新，降低了压力；不好在于，元数据的更新有延时，可能导致集群中的一些操作会有一些滞后

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221124172314448.png" alt="image-20221124172314448" style="zoom:50%;" />

**gossip 协议包含多种消息，包含 ping , pong , meet , fail 等等。**

- meet：某个节点发送 meet 给新加入的节点，让新节点加入集群中，然后新节点就会开始与其它节点进行通信。
- bash Redis-trib.rb add-node其实内部就是发送了一个 gossip meet 消息给新加入的节点，通知那个节点去加入我们的集
- 群。
- ping：每个节点都会频繁给其它节点发送 ping，其中包含自己的状态还有自己维护的集群元数据，互相通过 ping 交换元数据。
- pong：返回 ping 和 meeet，包含自己的状态和其它信息，也用于信息广播和更新。
- fail：某个节点判断另一个节点 fail 之后，就发送 fail 给其它节点，通知其它节点说，某个节
- 点宕机啦

#### **CLUSTER MEET命令实现步骤**

1. 当节点A收到CLUSTER MEET命令以后，A会为B创建一个数据结构（clusterNode）,并将其保存在自己的clusterState.nodes字典里。
2. 根据IP 和 port，A向B发送一条MEET消息
3. B收到MEET消息以后，B会为A创建一个数据结构（clusterNode）,并将其保存在自己的clusterState.nodes字典里。
4. B向A发送一条PONG消息
5. 如果A顺利收到PONG消息，返回一条PING消息
6. B收到PING消息以后，说明握手已经完成，连接建立

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221016150334549.png" alt="image-20221016150334549" style="zoom: 67%;" />

#### Redis集群分片

```c
struct clusterNode {
    //……
    unsigned char slots[16348 / 8];
    int numslots;
    //……
}
```

slots属性是一个二进制位数组，这个数组长度为`16348  / 8 = 2048`个字节，包含16348个二进制位，Redis以0为起始索引，以16383为终止索引 ，**并根据索引值来判断该节点能否处理该槽：**

- 如果在索引i上的二进制值为1，则能够处理该槽
- 如果在索引i上的二进制值为0，则不能够处理该槽

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221113205614754.png" alt="image-20221113205614754" style="zoom:47%;" />

Redis集群通过分片的方式来保存数据库中的键值对：集群被分为**16384个槽**,数据库中每个键都属与16384个槽中的一个，集群中每个节点可以处理0个或最多16384个槽。

:interrobang: **注意：槽指的是二进制位，一个slot有8个槽，一个slot占一个字节。**

可以通过向节点发送**CLUSTER ADDSLOTS <slot\> [slot...]命令**来为节点指派一个或多个槽 

当数据库中的节点**已经建立连接但还未指定槽，此时集群处于下线状态**，只有当数据库中16384个槽都指派给了相应节点，集群进入上线状态。

<img src="https://upload-images.jianshu.io/upload_images/15462057-05bac9515bad3cd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" alt="高级开发不得不懂的Redis Cluster数据分片机制" style="zoom:80%;" />

#### Cluster 模式的原理？

其实现**原理就是一致性 Hash**。Redis Cluster 中有一个 16384 长度的槽的概念，他们的编号为 0、1、2、3 …… 16382、16383。这个槽是一个虚拟的槽，并不是真正存在的。正常工作的时候，Redis Cluster 中的每个 Master 节点都会负责一部分的槽，当有某个 key 被映射到某个 Master 负责的槽，那么这个 Master 负责为这个 key 提供服务。

至于**哪个 Master 节点负责哪个槽，这是可以由用户指定的，也可以在初始化的时候自动生成（redis-trib.rb脚本）**。这里值得一提的是，在 Redis Cluster 中，**只有 Master 才拥有槽的所有权**，如果是某个 Master 的 slave，这个**slave只负责槽的使用**，但是没有所有权。


#### 什么是一致性 Hash 以及解决什么问题？

在普通的Hash算法中，如果如果要增加一台机器，或者节点宕机的情况下，在数据迁移之前，对key取模计算后，判断出来key所在slot的位置会与原来不符，**会造成缓存原有大量缓存失效**。

<img src="https://images.cnblogs.com/cnblogs_com/barantt/1957548/o_210406084846hash2.png" alt="一致性hash1" style="zoom:50%;" />

一致性 hash 算法将**整个 hash 值空间组织成一个虚拟的圆环，整个空间按顺时针方向组织**，下一步将各个 master 节点（使用服务器的 **ip 或主机名**）进行 hash。这样就能确定每个节点在其哈希环上的位置。

一致性 hash 其实是普通 hash 算法的改良版，其 hash 计算方法没有变化，但是 **hash 空间发生了变化**，由原来的线性的变成了环。

缓存 key 通过 hash 计算之后得到**在 hash 环中的位置**，然后**顺时针方向找到第一个节点**，这个节点就是存放 key 的节点。

<img src="https://img-blog.csdnimg.cn/img_convert/82caa780a7496c9c698409cb802deaa6.png" alt="img" style="zoom:50%;" />

由此可见，**一致性 hash 主要是为了解决普通 hash 中扩容和宕机的问题。**因为在**Hash环中的数据迁移只发生在两个节点之间，普通Hash中数据的迁移会涉及所有节点**

<img src="https://images.cnblogs.com/cnblogs_com/barantt/1957548/o_210406112557WechatIMG34.png" alt="img" style="zoom:45%;" />

**但是如果节点数量过少，并且偏向一侧**，很大一部分的key都会落在node0上，从而导致node0的压力过大挂掉，node0挂掉之后数据同时又会向node1上转移，node1又会挂掉，最终导致整个集群不可用！这就是**数据倾斜**的问题，会引起节点雪崩。

同时还可以通过**虚拟节点来解决数据倾斜的问题**：就是在节点稀疏的 hash 环上对物理节点虚拟出一部分虚拟节点，key 会打到虚拟节点上面，而虚拟节点上的 key 实际也是映射到物理节点上的，这样就**避免了数据倾斜导致单节点压力过大导致节点雪崩的问题。**

<img src="https://images.cnblogs.com/cnblogs_com/barantt/1957548/o_210406115754WechatIMG35.png" alt="img" style="zoom:50%;" />

#### Redis集群模式下，怎么计算哪个key属于哪个slot？

节点使用CRC16算法（冗余连接校验和）来计算key在哪个槽中：

```python
def slot_number(key):
    return CRC16(16) & 16384;
```

对于 `CRC16` 算法产生的 hash 值会有 16bit，可以产生 2^16 = 65536 个值。

Redis 集群提供了灵活的节点**扩容和收缩**方案。在不影响集群对外服务的情况下，可以为集群添加节点进行扩容也可以下线部分节点进行缩容。可以说，槽是 Redis 集群管理数据的基本单位，**集群伸缩就是槽和数据在节点之间的移动**。

#### Cluster集群扩容流程？

<img src="https://upload-images.jianshu.io/upload_images/15462057-72bf4f641aac154b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" alt="高级开发不得不懂的Redis Cluster数据分片机制" style="zoom: 67%;" />

1）首先启动一个 Redis 节点，记为 M4。

2）使用 cluster meet 命令，让新 Redis 节点加入到集群中。新节点刚开始都是主节点状态，由于没有负责的槽，所以不能接受任何读写操作，后续给他迁移槽和填充数据。

3）对 M4 节点发送 cluster setslot { slot } importing { **sourceNodeId** } 命令，让**目标节点(M4)准备导入槽的数据。**

4）对源节点，也就是 M1，M2，M3 节点发送 cluster setslot { slot } migrating { targetNodeId } 命令，让**源节点（M1、M2、M3）准备迁出槽**的数据。**(注意：只是准备数据)**

5）源节点执行 cluster getkeysinslot { slot } { count } 命令，获取 count 个属于槽 { slot } 的键，然后执行步骤 6）的操作进行迁移键值数据。

6）在源节点上执行 migrate { **targetNodeIp**} " " 0 { timeout } keys { key... } 命令，把获取的键通过 pipeline 机制批量迁移到目标节点，批量迁移版本的 migrate 命令在 Redis 3.0.6 以上版本提供。

7）重复执行步骤 5）和步骤 6）直到槽下**所有的键值数据迁移到目标节点**。

8）向集群内**所有主节点发送** cluster setslot { slot } node { targetNodeId } 命令，通知槽分配给目标节点。为了保证槽节点映射变更及时传播，需要遍历发送给所有主节点更新被迁移的槽执行新节点。

#### Cluster 集群收缩流程？

收缩节点就是将 Redis 节点下线，整个流程需要如下操作流程。

1）首先需要确认下线节点是否有负责的槽，如果是，需要把槽迁移到其他节点，保证节点下线后整个集群槽节点映射的完整性。

2）当下线节点不再负责槽或者本身是从节点时，就可以通知集群内其他节点忘记下线节点，当所有的节点忘记改节点后可以正常关闭。

#### 客户端如何路由？

当客户端查询一个键值对时，会通过一致性Hash算法来计算出哪个key在哪个slot，我们如何知道该key的值存储在哪个机器节点上呢？即我们需要一个查询路由，**该路由根据给定的key，返回存储该键值的机器地址。**

Redis的**每个节点**中都存储着如下所示的**整个集群**的状态，集群状态中一个**重要的信息就是每个桶的负责节点**。

```c
typedef struct clusterNode {
    ...
    unsigned char slots[CLUSTER_SLOTS/8];
    ...
} clusterNode;

typedef struct clusterState {
   
    // slots记录每个桶被哪个节点存储
    clusterNode *slots[CLUSTER_SLOTS];
    uint64_t slots_keys_count[CLUSTER_SLOTS];
    rax *slots_to_keys;
    ....
} clusterState;
```

Redis 接收任何键相关命令时首先计算键对应的桶编号，如果节点是自身，则处理键命令,如果不是，回复 **`MOVED` 重定向错误**，通知客户端请求正确的节点，这个过程称为 **MOVED 重定向**。**重定向信息**包含了**键所对应的桶以及负责该桶的节点地址**，根据这些信息客户端就可以向正确的节点发起请求。

#### 为什么是 163834 个槽位？

**从消息的大小考虑：**

crc16能得到65535个值，不同的余数，代表bitmap 有 65535 bit。所以bitmap的大小可以计算为

```text
65535 / 8 (8bit/byte)/1024(1k)=7.99 Kbytes
```

尽管crc16能得到65535个值，但redis选择16384个slot，是因为**16384的消息只占用了2k**，而**65535则需要8k**，这极大浪费了网络带宽

**集群规模设计考虑：**

集群节点越多，心跳包的消息体内携带的数据越多。**如果节点过1000个，也会导致网络拥堵**。Redis Cluster 不太可能扩展到超过 1000 个主节点。集群设计最多支持1000个分片，16384是相对比较好的选择，需要保证在最大集群规模下，slot均匀分布场景下，每个**分片平均分到的slot不至于太小**。



#### 集群的故障检测与转移？

**1.故障检测：**

集群中每个节点会定期向集群中其他节点发送PING消息来检测对方是否在线，如果对方**没有在限定时间内恢复PONG消息**，那么该节点就会被标记为**疑似（主观）下线（probable fail）**

集群中各个节点会**互发消息来交换集群中各个节点中的状态信息**，如果在一个集群里面，**半数以上**负责处理槽的主节点将某个节点x标记为主观下线，那么这个主节点x就被标记为客观下线（Fail）,将x标记为客观下线的节点会向集群**广播**一条关于x下线的消息，之后集群中的所有节点都会把该节点标记未主观下线。

**2.故障转移**

- 当一个已下线的从节点发现自己正在复制的主节点处于已下线状态时，从节点就会对下线主节点进行故障转移：
- 复制该下线主节点的从节点中会有一个被选中成为新的主节点**（该主节点由投票产生，投票者是集群中别的master，该投票算法基于Raft）**。
- 被选中的节点会执行 SALAVEOF no one命令，成为新的主节点。
- 新的主节点会撤销原主节点的slots,并将这些slots指派给自己。
- 新的主节点向集群广播一条PONG消息，让集群中其他主节点知道自己以接管原主节点处理请求

# 六、事务篇

## 事务的开始到结束分的三个阶段：

**1)事务开始：**

```bash
127.0.0.1:6379> multi 
OK
```

**2)命令入队：**

```bash
127.0.0.1:6379(TX)> set name victorG
QUEUED
127.0.0.1:6379(TX)> get name
QUEUED
127.0.0.1:6379(TX)> set hobby "BodyBuilding"
QUEUED
127.0.0.1:6379(TX)> get hobby
QUEUED

```

**3)执行事务：**

```bash
127.0.0.1:6379(TX)> exec
1) OK
2) "victorG"
3) OK
4) "BodyBuilding"
```

## Redis事务是否支持回滚？

**Redis 中并没有提供回滚机制**，虽然 Redis 提供了 DISCARD 命令，但是这个命令只能用来主动放弃事务执行，把暂存的命令队列清空，**起不到回滚的效果。**

下面是 **DISCARD** 命令用法：

```c
#读取 count 的值4
127.0.0.1:6379> GET count
"1"
#开启事务
127.0.0.1:6379> MULTI 
OK
#发送事务的第一个操作，对count减1
127.0.0.1:6379> DECR count
QUEUED
#执行DISCARD命令，主动放弃事务
127.0.0.1:6379> DISCARD
OK
#再次读取a:stock的值，值没有被修改
127.0.0.1:6379> GET count
"1"
```

### 原子性问题：

事务执行过程中，如果命令入队时没报错，而事务提交后，实际执行时报错了，**正确的命令依然可以正常执行**，所以这可以看出 **Redis 并不一定保证原子性**（原子性：事务中的命令要不全部成功，要不全部失败）。

### 一致性问题：

无论redis服务器运行在那种状态下，事务执行中途发生的停机都不会影响数据库数据的一致性。

### 隔离性问题：

隔离性是指，即使多个事务并发地执行，各个事务之间也不会互相影响，并且并发状态下执行的事务与串行执行的事务产生的结果完全相同。因为Redis使用的是单线程方式执行事务，并且服务器保证，在执行事务期间不会对事务进行中断，因此**Redis事务总是串行执行**，保证隔离性。

### 持久性问题：

Redis事务不过是简单用队列包裹起来的一组Redis命令，Redis并未对事务提供额外的持久化功能，所以Redis事务的持久性**由Redis所使用的持久化模式有关：**

- 在无持久化的内存模式下运作时，事务**不具有持久性**。
- 当服务器在RDB持久化模式下运作时，服务器只在特定条件被满足时，才会执行BGSAVE命令，并且异步执行的BGSAVE不能保证事务数据在第一时间被保存到RDB日志中，**不能保证持久性**。
- 在AOF持久化模式下运行时，当appendfsync选项值为**always**时，这种配置下的事务**具有持久性。**
- 在AOF持久化模式下运行时，当appendfsync选项值为**everysec**时,程序每秒同步一次数据到硬盘，可能会造成事务数据的丢失，**不具有持久性。**
- 在AOF持久化模式下运行时，当appendfsync选项值为**no**时,何时将数据写入硬盘由操作系统决定，事务数据可能在等待同步的过程中丢失，**不具有持久性。**

## 为什么事务不支持回滚？

作者不支持事务回滚的原因有以下两个：

- 他认为 Redis 事务的执行时，错误**通常都是编程错误造成**的，这种错误通常只会出现在开发环境中，而很少会在实际的生产环境中出现，所以他认为没有必要为 Redis 开发事务回滚功能；
- 不支持事务回滚是因为这种**复杂的功能和 Redis 追求的简单高效**的设计主旨不符合。

## Watch命令的实现

WATCH命令是一个乐观锁，客户端可以在开启事务（multi）之前watch任意数量的key,如果这些key在exec之前被其它客户端修改了，那么服务器将拒绝执行事务，并向客户端返回**执行失败的空回复。**

![image-20221019141656709](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221019141656709.png)

当客户端发来EXEC命令时，服务器会根据这个客户端**是否打开了REDIS_DIRTY_CAS**标识来决定是否执行事务。在执行WATCH命令后，如果watch_keys字典中的数据被修改时，REDIS_CIRTY_CAS标识被打开。如果标识被打开，说明客户端监视的key至少有一个被修改，事务无法执行，如果没有被打开，事务正常执行。

# 七、面试题扫盲篇

## 1.为什么用 Redis 作为 MySQL 的缓存？

***1、Redis 具备高性能***

假如用户第一次访问 MySQL 中的某些数据。这个过程会比较慢，因为是从硬盘上读取的。而Redis中的数据是在内存中的，操作 Redis 缓存就是直接操作内存，所以速度相当快。

***2、 Redis 具备高并发***

单台设备的 Redis 的 QPS（Query Per Second，每秒钟处理完请求的次数） **是 MySQL 的 10 倍**，Redis 单机的 QPS 能轻松破 10w，而 MySQL 单机的 QPS 很难破 1w。

## 2.Redis 数据类型以及使用场景分别是什么？

Redis 提供了丰富的数据类型，常见的有五种数据类型：**String（字符串），Hash（哈希），List（列表），Set（集合）、Zset（有序集合）**。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%85%AB%E8%82%A1%E6%96%87/%E4%BA%94%E7%A7%8D%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png" alt="img" style="zoom: 50%;" />

随着 Redis 版本的更新，后面又支持了四种数据类型： **BitMap（2.2 版新增）、HyperLogLog（2.8 版新增）、GEO（3.2 版新增）、Stream（5.0 版新增）**。 Redis 五种数据类型的应用场景：

- String 类型的应用场景：缓存对象、常规计数、分布式锁、共享 session 信息等。
- List 类型的应用场景：简单的消息队列（但是有两个问题：1. 生产者需要自行实现全局唯一 ID；2. 不能以消费组形式消费数据）等。
- Hash 类型：缓存对象、购物车等。
- Set 类型：聚合计算（并集、交集、差集）场景，比如点赞、共同关注、抽奖活动等。
- Zset 类型：排序场景，比如排行榜、电话和姓名排序等。

Redis 后续版本又支持四种数据类型，它们的应用场景如下：

- BitMap（2.2 版新增）：二值状态统计的场景，比如签到、判断用户登陆状态、连续签到用户总数等；
- HyperLogLog（2.8 版新增）：海量数据基数统计的场景，比如百万级网页 UV 计数等；
- GEO（3.2 版新增）：存储地理位置信息的场景，比如滴滴叫车，附近商家；
- Stream（5.0 版新增）：消息队列，相比于基于 List 类型实现的消息队列，有这两个特有的特性：自动生成全局唯一消息ID，支持以消费组形式消费数据。

## 3.五种常见的 Redis 数据类型是怎么实现？

### I.String内部实现

String 类型的底层的数据结构实现主要是 **SDS（简单动态字符串）**。 SDS 和我们认识的 C 字符串不太一样，之所以没有使用 C 语言的字符串表示，因为 SDS 相比于 C 的原生字符串：

- **SDS 不仅可以保存文本数据，还可以保存二进制数据**。 SDS 的所有 API 都会以**处理二进制的方式**来处理 SDS 存放在 **buf[] 数组里的**数据。所以 SDS 不光能存放文本数据，而且能保存图片、音频、视频、压缩文件这样的二进制数据。
- **SDS 获取字符串长度的时间复杂度是 O(1)**。SDS 结构里用 **len 属性**记录了字符串长度，所以复杂度为 O(1)。
- **Redis 的 SDS API 是安全的，拼接字符串不会造成缓冲区溢出**。因为 SDS 在拼接字符串之前会检查 SDS 空间是否满足要求，如果空间不够会**自动扩容**，所以不会导致缓冲区溢出的问题。

### II.List内部实现

List 类型的底层数据结构是由**双向链表或压缩列表**实现的：

- 如果列表的元素**个数小于 512 个**（默认值，可由 list-max-ziplist-entries 配置），列表每个元素的值**都小于 64 字节**（默认值，可由 list-max-ziplist-value 配置），Redis 会使用**压缩列表**作为 List 类型的底层数据结构；
- 如果列表的元素不满足上面的条件，Redis 会使用**双向链表**作为 List 类型的底层数据结构；

但是**在 Redis 3.2 版本之后，List 数据类型底层数据结构就只由 quicklist 实现了，替代了双向链表和压缩列表**。

<img src="https://img-blog.csdnimg.cn/img_convert/f46cbe347f65ded522f1cc3fd8dba549.png" alt="img" style="zoom:67%;" />

**quickList的节点属性有一个指向压缩列表结构的指针。**

### III、Hash内部实现

ash 类型的底层数据结构是由**压缩列表或哈希表**实现的：

- 如果哈希类型元素**个数小于 512 个**（默认值，可由 hash-max-ziplist-entries 配置），**所有值小于 64 字节**（默认值，可由 hash-max-ziplist-value 配置）的话，Redis 会使用**压缩列表**作为 Hash 类型的底层数据结构；
  - 同一键值对的两个Hash对象会紧紧挨在一起，键在前，值在后
  - 先添加到Hash对象的键值对放在表头，后添加的放在表尾。
  - <img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221113144305712.png" alt="image-20221113144305712" style="zoom:37%;" />

- 如果哈希类型元素不满足上面条件，Redis 会使用**哈希表**作为 Hash 类型的底层数据结构。
  - <img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221113144426496.png" alt="image-20221113144426496" style="zoom:47%;" />


**在 Redis 7.0 中，压缩列表数据结构已经废弃了，交由 listpack 数据结构来实现了**

### IV、Set内部实现

Set 类型的底层数据结构是由**哈希表或整数集合**实现的：

- 如果集合中的元素**都是整数**且元素个数小于 512 （默认值，set-maxintset-entries配置）个，Redis 会使用**整数集合**作为 Set 类型的底层数据结构；
- 如果集合中的元素不满足上面条件，则 Redis 使用**哈希表**作为 Set 类型的底层数据结构。

### V.Zset内部实现

- 如果有序集合的元素个数**小于 128 个**，并且每个元素的值**小于 64 字节时**，Redis 会使用**压缩列表**作为 Zset 类型的底层数据结构，它会将成员与成员的分值同时存入压缩列表中，分值小的元素靠近表头，分值大的元素靠近表尾；
  - ![image-20221113145236991](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221113145236991.png)

- 如果有序集合的元素不满足上面的条件，Redis 会使用**跳表** **+ 字典**作为 Zset 类型的底层数据结构；跳表中每个节点存储分值（权重）和成员本身，字典中每个节点的key保存了成员本身，value为成员的分值。
  - <img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221113145610554.png" alt="image-20221113145610554" style="zoom:70%;" />


#### 为什么要同时用跳表和字典来实现有序集合？

- 如果只用字典，虽然能以O(1)的时间复杂度查询到成员的分值`（ZSCORE key member）`，但是进行范围查询`（ZRANGE、ZRANK）`的时候，就要对字典保存的所有元素进行排序，这种排序时间复杂度至少O(NlogN)，空间复杂度为O(N)。
- 如果只用跳表的话，按成员查找分值这一操作的时间复杂度由O(1)上升为O(NlogN)
- 综上，跳表和字典在ZSet功能的实现算法优化方面相辅相成，并且在实际情况下，跳表和字典中会共享成员及其分值，不会造成内存浪费。

**在 Redis 7.0 中，压缩列表数据结构已经废弃了，交由 listpack 数据结构来实现了。**

## 4.Redis是单线程吗？

**Redis 单线程指的是「接收客户端请求->解析请求 ->进行数据读写等操作->发送数据给客户端」这个过程是由一个线程（主线程）来完成的**，这也是我们常说 Redis 是单线程的原因。

但是，**Redis 程序并不是单线程的**，Redis 在启动的时候，是会**启动后台线程**（BIO）的：

- **Redis 在 2.6 版本**，会启动 2 个后台线程，分别处理关闭文件、AOF 刷盘这两个任务；
- **Redis 在 4.0 版本之后**，新增了一个新的后台线程，用来**异步释放 Redis 内存**，也就是 lazyfree 线程。例如执行 unlink key / flushdb async / flushall async 等命令，会把这些删除操作交给后台线程来执行，

后台线程相当于一个消费者，生产者把耗时任务丢到任务队列中，消费者（BIO）不停轮询这个队列，拿出任务就去执行对应的方法即可。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%85%AB%E8%82%A1%E6%96%87/%E5%90%8E%E5%8F%B0%E7%BA%BF%E7%A8%8B.jpg" alt="img" style="zoom: 50%;" />

关闭文件、AOF 刷盘、释放内存这三个任务都有各自的任务队列：

- BIO_CLOSE_FILE，关闭文件任务队列：当队列有任务后，后台线程会调用 close(fd) ，将文件关闭；
- BIO_AOF_FSYNC，AOF刷盘任务队列：当 AOF 日志配置成 everysec 选项后，主线程会把 AOF 写日志操作封装成一个任务，也放到队列中。当发现队列有任务后，后台线程会调用 fsync(fd)，将 AOF 文件刷盘，
- BIO_LAZY_FREE，lazy free 任务队列：当队列有任务后，后台线程会 free(obj) 释放对象 / free(dict) 删除数据库所有对象 / free(skiplist) 释放跳表对象；

## 5、Redis单线程模型是怎样实现的？

Redis 6.0 版本之前的单线模式如下图：

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%85%AB%E8%82%A1%E6%96%87/redis%E5%8D%95%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.drawio.png" alt="img" style="zoom: 45%;" />

图中的**蓝色部分是一个事件循环，是由主线程负责的**，可以看到网络 I/O 和命令处理都是单线程。 Redis 初始化的时候，会做下面这几件事情：

- 首先，调用 epoll_create() 创建一个 epoll 对象和调用 socket() 一个服务端 socket
- 然后，调用 bind() 绑定端口和调用 listen() 监听该 socket；
- 然后，将调用 epoll_ctl() 将 listen socket **加入到 epoll**，同时注册「连接事件」处理函数。

- 接着，调用 epoll_wait 函数等待事件的到来：
  - 如果是**连接事件**到来，则会调用**连接事件处理函数**，该函数会做这些事情：调用 accpet 获取已连接的 socket -> 调用 epoll_ctl 将已连接的 socket 加入到 epoll -> 注册「读事件」处理函数；
  - 如果是**读事件**到来，则会调用**读事件处理函数**，该函数会做这些事情：调用 read 获取客户端发送的数据 -> 解析命令 -> 处理命令 -> 将客户端对象添加到发送队列 -> 将执行结果写到发送缓存区等待发送；
  - 如果是**写事件**到来，则会调用**写事件处理函数**，该函数会做这些事情：通过 write 函数将客户端发送缓存区里的数据发送出去，如果这一轮数据没有发送完，就会继续注册写事件处理函数，等待 epoll_wait 发现可写后再处理 。

**一句话：连接事件、读事件、写事件都属于一个事件循环，哪个事件到来，redis线程就处理哪个事件，该单线程得到复用。**

## 6、为什么Redis采用单线程还这么快？

- Redis 的大部分操作**都在内存中完成**，并且采用了高效的数据结构，因此 Redis 瓶颈可能是机器的内存或者网络带宽，而并非 CPU，既然 CPU 不是瓶颈，那么自然就采用单线程的解决方案了；
- Redis 采用单线程模型可以**避免了多线程之间的竞争**，省去了多线程切换带来的时间和性能上的开销，而且也不会导致死锁问题。
- Redis 采用了 **I/O 多路复用机制**处理大量的客户端 Socket 请求，IO 多路复用机制是指一个线程处理多个 IO 流，就是我们经常听到的 select/epoll 机制。简单来说，在 Redis 只运行单线程的情况下，该机制允许内核中，同时存在多个监听 Socket 和已连接 Socket。内核会一直监听这些 Socket 上的连接请求或数据请求。一旦有请求到达，就会交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个 IO 流的效果。

**一句话：Redis基于内存，CPU不是瓶颈，单线程避免竞争，无锁且无线程上下文切换，IO多路复用，一个线程处理多个IO流。**

## 7.Redis6.0之后为什么引入多线程？

**因为随着网络硬件的性能提升，Redis 的性能瓶颈有时会出现在网络 I/O 的处理上。**

所以为了**提高网络 I/O 的并行度**，Redis 6.0 对于网络 I/O 采用多线程来处理。**但是对于命令的执行，Redis 仍然使用单线程来处理。**

Redis 官方表示，**Redis 6.0 版本引入的多线程 I/O 特性对性能提升至少是一倍以上**。

Redis 6.0 版本支持的 I/O 多线程特性，默认情况下 I/O 多线程只针对发送响应数据（write client socket），并不会以多线程的方式处理读请求（read client socket）。要想开启多线程处理客户端读请求，就需要把 Redis.conf 配置文件中的 io-threads-do-reads 配置项设为 yes。

```c
//读请求也使用io多线程
io-threads-do-reads yes 
```

同时， Redis.conf 配置文件中提供了 IO 多线程个数的配置项。

```c
// io-threads N，表示启用 N-1 个 I/O 多线程（主线程也算一个 I/O 线程）
io-threads 4 
```

关于线程数的设置，官方的建议是如果为 **4 核的 CPU**，建议线程数设置为 **2 或 3**，如果为 **8 核 CPU** 建议线程数设置为 **6**，线程数一定要小于机器核数，线程数并不是越大越好，因为线程数越大，线程上下文切换开销也大，影响CPU性能。

因此， Redis 6.0 版本之后，Redis 在启动的时候，默认**情况下会创建 6 个线程**：

- Redis-server ： Redis的主线程，主要负责执行命令；
- bio_close_file、bio_aof_fsync、bio_lazy_free：三个后台线程，分别异步处理关闭文件任务、AOF刷盘任务、释放内存任务；
- io_thd_1、io_thd_2、io_thd_3：**三个 I/O 线程**，io-threads 默认是 4 ，所以会启动 3（4-1）个 I/O 多线程，用来分担 Redis 网络 I/O 的压力

## 8.Redis 如何实现数据不丢失？

Redis 的读写操作都是在内存中，所以 Redis 性能才会高，但是当 Redis 重启后，内存中的数据就会丢失，那为了保证内存中的数据不会丢失，Redis 实现了数据持久化的机制，这个机制会把数据存储到磁盘，这样在 Redis 重启就能够从磁盘中恢复原有的数据。

Redis 共有三种数据持久化的方式：

- **AOF 日志**：每执行一条写操作命令，就把该命令以追加的方式写入到一个文件里；
- **RDB 快照**：将某一时刻的内存数据，以二进制的方式写入磁盘；
- **混合持久化方式**：Redis 4.0 新增的方式，**集成了 AOF 和 RBD 的优点；**

## 9.AOF 日志是如何实现的？

Redis 在执行完一条写操作命令后，就会把该命令以追加的方式写入到一个文件里，然后 Redis 重启时，会读取该文件记录的命令，然后逐一执行命令的方式来进行数据恢复。

### 写AOF时为什么要先执行命令再写日志？

- **避免额外的检查开销**：因为如果先将写操作命令记录到 AOF 日志里，再执行该命令的话，如果当前的命令语法有问题，那么如果不进行命令语法检查，该错误的命令记录到 AOF 日志里后，Redis 在使用日志恢复数据时，就可能会出错。
- **不会阻塞当前写操作命令的执行**：因为当写操作命令执行成功后，才会将命令记录到 AOF 日志。

**带来的风险：**

- **数据可能会丢失：** 执行写操作命令和记录日志是两个过程，那当 Redis 在还没来得及将命令写入到硬盘时，服务器发生宕机了，这个数据就会有丢失的风险。
- **可能阻塞其他操作：** 由于写操作命令执行成功后才记录到 AOF 日志，所以不会阻塞当前命令的执行，但因为 AOF 日志也是在主线程中执行，所以当 Redis 把日志文件写入磁盘的时候，还是会阻塞后续的操作无法执行。

### AOF的写回策略有几种？

Redis 提供了 **3 种写回硬盘的策略**，**控制的就是fsync()调用的时机**。 在 Redis.conf 配置文件中的 appendfsync 配置项可以有以下 3 种参数可填：

- **Always**，这个单词的意思是「总是」，所以它的意思是每次写操作命令执行完后，同步将 AOF 日志数据写回硬盘；
- **Everysec**，这个单词的意思是「每秒」，所以它的意思是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，然后每隔一秒将缓冲区里的内容写回到硬盘；
- **No**，意味着不由 Redis 控制写回硬盘的时机，转交给操作系统控制写回的时机，也就是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，再由操作系统决定何时将缓冲区内容写回硬盘。

### 各种写回策略的优缺点？

<img src="https://img-blog.csdnimg.cn/img_convert/98987d9417b2bab43087f45fc959d32a.png" alt="img" style="zoom:67%;" />

### AOF日志过大，会触发什么机制？

Redis 为了避免 AOF 文件越写越大，提供了 **AOF 重写机制**，当 AOF 文件的大小超过所设定的阈值后，Redis 就会启用 AOF 重写机制，来压缩 AOF 文件。

在没有使用重写机制前，假设前后执行了「*set name csg*」和「*set name victorG*」这两个命令的话，就会将这两个命令记录到 AOF 文件。

但是**在使用重写机制后，就会读取 name 最新的 value（键值对） ，然后用一条 「set name victorG」命令记录到新的 AOF 文件**，之前的**第一个命令就没有必要记录**了，因为它属于「历史」命令，没有作用了。这样一来，一个键值对在重写日志中只用一条命令就行了。

重写工作完成后，就会将**新的 AOF 文件覆盖现有的 AOF 文件**，这就相当于压缩了 AOF 文件，使得 AOF 文件体积变小了。

### 10.重写AOF的过程是怎样的？

Redis 的重写 AOF 过程是由**后台子进程 bgrewriteaof**来完成的，这么做可以达到两个好处：

- 子进程进行 AOF 重写期间，主进程可以继续处理命令请求，从而避免阻塞主进程；

- 子进程带有主进程的数据副本，而当父子进程任意一方修改了该共享内存，就会发生**「写时复制」**，于是父子进程就有了独立的数据副本，就**不用加锁**来保证数据安全。

**但是重写过程中，主进程依然可以正常处理命令**，如果主线程此时修改了一个已经存在的键值对，那么此时这个 key-value 数据在子进程的内存数据就跟主进程的内存数据不一致了

为了解决这种数据不一致问题，Redis 设置了一个 **AOF 重写缓冲区**，这个缓冲区在创建 bgrewriteaof 子进程之后开始使用。

在 bgrewriteaof 子进程执行 AOF 重写期间，主进程需要执行以下三个工作:

- 执行客户端发来的命令；
- 将执行后的写命令追加到 「AOF 缓冲区」；
- 将执行后的写命令追加到 「AOF 重写缓冲区」；

在执行完重写任务后，bgrwriteof子进程会向主进程发送信号，然后**把AOF 重写缓冲区中的所有内容追加到新的 AOF 的文件中**，使得新旧两个 AOF 文件所保存的数据库状态一致，最后新的 AOF 的文件进行改名，覆盖现有的 AOF 文件。

## 10.RDB快照如何实现？

因为 AOF 日志记录的是操作命令，不是实际的数据，所以用 AOF 方法做故障恢复时，需要全量把日志都执行一遍，一旦 AOF 日志非常多，势必会造成 Redis 的恢复操作缓慢。

RDB 快照就是记录**某一个瞬间的内存数据**，记录的是实际数据，并且这些数据是以**二进制形式**记录下来的。

因此在 Redis 恢复数据时， **RDB 恢复数据的效率会比 AOF 高些**，因为直接将 RDB 文件读入内存就可以，不需要像 AOF 那样还需要额外执行操作命令的步骤才能恢复数据。

### RDB会阻塞主线程吗？

Redis 提供了两个命令来生成 RDB 文件，分别是 **save 和 bgsave**，他们的区别就在于是否在「主线程」里执行：

- 执行了 save 命令，就会在**主线程生成 RDB 文件**，由于和执行操作命令在同一个线程，所以如果写入 RDB 文件的时间太长，**会阻塞主线程**；
- 执行了 bgsave 命令，会创建一个子进程来生成 RDB 文件，这样可以**避免主线程的阻塞**；

Redis 还可以**通过配置文件**的选项来实现每隔一段时间自动执行一次 bgsave 命令，默认会提供以下配置：

```c
save 900 1
save 300 10
save 60 10000
```

Redis 的快照是**全量快照**，也就是说每次执行快照，都是把内存中的「所有数据」都记录到磁盘中。所以执行快照是一个比较重的操作，如果频率太频繁，可能会对 Redis 性能产生影响。如果频率太低，服务器故障时，丢失的数据会更多。

### RDB在执行快照的过程，数据可以被修改吗？

可以的，执行 bgsave 过程中，Redis 依然**可以继续处理操作命令**的，也就是数据是能被修改的，关键的技术就在于**写时复制技术（Copy-On-Write, COW）。**执行 bgsave 命令的时候，主进程会执行fork()系统调用创建子进程，同时时会创建一个子进程副本（包含了页表的副本），**两份页表指向同一片物理内存区域**，当父子进程修改了数据时，会发生写时复制，被修改的数据会复制一份副本，然后 bgsave 子进程会把该副本数据写入 RDB 文件，在这个过程中，**主线程仍然可以直接修改原来的数据**，被修改的数据并未被写入RDB文件，所以**数据的一致性无法保证。**

## 11.为什么会有混合持久化？

RDB 优点是数据恢复速度快，但是快照的频率不好把握。频率太低，丢失的数据就会比较多，频率太高，就会影响性能。

AOF 优点是丢失数据少，但是数据恢复不快。

为了集成了两者的优点， Redis 4.0 提出了**混合使用 AOF 日志和内存快照**，也叫混合持久化，既保证了 Redis 重启速度，又降低数据丢失风险。

**混合持久化在AOF重写过程中**，当开启混合持久化时，fork()出来的子进程bgrwriteof会以RDB的形式先将数据写进AOF文件，然后主线程处理命令的操作将会被记录到AOF重写缓冲区，缓冲区里的增量命令会以AOF的形式写入AOF文件，写入完成后通知主进程将新的含有 RDB 格式和 AOF 格式的 AOF 文件替换旧的的 AOF 文件。

使用了混合持久化，AOF 文件的**前半部分是 RDB 格式的全量数据，后半部分是 AOF 格式的增量数据**。

**混合持久化优点：**

- 混合持久化结合了 RDB 和 AOF 持久化的优点，开头为 RDB 的格式，使得 Redis 可以更快的启动，同时结合 AOF 的优点，有减低了大量数据丢失的风险。

**混合持久化缺点：**

- AOF 文件中添加了 RDB 格式的内容，使得 AOF 文件的**可读性变得很差**；
- **兼容性差**，如果开启混合持久化，那么此混合持久化 AOF 文件，就不能用在 Redis 4.0 之前版本了。

## 12.Redis如何实现服务高可用？

Redis提供了三种集群模式：主从复制、哨兵模式、分片集群（cluster）。它们都是以主从复制为基石，**主从复制是 Redis 高可用服务的最基础的保证**，当服务出现故障时，集群模式下的各个Redis实例会做出相应反应实现服务的高可用。

### 对于主从复制（能保证数据一致性但不能保证强一致性）：

主从复制共有三种阶段：**全量复制、基于长连接的命令传播、增量复制**。

主从服务器**第一次同步**的时候，会采用全量复制的方式进行同步，但这个过程是异步的，这是新增数据并没有进入从服务器，无法实现一致性。所以有了命令传播阶段。

第一次同步之后，主从服务器之间就会维护着一个长连接，每当主服务器有写入操作的时候，就会通过这个连接将命令传播给从服务器，**从而保证了数据的一致性。**

如果遇到**网络断开，增量复制**就可以上场了，不过这个还跟 repl_backlog_size 这个大小有关系。如果过小，可能导致未同步的数据被新数据覆盖，导致数据丢失，此时主从服务器之间又会开启全量复制，加大了系统开销。所以要调大这个参数的值，以降低主从服务器断开后全量同步的概率。

但是在命令传播阶段，主服务器并不会等到从服务器实际执行完命令后，再把结果返回给客户端，而是主服务器自己在本地执行完命令后，就会向客户端返回结果了。所以主从复制模式并**不能保证数据的强一致性。**

### 对于哨兵模式：

在使用 Redis 主从服务的时候，会有一个问题，就是当 Redis 的主从服务器出现故障宕机时，**需要手动进行恢复。**

为了解决这个问题，Redis 增加了哨兵模式（**Redis Sentinel**），因为哨兵模式做到了可以**监控主从服务器**，并且提供**主从节点故障转移的功能。**

### 对于集群分片模式：

Redis 的**哨兵模式**基本已经可以实现高可用，读写分离 ，但是在这种模式下**每台 Redis 服务器都存储相同的数据，很浪费内存**，所以在redis3.0上加入了 Cluster 集群模式，这也是官方推荐使用的集群方案，实现了 **Redis 的分布式存储**，也就是说**每台 Redis 节点上存储不同的内容**。该模式的基本原理就是**一致性哈希**。并且该模式也提供了故障转移功能。

## 13.集群脑裂数据丢失怎么办？

### 什么是集群脑裂问题？

由于网络问题，集群主节点与从节点或哨兵节点之间失去联系但主节点与客户端之间连接未断，这是**主节点继续处理客户端请求，此时主节点会将数据保存在缓冲区内**。因为主从的失联，从节点或哨兵协商过后，将该主节点判断为客观下线。此时重新向从节点中选举出新的主节点，**这时有两个主节点**。等网络恢复，旧主节点会降级为从节点，这是旧主节点会向新主节点请求同步复制，因为第一次复制是全量复制，旧主节点会清空自己的缓冲区，所以**导致之前客户端写入旧主节点的数据丢失了。**

<img src="https://img-blog.csdnimg.cn/img_convert/6d6ab49a41164844ea4eca6bafcc32a2.png" alt="img" style="zoom:25%;" />

<img src="https://img-blog.csdnimg.cn/img_convert/92e97d78404f59228e0b72a756cdf03a.png" alt="img" style="zoom:25%;" />

### 解决方案：

**主节点时刻检测自己与从节点的网络连接情况**，当主节点发现从节点下线或者通信超时的总数量小于阈值时，那么**禁止主节点接收写数据**，直接把错误返回给客户端。

在 Redis 的配置文件中有两个参数我们可以设置：

- **min-slaves-to-write x**，主节点必须要有至少 x 个从节点连接，如果小于这个数，主节点会禁止写数据。
- **min-slaves-max-lag x**，主从数据复制和同步的延迟不能超过 x 秒，如果超过，主节点会禁止写数据。

我们可以把 min-slaves-to-write 和 min-slaves-max-lag 这两个配置项搭配起来使用，分别给它们设置一定的阈值，假设为 N 和 T。

**当满足以上要求时，原主库就会被限制接收客户端写请求，客户端也就不能在原主库中写入新数据了**。

**等到新主库上线时，就只有新主库能接收和处理客户端请求，此时，新写的数据会被直接写到新主库中。而原主库会被哨兵降为从库，即使它的数据被清空了，也不会有新数据丢失。**

假设从库有 K 个，可以将 min-slaves-to-write 设置为 K/2+1（如果 K 等于 1，就设为 1），将 min-slaves-max-lag 设置为十几秒（例如 10～20s），在这个配置下，如果有一半以上的从库和主库进行的 ACK 消息延迟超过十几秒，我们就禁止主库接收客户端写请求。

**这种配置的核心在于保证了当新主节点被选举出后旧主节点就不再接收数据，保证主从恢复通信后不再接收数据。**

**注意：主从切换是在主从恢复通信时才开始的，没有恢复之前，旧主库是可以继续处理请求的。当主从恢复通信后，主库立马将数据同步给从库，只有在主从切换结束后，从库才升级为新主库，这是旧主节点发生降级，我们要防止的就是在主从恢复通信后到主从切换结束这个事件段内旧主库还在继续写入新数据。**

## 14.Redis使用的过期删除策略是什么？

当我们对Redis中的键值对设置了过期事件后，Redis会把Key带上一个过期事件存储到**过期字典**中，过期字典中保存了数据库中key的过期时间。

当我们查询或修改一个key时，如会先检查该key是否在过期字典中，若存在，则正常读取该key，否则会检查过期字典中该key过期时间，将其与当前时间做对比来判断是否过期。

Redis 使用的过期删除策略是「**惰性删除+定期删除**」这两种策略配和使用。

**惰性删除：**

不主动删除该key，只有当客户端访问该key时，通过判断是否过期来决定是否删除。

惰性删除策略的**优点**：

- 因为每次访问时，才会检查 key 是否过期，所以此策略只会使用很少的系统资源，因此，惰性删除策略对 CPU 时间最友好。

惰性删除策略的**缺点**：

- 一个key已经过期了，但客户端很久都未再访问该key，那么该key也一直占用内存，对内存空间不友好。

**定期删除：**

每隔一段时间「随机」从数据库中取出一定数量的 key 进行检查，并删除其中的过期key。

Redis 的定期删除的流程：

1. 从过期字典中随机抽取 20 个 key；
2. 检查这 20 个 key 是否过期，并删除已过期的 key；
3. 如果本轮检查的已过期 key 的数量，超过 5 个（20/4），也就是「已过期 key 的数量」占比「随机抽取 key 的数量」**大于 25%**，则继续**重复步骤 1**；如果已过期的 key 比例**小于 25%**，则**停止继续删除过期 key，然后等待下一轮再检查。**

定期删除策略的**优点**：

- 通过限制删除操作执行的时长和频率，来减少删除操作对 CPU 的影响，同时也能删除一部分过期的数据减少了过期键对空间的无效占用。

定期删除策略的**缺点**：

- 难以确定删除操作执行的时长和频率。如果执行的太频繁，就会对 CPU 不友好；如果执行的太少，那又和惰性删除一样了，过期 key 占用的内存不会及时得到释放。

**定期删除 + 惰性删除配合使用的优势：在合理使用CPU资源和内存空间之间得到了平衡。**

## 15.Redis在进行持久化时，对过期键怎么处理？

Redis 持久化文件有两种格式：RDB（Redis Database）和 AOF（Append Only File）。

RDB 文件分为两个阶段，RDB 文件生成阶段和加载阶段。

- **RDB 文件生成阶段**：从内存状态持久化成 RDB（文件）的时候，会对 key 进行过期检查，**过期的键「不会」被保存到新的 RDB 文件中**，因此 Redis 中的过期键不会对生成新 RDB 文件产生任何影响。

- RDB 加载阶段

  RDB 加载阶段时，要看服务器是主服务器还是从服务器，分别对应以下两种情况：

  - **如果 Redis 是「主服务器」运行模式的话，在载入 RDB 文件时，程序会对文件中保存的键进行检查，过期键「不会」被载入到数据库中**。所以过期键不会对载入 RDB 文件的主服务器造成影响；
  - **如果 Redis 是「从服务器」运行模式的话，在载入 RDB 文件时，不论键是否过期都会被载入到数据库中**。但由于主从服务器在进行数据同步时，从服务器的数据会被清空。所以一般来说，过期键对载入 RDB 文件的从服务器也不会造成影响。

AOF 文件分为两个阶段，AOF 文件写入阶段和 AOF 重写阶段。

- **AOF 文件写入阶段**：当 Redis 以 AOF 模式持久化时，**如果数据库某个过期键还没被删除，那么 AOF 文件会保留此过期键，当此过期键被删除后，Redis 会向 AOF 文件追加一条 DEL 命令来显式地删除该键值**。
- **AOF 重写阶段**：执行 AOF 重写时，会对 Redis 中的键值对进行检查，**已过期的键不会被保存到重写后的 AOF 文件中**，因此不会对 AOF 重写造成任何影响。

## 16、Redis主从模式中，过期键是怎么处理的？

当 Redis 运行在主从模式下时，**从库不会进行过期扫描，从库对过期的处理是被动的**。也就是即使从库中的 key 过期了，如果有客户端访问从库时，依然可以得到 key 对应的值，像未过期的键值对一样返回。

从库的过期键处理依靠主服务器控制**（命令传播）**，**主库在 key 到期时，会在 AOF 文件里增加一条 del 指令，同步到所有的从库**，从库通过执行这条 del 指令来删除过期的 key。

## 17.Redis 内存淘汰策略有哪些？

在 Redis 的运行内存达到了某个阀值，就会触发**内存淘汰机制**，Redis 内存淘汰策略共有八种，这八种策略大体分为「不进行数据淘汰」和「进行数据淘汰」两类策略。

***1、不进行数据淘汰的策略***

**noeviction**（Redis3.0之后，默认的内存淘汰策略） ：它表示当运行内存超过最大设置内存时，不淘汰任何数据，而是**不再提供服务**，直接返回错误。

***2、进行数据淘汰的策略***

针对「进行数据淘汰」这一类策略，又可以细分为「在设置了过期时间的数据中进行淘汰」和「在所有数据范围内进行淘汰」这两类策略。 在设置了过期时间的数据中进行淘汰：

- **volatile-random**：随机淘汰设置了过期时间的任意键值；
- **volatile-ttl**：优先淘汰更早过期的键值。
- **volatile-lru**（Redis3.0 之前，默认的内存淘汰策略）：淘汰所有设置了过期时间的键值中，**最久未使用**的键值；
- **volatile-lfu**（Redis 4.0 后新增的内存淘汰策略）：淘汰所有设置了过期时间的键值中，**最少使用**的键值；

在所有数据范围内进行淘汰：

- **allkeys-random**：随机淘汰任意键值;
- **allkeys-lru**：淘汰整个键值中最久未使用的键值；
- **allkeys-lfu**（Redis 4.0 后新增的内存淘汰策略）：**淘汰整个键值中最少使用**的键值。

## 18、LRU算法和LFU算法有什么区别？

LRU(Least Recently Used)，最近最少使用，传统的LRU算法基于链表实现，当一个数据被访问时，该数据所在的节点就会被移到链表头部，当要淘汰数据时，把链表尾数据淘汰即可。

LFU(Least Frequently Used), 最不常使用，LFU算法是根据**访问的频次**来淘汰数据的，LFU算法会记录数据的访问次数，如果一个数据在一个时间段内被多次使用，数据的使用频次会上升，淘汰数据是根据一个数据被访问频率来淘汰的，这种算法避免了LRU算法中**“某个不常用的数据被突然访问而逃过了被淘汰命运”**的缺点。

## 19、Redis是如何实现LRU和LFU的？

### LRU：

Redis 实现的是一种**近似 LRU 算法**，目的是为了更好的节约内存，它的**实现方式是在 Redis 的对象结构体中添加一个额外的字段，用于记录此数据的最后一次访问时间**。

当 Redis 进行内存淘汰时，会使用**随机采样的方式来淘汰数据**，它是随机取 5 个值（此值可配置），然后**淘汰最久没有使用的那个**。

Redis 实现的 LRU 算法的**优点**：

- 不用为所有的数据维护一个大链表，节省了空间占用；
- 不用在每次数据访问时都移动链表项，提升了缓存的性能；

### LFU：

Redis对象头的 24 bits 的 lru 字段被分成两段来存储，高 16bit 存储 ldt(Last Decrement Time)，用来记录 key 的访问时间戳；低 8bit 存储 logc(Logistic Counter)，用来记录 key 的**访问频次，它会随着时间的推移而减小**。

- ldt 是用来记录 key 的**访问时间戳**；
- logc 是用来记录 key 的**访问频次**，它的值越小表示使用频率越低，越容易淘汰，每个新加入的 key 的logc 初始值为 5。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E8%BF%87%E6%9C%9F%E7%AD%96%E7%95%A5/lru%E5%AD%97%E6%AE%B5.png)

## 20、解释一下缓存击穿、缓存穿透、缓存雪崩？

### 缓存击穿（针对某个热点数据的过期，数据在数据库中）：

我们的业务通常会有几个数据会被频繁地访问，比如秒杀活动，这类被频地访问的数据被称为热点数据。

如果缓存中的**某个热点数据过期**了，此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库很容易就被高并发的请求冲垮，这就是**缓存击穿**的问题。

### 缓存穿透（数据既不在缓存，也不在数据库，缓存无法构建）：

当用户访问的数据，**既不在缓存中，也不在数据库中**，导致请求在访问缓存时，发现缓存缺失，再去访问数据库时，发现数据库中也没有要访问的数据，没办法构建缓存数据，来服务后续的请求。那么当有大量这样的请求到来时，数据库的压力骤增，这就是**缓存穿透**的问题。

### 缓存雪崩（大量数据同时失效或缓存服务宕机）：

当**大量缓存数据在同一时间过期（失效）或者 Redis 故障宕机**时，如果此时有大量的用户请求，都无法在 Redis 中处理，于是全部请求都直接访问数据库，从而导致数据库的压力骤增，严重的会造成数据库宕机，从而形成一系列连锁反应，造成整个系统崩溃，这就是**缓存雪崩**的问题。

## 21、缓存击穿、缓存穿透、缓存雪崩的解决方案

### 缓存击穿：

- 互斥锁方案（Redis 中使用 setNX 方法设置一个状态位，表示这是一种锁定状态），保证同一时间只有一个业务线程请求缓存，未能获取互斥锁的请求，要么等待锁释放后重新读取缓存，要么就返回空值或者默认值。
- 不给热点数据设置过期时间，由后台异步更新缓存，或者在热点数据准备要过期前，提前通知后台线程更新缓存以及重新设置过期时间；
- 接口限流与**熔断，降级**。重要的接口一定要做好**限流策略**，防止用户恶意刷接口，同时要降级准备，当接口中的某些 服务 不可用时候，进行熔断，失败快速返回机制。**（可用SpringcloudAlibaba的Sentinel组件实现，或Springcloud的Hystix）**

### 缓存穿透：

缓存穿透的发生一般有这**两种情况**：

- 业务误操作，缓存中的数据和数据库中的数据都被误删除了，所以导致缓存和数据库中都没有数据；
- 黑客恶意攻击，故意大量访问某些读取不存在数据的业务；

应对缓存穿透的方案，常见的方案有三种。

- **非法请求的限制**：当有大量恶意请求访问不存在的数据的时候，也会发生缓存穿透，因此在 API 入口处我们要判断求请求参数是否合理，请求参数是否含有非法值、请求字段是否存在，如果判断出是恶意请求就直接返回错误，避免进一步访问缓存和数据库。
- **设置空值或者默认值**：当我们线上业务发现缓存穿透的现象时，可以针对查询的数据，在缓存中设置一个空值或者默认值，这样后续请求就可以从缓存中读取到空值或者默认值，返回给应用，而不会继续查询数据库。
- **使用布隆过滤器快速判断数据是否存在，避免通过查询数据库来判断数据是否存在**：我们可以在写入数据库数据时，使用布隆过滤器做个标记，然后在用户请求到来时，业务线程确认缓存失效后，可以**通过查询布隆过滤器快速判断数据是否存在**，如果不存在，就不用通过查询数据库来判断数据是否存在，即使发生了缓存穿透，大量请求只会查询 Redis 和布隆过滤器，而不会查询数据库，保证了数据库能正常运行，**Redis 自身也是支持布隆过滤器**的。

#### 布隆过滤器：

1. 通过K个哈希函数计算该数据，对应计算出的K个hash值
2. 通过hash值找到对应的二进制的数组下标
3. 判断：如果存在一处位置的二进制数据是0，那么该数据不存在。如果都是1，该数据**可能**存在集合中。

<img src="https://img-blog.csdnimg.cn/img_convert/bab5a3609761d1673465184227161021.png" alt="img" style="zoom: 80%;" />

**优点：**

- 由于存储的是二进制数据，所以占用的空间很小；
- 它的插入和查询速度是非常快的，时间复杂度是O（K）；
- 保密性很好，因为本身不存储任何原始数据，只有二进制数据。

**缺点：**

- 概率型数据结构，可能不准确，只能确定某元素一定不存在或者可能存在，不能确定某元素真的存在
- 删除困难，由于映射的K个点中可能出现多个元素共用的情况，删除的过程中，这几个点的变化可能会涉及其他值的变化，所以不能轻易易删除
  

**Redisson 实现布隆过滤器：**

```java
import org.redisson.Redisson;
import org.redisson.api.RBloomFilter;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
 
public class RedissonBloomFilter {
 
    public static void main(String[] args) {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://192.168.14.104:6379");
        config.useSingleServer().setPassword("123");
        //构造Redisson
        RedissonClient redisson = Redisson.create(config);
 
        RBloomFilter<String> bloomFilter = redisson.getBloomFilter("phoneList");
        //初始化布隆过滤器：预计元素为100000000L,误差率为3%
        bloomFilter.tryInit(100000000L,0.03);
        //将号码10086插入到布隆过滤器中
        bloomFilter.add("10086");
 
        //判断下面号码是否在布隆过滤器中
        System.out.println(bloomFilter.contains("123456"));//false
        System.out.println(bloomFilter.contains("10086"));//true
    }
}
```

### 缓存雪崩：

- **将缓存失效时间随机打散：** 我们可以在原有的失效时间基础上增加一个随机值（比如 1 到 10 分钟）这样每个缓存的过期时间都不重复了，也就降低了缓存集体失效的概率。
- **设置缓存不过期：** 我们可以通过后台服务来更新缓存数据，从而避免因为缓存失效造成的缓存雪崩，也可以在一定程度上避免缓存并发问题。

缓存雪崩的事前事中事后的解决方案如下：

- 事前：Redis 高可用，主从+哨兵，Redis cluster，避免全盘崩溃。
- 事中：本地 ehcache 缓存 + hystrix 或Sentinel限流&降级，避免 MySQL 被打死。
- 事后：Redis 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据。

## 22.如何设计一个缓存策略，可以动态缓存热点数据呢？

系统并不是将所有数据都需要存放到缓存中的，而**只是将其中一部分热点数据缓存起来**，所以我们要设计一个热点数据动态缓存的策略。

热点数据动态缓存的策略总体思路：**通过数据最新访问时间来做排名，并过滤掉不常访问的数据，只留下经常访问的数据**。

以**电商平台场景**中的例子，现在要求只缓存用户经常访问的 **Top 1000 的商品**。具体细节如下：

- 先通过缓存系统做一个**排序队列（比如存放 1000 个商品）**，系统会根据商品的访问时间，更新队列信息，越是最近访问的商品排名越靠前；
- 同时系统会定期过滤掉队列中排名最后的 200 个商品，然后再从数据库中随机读取出 200 个商品加入队列中；
- 这样当请求每次到达的时候，会先从队列中获取商品 ID，如果命中，就根据 ID 再从另一个缓存数据结构中读取实际的商品信息，并返回。

在 Redis 中可以用 **zadd 方法和 zrange 方法**来完成排序队列和获取 200 个商品的操作。

**StringRedisTemplate可以实现，根据索引删除前200个元素，以时间戳为score，获取后200个元素。**

## 23.说说常见的缓存更新策略？

常见的缓存更新策略共有3种：

- Cache Aside（旁路缓存）策略；
- Read/Write Through（读穿 / 写穿）策略；
- Write Back（写回）策略；

实际开发中，Redis 和 MySQL 的更新策略用的是 **Cache Aside**，另外两种策略应用不了。

### Cache Aside（旁路缓存）策略

<img src="https://img-blog.csdnimg.cn/img_convert/6e3db3ba2f829ddc14237f5c7c00e7ce.png" alt="img" style="zoom:67%;" />

Cache Aside（旁路缓存）策略是最常用的，应用程序直接与「数据库、缓存」交互，并负责对缓存的维护，该策略又可以细分为「读策略」和「写策略」。

**写策略步骤：**

- 先更新数据库再删除缓存

**读策略步骤：**

- 先查询缓存，如果未命中，则查询数据库，最后将查询出来的数据缓存并返回给用户。

**注意！**写策略的步骤的顺序顺序不能倒过来，即**不能先删除缓存再更新数据库**，原因是在「读+写」并发的时候，会出现缓存和数据库的数据不一致性的问题。

**举例分析先删缓存再更新数据库的分析：**

假设有<k1 100>这样一个数据线程A要求将100改为200，此时缓存被删除，在更新数据库之前，线程A来读，发现缓存失效了，于是到数据库中读取，读出来的仍是旧值100，于是把旧值缓存起来，此时线程A将数据库更新为200，但之后的线程进来读取该数据时，读到的都是旧数据100，最终导致了缓存和数据库的不一致。

**当然先更新数据库再删除缓存也会有缓存与数据库数据不一致的问题：**

**比如一个线程访问了一个缓存中不存在的数据时会发生。**

<img src="https://img-blog.csdnimg.cn/img_convert/cc208c2931b4e889d1a58cb655537767.png" alt="img" style="zoom:67%;" />

**但是这种情况发生的概率比较小，因为缓存的写入通常要远远快于数据库的写入**

**Cache Aside 策略适合读多写少的场景，不适合写多的场景**，因为当写入比较频繁时，缓存中的数据会被频繁地清理，这样会对缓存的命中率有一些影响。如果业务对缓存命中率有严格的要求，那么可以考虑两种解决方案：

- 一种做法是在**更新数据时也更新缓存**，只是在**更新缓存前先加一个分布式锁**，因为这样在同一时间只允许一个线程更新缓存，就不会产生并发问题了。当然这么做对于写入的性能会有一些影响；
- 另一种做法同样也是在**更新数据时更新缓存**，只是给缓存加一个**较短的过期时间**，这样**即使出现缓存不一致**的情况，缓存的数据也会**很快过期**，对业务的影响也是可以接受。

### Read/Write Through（读穿 / 写穿）策略

Read/Write Through（读穿 / 写穿）策略原则是**应用程序只和缓存交互，不再和数据库交互**，而是由缓存和数据库交互，相当于更新数据库的操作由缓存自己代理了。

***1、Read Through 策略***

先查询缓存中数据是否存在，如果存在则直接返回，如果不存在，则由**缓存组件负责从数据库查询数据**，并将结果写入到缓存组件，最后缓存组件将数据返回给应用。

***2、Write Through 策略***

当有数据更新的时候，先查询要写入的数据在缓存中是否已经存在：

- 如果缓存中数据已经存在，则更新缓存中的数据，并且由**缓存组件同步更新**到数据库中，然后缓存组件告知应用程序更新完成。
- 如果缓存中数据不存在，直接更新数据库，然后返回；

**无论是 Memcached 还是 Redis 都不提供写入数据库和自动加载数据库中的数据的功能。**而我们在使用本地缓存的时候可以考虑使用这种策略。

### Write Back（写回）策略

Write Back（写回）策略在更新数据的时候，**只更新缓存，同时将缓存数据设置为脏的，然后立马返回，并不会更新数据库**。对于数据库的更新，会通过批量异步更新的方式进行。

实际上，Write Back（写回）策略也不能应用到我们常用的数据库和缓存的场景中，因为 Redis 并没有异步更新数据库的功能。

Write Back 是计算机体系结构中的设计，比如 **CPU 的缓存**、操作系统中**文件系统的缓存**都采用了 Write Back（写回）策略。

## 24.Redis如何实现延迟队列

延迟队列是指把**当前要做的事情，往后推迟一段时间再做**。延迟队列的常见使用场景有以下几种：

- 在淘宝、京东等购物平台上下单，超过一定时间未付款，订单会自动取消；
- 打车的时候，在规定时间没有车主接单，平台会取消你的单并提醒你暂时没有车主接单；
- 点外卖的时候，如果商家在10分钟还没接单，就会自动取消订单；

Redis可以用Zset的方式来实现延迟队列，ZSet 有一个 **Score 属性可以用来存储延迟执行的时间。**

- 延迟队列可以通过redis的zset（有序列表）实现。我们将消息序列化成一个字符串作为zset的value，这个消息的到期处理时间作为score，然后用多个线程轮询zset获取到期的任务进行处理
- 多个线程是为了保障可用性，万一挂了一个线程还有其他线程可以继续处理
- 因为有多个线程，所以需要考虑并非争抢任务，确保任务不会被多次执行

**1.生产者实现**
可以看到生产者很简单，其实就是利用zset的特性，给一个zset添加元素而已，而时间就是它的score。

@Component
@Slf4j
public class RedisKeyExpirationListener extends KeyExpirationEventMessageListener  {

```java
@Autowired
private KafkaProducerService kafkaProducerService;

public RedisKeyExpirationListener(RedisMessageListenerContainer listenerContainer) {
    super(listenerContainer);
}

/**
 * 针对 redis 数据失效事件，进行数据处理
 * @param message
 * @param pattern
 */
@Override
public void onMessage(Message message, byte[] pattern){
    if(message == null || StringUtils.isEmpty(message.toString())){
        return;
    }
    String content = message.toString();
    //key的格式为   flag:时效类型:运单号  示例如下
    try {
        if(content.startsWith(AbnConstant.EMS)){
            kafkaProducerService.sendMessageSync(TopicConstant.EMS_WAYBILL_ABN_QUEUE,content);
        }else if(content.startsWith(AbnConstant.YUNDA)){
            kafkaProducerService.sendMessageSync(TopicConstant.YUNDA_WAYBILL_ABN_QUEUE,content);
        }
    } catch (Exception e) {
        log.error("监控过期key，发送kafka异常，",e);
    }
}
```
}
**2,消费者实现**
就是把已经过期的zset中的元素,比如订单id，直接删除然后给用户发消息。

```java
public void consumer() {
    Executors.newSingleThreadExecutor().submit(new Runnable() {
        @Override
        public void run() {
            while (true) {
                Set<String> taskIdSet = RedisOps.getJedis().zrangeByScore(RedisOps.key, 0, 				System.currentTimeMillis(), 0, 1);
                if (taskIdSet == null || taskIdSet.isEmpty()) {
                    System.out.println("没有任务");
            } else {
                taskIdSet.forEach(id -> {
                    long result = RedisOps.getJedis().zrem(RedisOps.key, id);
                    if (result == 1L) {
                        System.out.println("从延时队列中获取到任务，taskId:" + id + " , 当前时间：" + LocalDateTime.now());
                    }
                });
            }
            try {
                TimeUnit.MILLISECONDS.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
});
}
```



## 25.Redis如何处理大Key?

一般而言，下面这两种情况被称为大 key：

- String 类型的值大于 10 KB；
- Hash、List、Set、ZSet 类型的元素(member)的个数超过 5000个；

### 大 key 会造成什么问题？

- **客户端超时阻塞**。由于 Redis 执行命令是单线程处理，然后在操作大 key 时会比较耗时，那么就会阻塞 Redis，从客户端这一视角看，就是很久很久都没有响应。
- **引发网络阻塞**。每次获取大 key 产生的网络流量较大，如果一个 key 的大小是 1 MB，每秒访问量为 1000，那么每秒会产生 1000MB 的流量，这对于普通千兆网卡的服务器来说是灾难性的。
- **阻塞工作线程**。如果使用 **del 删除**大 key 时，会阻塞工作线程，这样就没办法处理后续的命令。
- **内存分布不均**。集群模型在 slot 分片均匀情况下，会出现**数据和查询倾斜**情况，部分**有大 key 的 Redis 节点占用内存多**，QPS 也会比较大。

### 如何查找大key?

***1、redis-cli --bigkeys 查找大key***

可以通过 redis-cli --bigkeys 命令查找大 key：

```shell
[root@gcs100 ~]# redis-cli --bigkeys

# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).

[00.00%] Biggest string found so far '"hobby"' with 12 bytes
[00.00%] Biggest stream found so far '"mq"' with 1 entries
[00.00%] Biggest hash   found so far '"user01"' with 2 fields
[37.93%] Biggest stream found so far '"mystream"' with 3 entries
[37.93%] Biggest zset   found so far '"cars:locations"' with 2 members

-------- summary -------

Sampled 29 keys in the keyspace!
Total key length in bytes is 415 (avg len 14.31)

Biggest   hash found '"user01"' has 2 fields
Biggest string found '"hobby"' has 12 bytes
Biggest stream found '"mystream"' has 3 entries
Biggest   zset found '"cars:locations"' has 2 members

0 lists with 0 items (00.00% of keys, avg size 0.00)
1 hashs with 2 fields (03.45% of keys, avg size 2.00)
24 strings with 50 bytes (82.76% of keys, avg size 2.08)
3 streams with 4 entries (10.34% of keys, avg size 1.33)
0 sets with 0 members (00.00% of keys, avg size 0.00)
1 zsets with 2 members (03.45% of keys, avg size 2.00)

```

使用的时候注意事项：

- 最好选择在**从节点上执行**该命令。因为主节点上执行时，会阻塞主节点；
- 如果没有从节点，那么可以选择在 Redis 实例业务压力的**低峰阶段进行扫描查询**，以免影响到实例的正常运行；或者可以使用 **-i 参数控制扫描间隔**，避免长时间扫描降低 Redis 实例的性能。

该方式的不足之处：

- 这个方法**只能返回每种类型中最大的那个 bigkey**，无法得到大小排在前 N 位的 bigkey；
- 对于集合类型来说，这个方法**只统计集合元素个数的多少，而不是实际占用的内存量**。但是，一个集合中的元素个数多，并不一定占用的内存就多。因为，有可能每个元素占用的内存很小，这样的话，即使元素个数有很多，总内存开销也不大；

***2、使用 SCAN 命令查找大 key***

使用 SCAN 命令对数据库扫描，然后用 TYPE 命令获取返回的每一个 key 的类型。

对于 String 类型，可以直接使用 STRLEN 命令获取字符串的长度，也就是占用的内存空间字节数。

对于集合类型来说，有两种方法可以获得它占用的内存大小：

- 如果能够预先从业务层知道集合元素的平均大小，那么，可以使用下面的命令获取集合元素的个数，然后乘以集合元素的平均大小，这样就能获得集合占用的内存大小了。List 类型：`LLEN` 命令；Hash 类型：`HLEN` 命令；Set 类型：`SCARD` 命令；Sorted Set 类型：`ZCARD` 命令；
- 如果不能提前知道写入集合的元素大小，可以使用 `MEMORY USAGE` 命令（需要 Redis 4.0 及以上版本），查询一个键值对占用的内存空间。

***3、使用 RdbTools 工具查找大 key***

使用 RdbTools 第三方开源工具，可以用来解析 Redis 快照（RDB）文件，找到其中的大 key。

比如，下面这条命令，**将大于 10 kb 的  key  输出到一个表格文件。**

```shell
rdb dump.rdb -c memory --bytes 10240 -f redis.csv
```

### 如何删除大key?

***1、分批次删除***

对于**删除大 Hash**，使用 `hscan` 命令，每次获取 100 个字段，再用 `hdel` 命令，每次删除 1 个字段。

```python
def del_large_hash():
  r = redis.StrictRedis(host='redis-host1', port=6379)
    large_hash_key ="xxx" #要删除的大hash键名
    cursor = '0'
    while cursor != 0:
        # 使用 hscan 命令，每次获取 100 个字段
        cursor, data = r.hscan(large_hash_key, cursor=cursor, count=100)
        for item in data.items():
                # 再用 hdel 命令，每次删除1个字段
                r.hdel(large_hash_key, item[0])
```

对于**删除大 List**，通过 `ltrim` 命令，每次删除少量元素。

```python
def del_large_list():
  r = redis.StrictRedis(host='redis-host1', port=6379)
  large_list_key = 'xxx'  #要删除的大list的键名
  while r.llen(large_list_key)>0:
      #每次只删除最右100个元素
      r.ltrim(large_list_key, 0, -101) 
```

对于**删除大 Set**，使用 `sscan` 命令，每次扫描集合中 100 个元素，再用 `srem` 命令每次删除一个键。

```python
def del_large_set():
  r = redis.StrictRedis(host='redis-host1', port=6379)
  large_set_key = 'xxx'   # 要删除的大set的键名
  cursor = '0'
  while cursor != 0:
    # 使用 sscan 命令，每次扫描集合中 100 个元素
    cursor, data = r.sscan(large_set_key, cursor=cursor, count=100)
    for item in data:
      # 再用 srem 命令每次删除一个键
      r.srem(large_size_key, item)
```

对于**删除大 ZSet**，使用 `zremrangebyrank` 命令，每次删除 top 100个元素。

```python
def del_large_sortedset():
  r = redis.StrictRedis(host='large_sortedset_key', port=6379)
  large_sortedset_key='xxx'
  while r.zcard(large_sortedset_key)>0:
    # 使用 zremrangebyrank 命令，每次删除 top 100个元素
    r.zremrangebyrank(large_sortedset_key,0,99) 
```

***2、异步删除***

从 Redis 4.0 版本开始，可以采用**异步删除**法，**用 unlink 命令代替 del 来删除**。

这样 Redis 会将这个 key 放入到一个**异步线程中进行删除**，这样不会阻塞主线程。

除了主动调用 unlink 命令实现异步删除之外，我们还可以通过配置参数，达到某些条件的时候自动进行异步删除。

主要有 4 种场景，默认都是关闭的：

```text
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del
noslave-lazy-flush no
```

它们代表的含义如下：

- lazyfree-lazy-eviction：表示当 Redis 运行内存超过 maxmeory 时，是否开启 lazy free 机制删除；
- lazyfree-lazy-expire：表示设置了过期时间的键值，当过期之后是否开启 lazy free 机制删除；
- lazyfree-lazy-server-del：有些指令在处理已存在的键时，会带有一个隐式的 del 键的操作，比如 rename 命令，当目标键已存在，Redis 会先删除目标键，如果这些目标键是一个 big key，就会造成阻塞删除的问题，此配置表示在这种场景中是否开启 lazy free 机制删除；
- slave-lazy-flush：针对 slave (从节点) 进行全量数据同步，slave 在加载 master 的 RDB 文件前，会运行 flushall 来清理自己的数据，它表示此时是否开启 lazy free 机制删除。

**建议开启其中的 lazyfree-lazy-eviction、lazyfree-lazy-expire、lazyfree-lazy-server-del 等配置，这样就可以有效的提高主线程的执行效率。**

## 26.Redis管道有什么作用？

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%85%AB%E8%82%A1%E6%96%87/%E7%AE%A1%E9%81%93%E6%A8%A1%E5%BC%8F.jpg" alt="img" style="zoom:67%;" />

使用**管道技术可以解决多个命令执行时的网络等待**，它是把多个命令整合到一起发送给服务器端处理之后统一返回给客户端，这样就免去了每条命令执行后都要等待的情况，从而有效地**提高了程序的执行效率**。

但使用管道技术也要注意避免发送的命令过大，或管道内的数据太多而导致的网络阻塞。

要注意的是，管道技术本质上是客户端提供的功能，而非 Redis 服务器端的功能。

## 27、Redis与MemCache有什么区别?

### I.支持更多数据类型

Redis 相比 Memcached 来说，拥有更多的数据结构，能支持更丰富的数据操作。如果需要缓存能够支持更复杂的结构和操作， Redis 会是不错的选择。

### II.Redis 原生支持集群模式 

在 Redis3.x 版本中，便能支持 cluster 模式，而 Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据。

### III.性能对比

由于 **Redis 只使用单核，而 Memcached 可以使用多核**，所以平均每一个核上 Redis 在存储小数据时比 Memcached 性能更高。而在 100k 以上的数据中，Memcached 性能要高于 Redis。虽然Redis 最近也在存储大数据的性能上进行优化，但是比起 Memcached，还是稍有逊色。

## 28.RDB的优缺点

- RDB 会生成多个数据文件，每个数据文件都代表了某一个时刻中 Redis 的数据，这种多个数据文件的方式，**非常适合做冷备**，可以将这种完整的数据文件发送到一些远程的安全存储上去，比如说 Amazon 的 S3 云服务上去，在国内可以是阿里云的 ODPS 分布式存储上，以预定好的备份策略来定期备份 Redis 中的数据。
- RDB 对 Redis 对外提供的读写服务，影响非常小，可以让 Redis 保持高性能，因为 Redis 主进程只需要 fork 一个子进程，让子进程执行磁盘 IO 操作来进行 RDB 持久化即可。
- 相对于 AOF 持久化机制来说，**直接基于 RDB 数据文件**来重启和恢复 Redis 进程，更加快速。
- 如果想要在 Redis 故障时，尽可能少的丢失数据，那么 RDB 没有 AOF 好。一般来说，RDB数据快照文件，都是每隔 5 分钟，或者更长时间生成一次，这个时候就得接受一旦 Redis 进程宕机，那么会丢失最近 5 分钟的数据。
- RDB 每次在 fork 子进程来执行 RDB 快照数据文件生成的时候，如果数据文件特别大，可能会导致对客户端提供的服务暂停数毫秒，或者甚至数秒。

## 29.AOF的优缺点

- AOF 可以**更好的保护数据不丢失**，一般 AOF 会每隔 1 秒，通过一个后台线程执行一次fsync 操作，最多丢失 1 秒钟的数据。
- AOF 日志文件以 append-only 模式写入，所以**没有任何磁盘寻址的开销**，写入性能非常高，而且文件不容易破损，即使文件尾部破损，也很容易修复。
- AOF 日志文件即使过大的时候，**出现后台重写操作，也不会影响客户端的读写**。因为在rewrite log 的时候，会对其中的指令进行压缩，创建出一份需要恢复数据的最小日志出来。在创建新日志文件的时候，老的日志文件还是照常写入。当新的 merge 后的日志文件ready 的时候，再交换新老日志文件即可。
- AOF 日志文件的命令通过可读较强的方式进行记录，这个特性非常**适合做灾难性的误删除的紧急恢复**。比如某人不小心用 **flushall 命令**清空了所有数据，只要这个时候后台rewrite 还没有发生，那么就可以**立即拷贝 AOF 文件，将最后一条 flushall 命令给删了**，然后再将该 AOF 文件放回去，就可以通过恢复机制，自动恢复所有数据。
- 对于同一份数据来说，AOF 日志文件通常比 RDB 数据快照文件更大。AOF 开启后，支持的写 QPS 会比 RDB 支持的写 QPS 低，因为 AOF 一般会配置成每秒fsync 一次日志文件，当然，每秒一次 fsync ，性能也还是很高的。（如果实时写入，那么 QPS 会大降，Redis 性能会大大降低）
- 以前 AOF 发生过 bug，就是通过 AOF 记录的日志，进行数据恢复的时候，没有恢复一模一样的数据出来。所以说，类似 AOF 这种较为复杂的基于命令日志 / merge / 回放的方式，比基于 RDB 每次持久化一份完整的数据快照文件的方式，更加脆弱一些，容易有 bug。不过AOF 就是为了避免 rewrite 过程导致的 bug，因此每次 rewrite 并不是基于旧的指令日志进行merge 的，而是基于当时内存中的数据进行指令的重新构建，这样健壮性会好很多。