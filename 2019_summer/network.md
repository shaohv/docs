<center>网络编程</center>
### OSI七层模型与TCP/IP四层模型

| osi        | tcp/ip     |
| ---------- | ---------- |
| 应用层     | /          |
| 表示层     | /          |
| 会话层     | 应用层     |
| 传输层     | 传输层     |
| 网络层     | 网络层     |
| 数据链路层 | 物理链路层 |
| 物理层     | /          |

​		常见各层协议：

​			数据链路层: wifi, 以太网

​			网络层: ip, icmp, arp, rarp （后面两者介于数据链路层和网络层之间）

​			传输层：udp/ tcp

​			应用层: http

#### 一次路由过程

 1. 局域网内部 machine A  -> machine B

    ​	应用层协议，如http请求经过层层封装到达网络层，此时报文中含有src ip/port, dst ip/port；

    ​	网络层利用dst ip在自己路由表中对每个条目，拿mask计算dst 子网ip，计算得到就在当前lan中；

    ​	利用本地缓存的arp表查找mac地址，如果没找到，使用arp协议，询问当前lan中各node，得到dst mac，构造物理链路层包发出去；

    ​	machine B收到包后，比对dst mac与自己一致，进行解析向上层传递。

    FAQ：lan中所有node都会收到链路层包吗？

    ​	如果lan中有交换机，交换机能隔离冲突域

 2. 跨局域网  machine A -> machine B

    ​	对比lan内情况，计算dst 子网ip后，发给对应网口，注意此时链路层包中的dst mac不是dst ip对应的mac，而是下一跳router的mac。

### 链路接入层

#### arp工作场景

#### rarp出现场景

### 网络层

#### ip协议

#### icmp协议

	1. ping的实现
 	2. traceroute的实现

### 传输层

#### tcp

##### 三次握手

##### 四次挥手

##### 滑动窗口

##### 拥塞窗口

##### 快重传

#### udp

#### 两者区别

### 应用层

#### http

### 基于TCP的socket编程

​		重点：

​				listen, connect 和 accept的行为

​				read,write 阻塞非阻塞正常读写行为（针对send buff, recv buff情况）

##### tcp关闭行为（close,shutdown,so_linger）

​	shutdown是关闭连接，close是关闭socket，socket关闭后应用层就不能调用读写接口了（如果不能read了，半关闭的socket场景，对端发送的数据将很快占满本端recv buff，导致对端滑窗为0 **??**）。

​	两者正常都是发送fin，进入fin_wait_1，异常发送rst，直接关闭连接结束（不等待对端ack）。

​	shutdown比较简单

  1. 关闭的是conn socket，如果recv buff有数据，则发送rst；如果无数据，将则send buff中最后报文上追加fin，发送给对端。

  2. 关闭的是listen socket，将对syn队列的sock发送rst.

     close without so_linger

     ​	行为同shutdown，不过close函数直接返回，kernal的动作是异步的。

     close with so_linger & 等待时常不为0

     ​	close会阻塞进程，等待对端的fin ack，直到超时

​	close with  so_linger & 等待时常为0

​			直接发送rst

[高性能网络编程4--TCP连接的关闭](https://blog.csdn.net/russell_tao/article/details/13092727)。



##### read, write在异常场景行为

​				

#### 接口

1. server

   socket ：socket在文件系统中生成了相应的结构，返回了fd

   bind ：跟端口绑定

   listen : listen socket，此时client就可以connect了;client connect触发syn，此时listen状态下，可以syn +  ack；再收到client的ack之前，产生的conn socket都呆在未连接队列中，如果收到client ack后，则将conn socket放入已连接队列

   accept ：从已连接socket队列取出一个socket，如果已连接队列为空，将阻塞

   read/recv

   write/send

2. client

   socket

   bind（可省略）

   connect：收到server的syn + ack后，返回成功；如果未能及时收到syn ack，connect可能返回EINPROgreSS;返回einprogress并不表示失败，可以用select/epoll来监听socket可写，然后再调用getsocket

   The socket is nonblocking and the connection cannot be completed immediately. It is possible to **select**(2) **or** poll(2) **for** completion **by** selecting the socket **for** writing. **After** **select**(2) indicates writability, **use** getsockopt(2) **to** **read** the SO_ERROR **option** **at** **level** SOL_SOCKET **to** determine whether **connect**() completed successfully (SO_ERROR **is** zero) **or** unsuccessfully (SO_ERROR **is** one **of** the usual **error** codes listed here, explaining the reason **for** the **failure**).

   read:  nread = read(sock_fd, buf, bufflen)

   nread > 0 正常读到数据

   nread = 0  读到fin，对端关闭

   nread < 0 ，徐判断errno，EINTER,EWOUDBLOCK,EAGAIN都是正常的

   无论是阻塞还是非阻塞，只要rcv buff不为空，read立刻返回；

   rcv buff为空，阻塞socket将block；非阻塞socket将返回-1,error = EAGAIN/EWOULDBLOCK

   write：

   阻塞：直至send buff能放下所有数据，才写入返回；特例，如果write阻塞时，对端关闭socket，write将能拷多少烤多少。

   非阻塞：能写多少写多少，如果没有写完，返EWOULDBLOCK, EAGAIN

##### 读写异常处理 

​	关闭的行为可参考[高性能网络编程4--TCP连接的关闭](https://blog.csdn.net/russell_tao/article/details/13092727)。关闭无论是shutdown还是close，无论close是否设置so_linger，在对端看来行为无非两种，收到了fin，收到了rst；向对端发送rst无需等待对端响应直接关闭socket。

 1. 对端正常关闭，发送fin

    kernal会对fin响应ack；此时本端处于close wait状态，连接处于半关闭，对本端write 无影响；；如果阻塞在read，read将返回0；**如果没有阻塞在read，下一次read还会返回0吗**

    如果对端发送过来的fin是os代劳，对端进程已经不存在，那么此时在本端socket写会收到rst。

 2. 对端异常关闭，发送rst

    本端读写应都能感知到 ??

    理论上如果读写正阻塞着，收到rst后，应该收到econnrst错误；如果没有阻塞在read/write，下一次read/write时，将收到econnrst错误；（**自己想的未验证**）

 3. 对端掉电或网络不可达

    如果阻塞在read将永远阻塞，如果是nonblock，则永远不会触发可读事件；如果对socket写，本次写将正常返回，但kernal在发送send buff中的数据时，可能很久都等不到ack，将socket置ETIMEOUT或不可达；下一次写（**读呢**）将收到该错误；

 4. 对端掉电恢复

    本端读感知不到，本端写第一次将收到rst，即ECONNRST，再次写将收到EPIPE。

 5. 本端关闭

    

6. 公共

   setsocket: so_linger

   getsocket so_error，将检查struct sock.sk_err，将其清0并返回。

#### socket行为

​	1. 三次握手的发生时机

