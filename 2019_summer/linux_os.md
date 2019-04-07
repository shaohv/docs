<center>Linux知识要点</center>

---

### 内存管理

#### 页式内存

##### [进程的虚拟地址空间](<http://0xffffff.org/2013/05/23/18-linux-process-address-space/>)

​	![process virtual adress space](http://0xffffff.org/images/18/1.png)

​	如上图所示，地址空间被分成内核空间和用户空间，应用程序通过调用系统调用进入内核，内核中通过copy_from_user将数据从用户空间拷贝到内核空间（同样还有copy_to_user），该过程参考[关于系统调用sethostname的描述](<https://www.cnblogs.com/wuchanming/p/4360277.html>)。以上是进程运行时使用的地址，实际cpu执行指令时，也是用的这个地址，称作逻辑地址。

##### 地址映射

​	用户空间：如下图一句话，逻辑地址 -> (段式管理) -> 线性地址 -> (页式管理) -> 物理地址。由于linux系统实现中，每个段基址都是0，每个段的地址空间都是0～2^32-1，所以这里可忽略段机制，逻辑地址 = 线性地址。页式管理中，利用线性地址的高10位作为索引到页目录（每个进程都有页目录，保存在寄存器cr3中，进程切换时，cr3也会换成新进程的页目录地址）中找到一项，即为页表的地址；利用中间10位在页表中查找一项，其值（应该需要左移12位）加上现象地址的低12位，就得到物理地址。

​	![段页式管理机制](<https://raw.githubusercontent.com/hurley25/wiki/gh-pages/_posts/picture/chapt10/ADDR_TRAN.png>)

​	注意：

​	a. 上面介绍的是进入保护模式后的转换原理，实模式中，页式管理未开启。

​	b. 内核空间地址映射就是简单的线性映射，PAGE_OFFSET=0xc0000000，物理地址=逻辑地址-PAGE_OFFSET. 

#### [物理内存管理机制](<http://edsionte.com/techblog/archives/3937>)

##### [Buddy系统](<http://edsionte.com/techblog/archives/3763>)

​	伙伴算法主要解决物理内存外部碎片问题，基本思路：每个内存管理区中都有一个free_area数组，数组长度为11（可配置），每个元素结构如下：

```c
struct free_area {
        struct list_head        free_list[MIGRATE_TYPES]；
        unsigned long           nr_free;
};
```

​	每个元素是链表头，第0个元素链表中，包含的是2^0个页框，以此类推。

![](<http://edsionte.com/techblog/wordpress/wp-content/uploads/2012/03/buddy.jpeg>)

​	[分配与释放](<https://www.cnblogs.com/cherishui/p/4246133.html>)：例如现在仅存需要分配8K的物理内存，系统首先从8K那个链表中查询有无可分配的内存，若有直接分配；否则查找16K大小的链表，若有，首先将16K一分为二，将其中一个分配给进程，另一个插入8K的链表中，若无，继续查找32K，若有，首先把32K一分为二，其中一个16K大小的内存插入16K链表中，然后另一个16K继续一分为二，将其中一个插入8K的链表中，另一个分配给进程........以此类推。当内存释放时，查看相邻内存有无空闲，若存在两个联系的8K的空闲内存，直接合并成一个16K的内存，插入16K链表中。

​	深入理解，可参考[伙伴系统的极简实现](<https://coolshell.cn/articles/10427.html>)

##### [slab](<https://www.ibm.com/developerworks/cn/linux/l-linux-slab-allocator/>)

​	伙伴系统主要分配大块内存（页框为单位），解决外部碎片问题，而slab主要解决内部碎片。工程中常用的空闲队列与其思路很相似，但绝非空闲队列那么简单。

![](<https://www.ibm.com/developerworks/cn/linux/l-linux-slab-allocator/figure1.gif>)

​	如上图所示，每个slab由一个或多个page物理内存构成，每个slab是一个对象数组；每组slab称作一个cache，在这些cache之上，有一个称作cache_chain的链表，将这些cache连接起来。

​	slab提供kmalloc/kfree, kmem_cache_alloc/kmem_cache_free两组接口平面。kmem_cache_alloc这一组是专用的，例如mem_struct, inode这些结构进行过显示的创建cache，在分配时，就直接使用专用接口；也有时候，并未创建cache，但需要使用slab，就是调用kmalloc接口，其遍历cache_chain找到合适大小的空闲slab对象。

​	对比发现，单就slab对象数组这一层，跟空闲队列思想一样，单加上了cache_chain之后，就可以全局控制了，例如，调用kmalloc就可以直接分配到对象，而无需专门创建slab对象组（实际是占用其他cache了，最大化利用空闲队列）。

##### vmalloc

​	略。

#### malloc实现

​	参考[如何实现一个malloc](<http://blog.codinglabs.org/articles/a-malloc-tutorial.html>)

#### mmap

​	malloc的底层调用的是系统调用blk(), sblk(), mmap(), munmap()，当申请的内存大于128k时，就调用mmap()。参考[linux内存分配小节--malloc,blk,mmap](<https://blog.csdn.net/gfgdsg/article/details/42709943>)

​	之前只知道mmap做内存映射，将[文件映射到内存](<https://www.cnblogs.com/wanpengcoder/articles/5306688.html>)，而不知malloc的使用方式，可参考[malloc获取内存两种方式的区别](<http://netsecurity.51cto.com/art/201611/520641.htm>)

#### 一个进程占用多大内存？

### 进程、线程

#### 创建、销毁

#### 两者区别

#### 任务调度

#### 进程间同步

#### 互斥、死锁

#### 多线程题目

### 文件io

#### 标准io与系统调用

[图不错](<https://www.cnblogs.com/cherishui/p/4246822.html>)

#### io调度算法

### 网络io

#### 三握四挥

#### 收发包

### 中断、异常

#### signal实现原理