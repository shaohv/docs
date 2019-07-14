<center>Linux知识要点</center>

---

### 内存管理

#### 页式内存

##### [进程的虚拟地址空间](<http://0xffffff.org/2013/05/23/18-linux-process-address-space/>)	

![process virtual adress space](<https://github.com/shaohv/shaohv.resources/raw/master/linux_os_address_space.png>)

​	如上图所示，地址空间被分成内核空间和用户空间，应用程序通过调用系统调用进入内核，内核中通过copy_from_user将数据从用户空间拷贝到内核空间（同样还有copy_to_user），该过程参考[关于系统调用sethostname的描述](<https://www.cnblogs.com/wuchanming/p/4360277.html>)。以上是进程运行时使用的地址，实际cpu执行指令时，也是用的这个地址，称作逻辑地址。

##### 地址映射

​	用户空间：如下图一句话，逻辑地址 -> (段式管理) -> 线性地址 -> (页式管理) -> 物理地址。由于linux系统实现中，每个段基址都是0，每个段的地址空间都是0～2^32-1，所以这里可忽略段机制，逻辑地址 = 线性地址。页式管理中，利用线性地址的高10位作为索引到页目录（每个进程都有页目录，保存在寄存器cr3中，进程切换时，cr3也会换成新进程的页目录地址）中找到一项，即为页表的地址；利用中间10位在页表中查找一项，其值（应该需要左移12位）加上现象地址的低12位，就得到物理地址。​	![段页式管理机制](<https://github.com/shaohv/shaohv.resources/raw/master/linux_os_address_mapping.png>)

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

![](<https://github.com/shaohv/shaohv.resources/raw/master/linux_os_buddy.png>)

​	[分配与释放](<https://www.cnblogs.com/cherishui/p/4246133.html>)：例如现在仅存需要分配8K的物理内存，系统首先从8K那个链表中查询有无可分配的内存，若有直接分配；否则查找16K大小的链表，若有，首先将16K一分为二，将其中一个分配给进程，另一个插入8K的链表中，若无，继续查找32K，若有，首先把32K一分为二，其中一个16K大小的内存插入16K链表中，然后另一个16K继续一分为二，将其中一个插入8K的链表中，另一个分配给进程........以此类推。当内存释放时，查看相邻内存有无空闲，若存在两个联系的8K的空闲内存，直接合并成一个16K的内存，插入16K链表中。

​	深入理解，可参考[伙伴系统的极简实现](<https://coolshell.cn/articles/10427.html>)

##### [slab](<https://www.ibm.com/developerworks/cn/linux/l-linux-slab-allocator/>)

​	伙伴系统主要分配大块内存（页框为单位），解决外部碎片问题，而slab主要解决内部碎片。工程中常用的空闲队列与其思路很相似，但绝非空闲队列那么简单。

![image](<https://github.com/shaohv/shaohv.resources/raw/master/linux_os_slab.png>)

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

#### 进程创建、退出

**fork:** linux通过系统调用*clone()*实现了多个库函数，包括fork(),vfork()和clone()，它们的区别在于传递不同的标志给*clone()*。该系统调用主要完成下列四件事情：

1. 分配内核数据结构给新的进程，如thread info；

2. 拷贝父进程的部分数据结构内容至子进程，例如堆栈；

3. 将子进程添加到任务列表中；

4. 返回。

   注意：为什么fork会在父子进程中各返回一次？

**execve:** 加载并解析可执行文件，修改进程的代码段、数据段，并修改进程的堆栈，确保进程接下来执行新的程序逻辑。

​	源码分析，可参见[fork+execve:一个进程的诞生](<https://blog.csdn.net/chengonghao/article/details/51334663>)

**退出：** 进程退出有exit和_exit两个函数，前者比后者多做一些事情，例如将文件缓冲区写入文件等。c语言main函数return出去的值就是exit的参数。

![image](<https://github.com/shaohv/shaohv.resources/raw/master/linux_os_exit.png>)

了解这些库函数如何使用，可以阅读[实现一个简单的shell](https://www.cnblogs.com/tp-16b/p/9096838.html)。

更多理论分析，参见[linux进程管理剖析](<https://www.ibm.com/developerworks/cn/linux/l-linux-process-management/>)，[Linux内核设计与实现-进程管理](<https://www.cnblogs.com/20135235my/p/5349711.html>)。

#### 两者区别

​	参见[linux进程与线程的区别](https://www.cnblogs.com/fah936861121/articles/8043187.html)。

#### 任务调度

​	包含三方面：**进程状态机、上下文切换和调度算法**。

##### 状态机

​	通俗：运行态、就绪、阻塞

​	详细：running(运行、就绪) 。。。 

##### 调度算法

​	[linux系统调度策略比较](https://blog.csdn.net/fchyang/article/details/79268204) , [linux完全公平调度粗略](http://c.biancheng.net/view/1255.html)

​	CFS是不支持优先级的，其调度的依据是虚拟运行时间。cpu密集型任务更加倾向于用完分配到的时间片，而io密集型任务经常会因阻塞而放弃cpu，而给其他任务使用，这样的情形下，io密集型任务就是更加友好的。虚拟运行时间计算与任务的友好值反相关，这样有好值越高，其虚拟运行时间越低。

​	cfs有好值是[-20, 19]，而linux系统实时策略（fifo, rr）的优先级是[0，99]。所有任务在一起调度时，会将cfs映射到[99,139]。

​	linux目前默认的时候CFS(Completely Fair Schedual)，即我们使用pthread_create不指定policy时，创建出来的线程就是CFS的。

​	



#### 同步、互斥

​	进程间同步方式有：共享内存、消息队列、管道、信号量和socket等。

​	线程间同步方式除上述几种外，还有条件变量、自旋锁、互斥锁、读写锁。

​	同步与互斥有一定区别，前者主要用来控制时序，让多个进程或线程按照一定的逻辑来执行；互斥主要用来保护临界区，禁止多个线程同时操作。

​	我接触到的项目中，条件变量和锁的应用比较多，例如进程启动时候，要做一些初始化，包括起一些业务线程等，这些业务线程创建出来之后并非立即执行，而是等待条件变量，进程在启动的最后阶段（一切初始化完成后），置上条件变量，各业务线程开始处理业务；另外，自旋锁、读写锁之类的，在多io业务线程进程用到。

​	锁的实现中，一般都需要硬件支持，例如spin lock的实现，需要原子操作test_and_set来支持，基本逻辑如下:

```c
void spin_lock(int* lock_val){
    int old_val = 0;
    while((old_val = test_and_set(*lock_val,1) == 1)); //lock_val为0，表示lock没有空闲
    return;
}

void spin_unlock(int* lock_val){
    *lock_val = 0;
    return;
}
```

​	上面的代码在获取不到锁时，会让cpu一直空转，实际实现中，例如arm上，有一对ldrex,strex操作，ldrex将数据加载到寄存器中，strex将数据保存到内存中，在strex保存时，如果内存中，被其他执行路径（线程）改变过，将返回错误，参见[linux同步机制spin lock](<http://www.wowotech.net/kernel_synchronization/spinlock.html>)。

​	更多关于锁的讨论，参见[Linux Futex浅析](<https://blog.csdn.net/mitushutong11/article/details/51336136>)，[Linux Futex的设计与实现](<https://blog.csdn.net/jianchaolv/article/details/7544316>)。

##### 利用mutex实现读写锁

​	假设mutex锁为mutex_lock, mutex_unlock

```c
read_count = 0
mutex_t write_mutex, read_count_mutex;

void read_mutex_lock(){
    mutex_lock(read_count_mutex);
    if(read_count == 0){
        read_count ++;
        mutex_lock(write_mutex);
    }
    mutex_unlock(read_count_mutex);
}

void read_mutex_unlock(){
    mutex_lock(read_count_mutex);
    read_count--;
    if(read_count == 0){
        mutex_unlock(write_mutex);
    }
    mutex_unlock(read_count_mutex);
}

void write_mutex_lock(){
    mutex_lock(write_mutex);
}

void write_mutex_unlock(){
    mutex_unlock(write_mutex);
}
```

#### 多线程题目

#### 进程如何后台化

### 文件io

#### 标准io与系统调用

几篇参考文章：

源码分析参考：[Linux VFS与Read/Write系统调用](<https://www.cnblogs.com/Zachary-Fan/p/dborcachefirst.html>)，[linux read 系统调用剖析](https://www.cnblogs.com/tcicy/p/8454740.html)。

理论分析：[漫谈linux文件IO](https://www.cnblogs.com/alantu2018/p/8460061.html)

文件操作需注意，缓存io与direct io的处理区别。

另外，尽量结合自己的项目去理解文件io各个层面。

#### 管道实现

<https://www.cnblogs.com/tp-16b/p/8886378.html>

### 网络

#### 网络io模型

[网络IO模型：同步IO和异步IO，阻塞IO和非阻塞IO](http://blog.chinaunix.net/uid-28458801-id-4464639.html)

[linux下的io复用与epoll详解](<https://www.cnblogs.com/lojunren/p/3856290.html>)

​	epoll机制在工程实践中用的非常多，这里结合项目仔细体会一下。

#### linux网络栈

[理解linux网络栈](<https://www.cnblogs.com/sammyliu/p/5225623.html>)

#### 五层模型

osi七层模型，tcp/ip五层模型（物理层、链路层、网络层、传输层和应用层）。

#### 三握四挥

[协议森林](<https://www.cnblogs.com/vamei/archive/2012/12/05/2802811.html>)

三握四挥遇到**异常**怎么处理？例如，三次握手第三次丢包。

[三次握手异常情况](<https://www.cnblogs.com/quehualin/p/10409607.html>)，[参考2](<https://www.jianshu.com/p/29868fb82890>)

#### 收发包

参考《linux内核源代码情景分析》的**基于socket的进程间通信**。

#### http/https如何工作的

#### 一次完整的通信

### 中断、异常

#### signal实现原理