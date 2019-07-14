linux后端

---

### epoll

[Linux下的I/O复用与epoll详解](https://www.cnblogs.com/lojunren/p/3856290.html)。

### 分布式

#### 一致性协议

Pacifica, raft, paxos( zab? )

#### 一致性hash

​	为什么叫一致性hash? [参考](http://itindex.net/detail/48809-%E4%B8%80%E8%87%B4%E6%80%A7-hash)
	[这篇也不错](https://www.cnblogs.com/moonandstar08/p/5405991.html)
#### 分布式事务

​	2pc, rocketMQ

#### 分布式锁

​	zookeeper中zk-client可以通过监听临时znode，例如/dsware/master_mdc，如果该znode不存在，zk-client就都尝试往zookeeper中添加zone，写入成功的就算是拿锁成功；zk-client主动释放锁就将临时znode删除；如果是持有锁的zk-client挂掉，该znode也会自动删除（临时znode基于zk-client和zk-server之间的session）。

​	[故障场景参考](<https://blog.csdn.net/qiangcuo6087/article/details/79067136>)

#### zookeeper/etcd

​	[zk](<https://www.cnblogs.com/felixzh/p/5869212.html>)

​	[etcd](<https://www.infoq.cn/article/etcd-interpretation-application-scenario-implement-principle>)

#### 分布式缓存

#### 理论

​	

 

