#### mysql 数据Buffer Pool



![image-20210921004926277](http://typoradabin.oss-cn-shenzhen.aliyuncs.com/img/image-20210921004926277.png)

##### 什么是Buffer Pool:



对数据库执行增删查改操作的时候，实际上主要针对内存里面的Buffer Pool 中的数据进行的，在数据库的所有操作都是先去针对Buffer Pool 的数据执行的，同时配合了后续的redo log 、刷磁盘的机制和操作。



##### 怎么设置Buffer Pool的大小？

​		本质上是一个数据库的一个内存组件，可以理解为他就是一个内存数据结构，所以这个内存数据结构肯定是有一定的大小，不可能无限大的。这个Buffer Pool默认情况下是128MB，所以还是有一点偏小了，所以我们实际在使用过程中调整。

​		如果我们的机器是16核32G，那我们就可以设置Buffer Pool 分配个2GB的内存，只要执行以下命令：

```
		innodb_buffer_pool_size = 2147483648;
```





##### 数据页：Mysql中抽象出来的数据单位

​		数据库的核心数据模型就是表+字段+行的概念。数据库实际的存储不是一行一行的放在Buffer Pool 里面的，而是对数据抽象出来了一个数据页的概念，他把很多行的数据放在了一个数据页里面，也就是说我们磁盘会有很对数据页。可以理解为：**Buffer Pool 中保存的就是一个一个的数据页**

​		磁盘的数据页和Buffer Pool 中的缓存页默认情况下都是16KB，也就是说一页对应了16KB的数据。

​		Buffer Pool中的描述数据大概相当于缓存页大小的百分之五左右，也就是每个描述数据大概是８００个字节左右的大小，假设设置的Buffer Pool 大小是128MB，实际上真正的大小是会超出一些，

![image-20210921235302082](http://typoradabin.oss-cn-shenzhen.aliyuncs.com/img/image-20210921235302082.png)



#####  怎么感知Buffer Pool 中哪些缓存页是空闲的？

​		数据库会给Buffer Pool 设计一个free链表，他是一个双向链表的数据结构，在这个free链表中，每一个节点就是一个空闲的缓存页的描述数据块的地址，也就是说，只要有一个缓存页是空闲的，那么他的描述数据块就会被放入free链表中。

​		另外，这个free链表有一个基础节点，就是他会引用链表的头节点和尾节点，里面存储了链表中有多少个描述数据块的节点，也就是说还存在多少空闲的数据页。

![image-20210922082736264](http://typoradabin.oss-cn-shenzhen.aliyuncs.com/img/image-20210922082736264.png)

##### Buffer Pool 和free中的描述数据会不会重复？

​		其实这个free链表，它本身就是buffer pool 里面的描述数据块组成的，我们可以认为每个描述数据块都有两个指针，分别保存上一个以及下一个节点。

​		其实对于free链表来讲，只有一个基础节点是不属于Buffer Pool 的，他是一个40字节大小的一个节点，存放的就是free链表的头节点的地址，尾节点的地址，以及free链表中当前有多少的节点。

![image-20210922083943570](http://typoradabin.oss-cn-shenzhen.aliyuncs.com/img/image-20210922083943570.png)







