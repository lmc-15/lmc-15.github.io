### 实现分布式锁

- 通过有序临时节点实现分布式锁
- 每个客户端往指定节点注册临时节点
- 节点编号最小的获取锁
- 最小节点删除后，通过watcher事件再次将编号最小的节点获取锁（避免羊群效应）

### Zookeeper实现leader选举

- 类似分布式锁 不一一解释

### Zookeeper数据同步流程

- 如果是读请求，直接从当前节点读取数据
- 如果是写请求，将请求转发给**leader**提交事务
- **leader**会广播事务，超过半数节点写入成功

### ZAB协议

​	`zookeeper`主要依赖**ZAB**协议来实现分布式数据一致性，`zookeeper`实现了一种主备模式的系统架构来保持集群中各个副本数据的一致性

- 消息广播实现原理
  - **leader**接收到消息后，将消息赋予一个全局ID（zxid）
  - **leader**为每一个**follwer**准备一个**FIFO**队列，将带`zxid`分发给所有**follwer**
  - **follwer**接收到消息，把消息写入磁盘，向**leader**回复一个ack
  - **leader**收到超过半数节点的ack后，**leader**会向**follwer**发送`commit`命令，同时在本地执行该消息
  - **follwer**接收到`commit`命令后，提交该消息
- 崩溃恢复的实现原理
  - **leader**失去过半的**follwer**的联系将进入崩溃恢复
  - 已经被处理的消息不能丢
  - 被丢弃的不能再次出现
  - **leader**选举算法保证**leader**服务器上的`zxid`最大
  - `zxid`64位，高32位`epoch`编号，每选一次新的**leader**，新的**leader**会把`epoch`+1,低32消息计数器，没接收到一条消息这个值+1，新的**leader**出现后重置0，老**leader**重启后不会被选为**leader**因为`zxid`肯定小于当前**leader**，新**leader**将未被commit的消息清除







