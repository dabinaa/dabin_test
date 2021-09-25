### 基于冷热数据分离思想设计的LRU链表



​		即**数据库真正使用的LRU 链表是分成两份，一份为热数据，一份为冷数据**。他们之间的比例是用过参数：innodb_old_blocks_pct 来控制的，默认为37，即表示冷数据的占比为37，如图所示：

![image-20210923130801222](http://typoradabin.oss-cn-shenzhen.aliyuncs.com/img/image-20210923130801222.png)

为了第一次加载的时候不影响到最近最多使用的缓存页，所以第一次加载缓存的时候默认的起始顺序为冷数据区域的头节点。

​		同时一个数据页缓存到冷数据区域的时候，他会在一定的时间间隔后查询这个数据，则这个数据会更新到热数据区域，对应的时间间隔通过innodb_old_blocks_time参数，默认为1000，单位是毫秒。

​		并且热数据部分的顺序更新规则也有一定的优化，因为热数据区域的数据属于会经常调用，如果每次访问其中一个缓存页的话就将他的顺序调整到LRU链表的最前方的话，这样调整顺序会耗费很多时间，所以这里优化了一点：**只有在热数据的后3/4区域的数据访问才会将其重排到LRU链表的最前方。**



#### 缓存页刷盘的几个时间点

- ​		存在定时任务每隔一点时间会把LRU链表中冷数据区域尾部的一些缓存页进行刷盘，清空这几个缓存页，并将这几个缓存页加入到free链表。并且从flush表中移除。从LRU链表中移除。（不一定针对满了才去处理。）
- ​		将flush链表中的一些缓存页定时刷入磁盘。即会有一个后台线程在数据库不繁忙的时候，找时间将flush链表中的缓存页刷入磁盘，只要该缓存页被刷入磁盘，那么他就会从flush链表以及lru链表中一处，然后加入free链表中。
- ​		如果free真的没有对应的空闲的缓存页的时候，那么就会从lru链表中的尾部选择一个缓存页刷盘，腾出对应的缓存页的存储空间。



#### Mysql的生产优化经验：多个Buffer Pool优化并发能力。

​		**前提**： 当多个线程并发访问一个buffer Pool，这里的操作必然是需要进行加锁操作的，既然是串行消费，对性能一定是有影响的，**如果是Buffer Pool存在对应数据页的缓存页，耗时相对会好一点，并且更新free ，flush，lru等链表都是基于链表进行一些指针操作**，所以性能也是很好的，但是**如果对应的buffer Pool 中没有对应的数据页，则需要从磁盘中进行数据读取，这里就发生了一次磁盘IO**，这里的耗时就会多一些，后面排队的就会多一点。

​		一般来说，Mysql的默认规则是：如果你给Buffer Pool 分配的内存小于1GB，那么最多就只会给你一个Buffer Pool，

但是如果你的机器内存很大，那么你必然会给Buffer Pool 分配比较大的内存，比如给他8G内存，那么此时你可以同时配置多个Buffer Pool 的。

```
举个例子：

innodb_buffer_pool_size  = 8589934592

innodb_buffer_pool_instances=4
```

​		![image-20210925173031092](http://typoradabin.oss-cn-shenzhen.aliyuncs.com/img/image-20210925173031092.png)



#### 如何实现对Buffer Pool 的动态扩容？

按照上面的描述，Buffer Pool 是绝不能支持运行期间动态调整大小的。但是我们如果了解Buffer Pool 中更细致的结构，我们会发现，其中还是可以处理的，其实就是通过chunk机制进行一个处理，也就是说：**Buffer Pool 本质上是由多个chunk组成的，他的大小就是通过innodb_buffer_pool_chunk_size进行设置的**，默认大小为128MB。

​		如果我们要对Buffer Pool 进行扩容的话，只需要申请一系列的chunk，然后再把这些新的chunk分配给对应的Buffer Pool 就可以了。

![image-20210925191000448](http://typoradabin.oss-cn-shenzhen.aliyuncs.com/img/image-20210925191000448.png)



#### Mysql的生产优化经验：如何基于机器配置来合理的设置Buffer Pool 

​	虽然说Mysql大部分的curd都是基于内存进行操作的，但是Buffer Pool 并不是越高越好，因为要考虑到操作系统内核以及其他服务或者数据库处理Buffer Pool 还是存在其他的内存操作，**建议的大小是设置机器内存的50%-60%左右。**

​	确定Buffer Pool的大小后，我们要考虑设置多少个Buffer Pool 以及chunk的大小了，我们要记住以下的一个公式
$$
Buffer pool 总大小 = （chunk大小 ＊　buffer Pool 数量）的倍数。
$$


#### 如何查看数据库配置信息？

 执行 show engine innodb status 可以看到如下数据：

![image-20210925194414272](http://typoradabin.oss-cn-shenzhen.aliyuncs.com/img/image-20210925194414272.png)

![image-20210925194431385](http://typoradabin.oss-cn-shenzhen.aliyuncs.com/img/image-20210925194431385.png)