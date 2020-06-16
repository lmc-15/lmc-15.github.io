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

### 数据一致性

- 如何保证顺序一致性
  - 由于每次更新数据都会将*zxid*递增，所以用*zxid*来保证顺序一致，每次读取读最大的*zxid*即可
  - 如果网络重连时，如果发现`client`的*zxid*比`server`小，则连接失败。

### leader选举的原理

- 重要参数
  - 服务器ID(myid),编号越大值权重越大
  - *zxid*事务id，值越大数据越新，权重越大

#### 1、服务器启动时leader选举

- 节点启动时都是LOOKING，处于观望状态。

- 每个`server`发出投票，每次投票通过(myid,zxid,epoch)来发送给集群中的其他服务

- 接收服务器投票，判断票据是否有效，是否是本次投票的（epoch），是否来自LOOKING状态的服务器

  -  Epoch：逻辑时钟：或者叫投票的次数，同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加，然后与接收到的其它服务器返回的投票信息中的数值相比，根据不同的值做出不同的判断。

- 处理投票进行PK，PK优先级：
  - 优先比较epoch
  - 再次比较zxid
  - 最后比较myid。（**以上都是较大的胜出**）
  
- 统计票数，超过过半机器票数的胜出，位Leader

- 改变服务状态，一旦`Leader`确定后，其他服务器更新自己的状态位`follower`

#### 2、运行过程中leader选举

- 确定`leader`挂了之后，所有`follower`会将自己的服务器状态变为LOOKING,然后进入leader选举过程
- 接下来操作与服务器启动差不多

### zookeeper的Watcher实现原理

- Watcher 机制，总的来说可以分为三个过程：**客户端注册 Watcher**、**服务器处理 Watcher** 和**客户端回调 Watcher**
- 在创建一个 ZooKeeper 客户端对象实例时，可以向构造方法中传入一个默认的 Watcher：
  - 这个 Watcher 将作为整个 ZooKeeper会话期间的默认 Watcher，会一直被保存在**客户端 ZKWatchManager** 的 **defaultWatcher** 中。
  - ZooKeeper 客户端也可以通过 **getData**、**exists** 和 **getChildren** 三个接口来向 ZooKeeper 服务器注册 Watcher
- 一次性
  - 无论服务端还是客户端，**Watcher**被触发，zk会将器清除，所以**Watcher**需要被反复的注册
- 轻量级
  - WatchedEvent 是最小通知单元（通知状态，事件类型，节点路径）
  - 服务端仅仅通知客户端发生了事件，不带具体内容
  - 客户端像服务端传的不是watcher，用boolean类似标记属性，服务端保存当前的连接的SercverCnxn
- **ClientCnxn**:是**Zookeeper **客户端和服务端进行通信和事件通知的主要类。
  - **SendThread**：负责客户端和服务器端的数据通信, 也包括事件信息的传输
  - **EventThread**：主要在客户端回调注册的 Watchers 进行通知处理
- 首先要有一个main()线程,在main线程中创建Zookeeper客户端，这时就会创建两个线程，一个负责网络连接通信(connet)，一个负责监听(listener)。
- 通过connect线程将注册的监听事件发送给Zookeeper服务端。
- 在Zookeeper服务端的注册监听器列表中将注册的监听事件添加到列表中。
- Zookeeper监听到有数据或路径变化，就会将这个消息发送给listener线程。

### CAP

- C——数据一致性，A——服务可用性，P——服务容错性

- zookeeper通过zab保证数据一致性，多个服务保证服务容错性，如果大于一半节点挂了则会导致网络不可用不能保证服务可用性
- Eureka Clients上也会缓存服务调用的信息。这就保证了我们微服务之间的互相调用是足够健壮的。针对同一个服务，即使注册中心的不同节点保存的服务提供者信息不尽相同，也并不会造成灾难性的后果。拿到可能不正确的服务实例信息后尝试消费一下。
- 所以，对于服务发现而言，可用性比数据一致性更加重要——AP胜过CP。





