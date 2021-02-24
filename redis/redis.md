##  1 数据结构

- 全局hash表(数组+链表),和java的hashmap类似

  ```c++
  typedef struct dictEntry {
      void *key; /* key 关键字定义 */ 
      union { 
          void *val; uint64_t u64; /* value 定义 */ 
          int64_t s64;double d; 
      }v;
      struct dictEntry *next; /* 指向下一个键值对节点 */ } dictEntry;
  ```

  

  rehash,是扩容为原来的两倍,只是不像hashmap那样一次性复制完毕,而是采用渐进式rehash

  每一次处理请求时,将相应桶位的原数据迁移到新数组的的原数据

  ![1614087850460]('redis'/1614087850460.png)

- 字典的key使用的是简单动态字符串SDS

  value是存在redisObject中

  ![1614089180353]('redis'/1614089180353.png)

  - 字符串类型的编码格式

    如果是数字类型,int 存储8个字节的长整型,大于8个字节,会使用emstr或raw
  
    embstr,可以存储小于44字节的字符串,会使用sdshdr8,该格式内存连续,申请和释放只要一次;只能只读,一旦修改就会转成raw
    
    raw,可以存大于44字节的字符串,该格式申请和释放都需要两次
  
- zipList 

  压缩列表类似一个数组,数组每一个元素存放一个数据,列表头部有zlbytes(列表大小),zltail(列表尾部偏移量),zllen(列表entry的总个数),body是zlEntry集合,尾部有个zlEnd作为结束

  ![1614167685578]('redis'/1614167685578.png)

  插入和删除的复杂度O(n),查询头结点和尾节点O(1),长度也是O(1)

  zipList的优势相对于数组的优势是,数组元素大小是固定的,用不上也会被占用,而且元素大小受最大元素

  相对于链表的优势,连续内存可以有效利用缓存行的优势

- skipList

  有序链表+索引

  有序链表查找的复杂度为O(n),索引可以理解为每隔一个或多个数据加上指针,构成新一层链表,这样查找数据可以从新链表开始,前进回退,这样查找复杂度可以降为O(n)

- 双向链表
  
- 整数数组
  
  ![1614172348319]('redis'/1614172348319.png)

## 2数据类型

- list 

  ![1614172541130]('redis'/1614172546895.png)

  满足list-max-ziplist-size,使用压缩列表,否则使用双向链表

  使用场景:

  ​	因为是有序的,可以用作队列(先进先出),栈(先进后出)

- Set

  set-max-intset-entries默认512,如果小于512个元素,使用整数数组,如果大于,则使用hash表

  使用场景:

  ​	随机抽奖:

  ​	标签;

  ​	点赞,点赞数

  ​	取交集

- Zset

  zset-max-ziplist-entries 128
  zset-max-ziplist-value 64
  
  小于这两个配置是使用zipList
  
  大于时使用skipList
  
  使用场景
  
  ​	新闻点击数+1,获取排行前几的新闻数
  
- Hash

  hash-max-ziplist-entries 512
  hash-max-ziplist-value 64

  小于这个配置时使用zipList

  大于时使用hash表

  使用场景: 

  ​	购物车:用户id,商品id,商品数量
  
- bitmap 
  
  底层是一个字符串
  
  可以用统计当月的登录次数
  
  布隆过滤器
  
  

## 3.使用场景

(1)聚合统计(取差集,交集,并集)可以使用set ,SUNIONSTORE,SINTERSTORE

​	统计前一天登录用户和今天登录用户的差值,算留存数据

​	可以用set app:昨天 数据

(2)排序统计(List,和zset都可以)

   在面对需要展示最新列表、排行榜等场景时，如果数据更新频繁或者需要分页显示，建议你优先考虑使用 Sorted Set。 

(3) 二值状态统计

   可以bitmap

   比如统计zuoqi,2008年8月登录次数,可以每天setBit user:zuoqi:200808  01 1,然后bitcount

(4)基数统计

​	不要求精确 HyperLogLog ,比如网站的uv