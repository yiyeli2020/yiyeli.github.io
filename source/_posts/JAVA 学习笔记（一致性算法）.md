---
title: JAVA 学习笔记（一致性算法）

date: 2022-01-06 17:12:12  

categories: 2022年1月

tags: [Java, Consistency]

---

Paxos，Zab，Raft，NWR和Gossip，一致性Hash算法

<!-- more -->

[TOC]


# Paxos
Paxos 算法解决的问题是一个分布式系统如何就某个值（决议）达成一致。一个典型的场景是，在一个分布式数据库系统中，如果各节点的初始状态一致，每个节点执行相同的操作序列，那么他们最后能得到一个一致的状态。为保证每个节点执行相同的命令序列，需要在每一条指令上执行一个“一致性算法”以保证每个节点看到的指令一致。zookeeper 使用的 zab 算法是该算法的一个实现。 在 Paxos 算法中，有三种角色：Proposer，Acceptor，Learners

## Paxos 三种角色：Proposer，Acceptor，Learners
### Proposer：
只要 Proposer 发的提案被半数以上 Acceptor 接受，Proposer 就认为该提案里的 value 被选定了。

### Acceptor：
只要 Acceptor 接受了某个提案，Acceptor 就认为该提案里的 value 被选定了。

### Learner：
Acceptor 告诉 Learner 哪个 value 被选定，Learner 就认为那个 value 被选定。

## Paxos 算法分为两个阶段。具体如下：
###  阶段一（准 leader 确定 ）：
(a) Proposer 选择一个提案编号 N，然后向半数以上的 Acceptor 发送编号为 N 的 Prepare 请求。

(b) 如果一个 Acceptor 收到一个编号为 N 的 Prepare 请求，且 N 大于该 Acceptor 已经响应过的所有Prepare请求的编号，那么它就会将它已经接受过的编号最大的提案（如果有的话）作为响应反馈给 Proposer，同时该 Acceptor 承诺不再接受任何编号小于 N 的提案。

### 阶段二（leader 确认）：
(a) 如果 Proposer 收到半数以上 Acceptor 对其发出的编号为 N 的 Prepare 请求的响应，那么它就会发送一个针对[N,V]提案的 Accept 请求给半数以上的 Acceptor。注意：V 就是收到的响应中编号最大的提案的 value，如果响应中不包含任何提案，那么 V 就由 Proposer 自己决定。

(b) 如果 Acceptor 收到一个针对编号为 N 的提案的 Accept 请求，只要该 Acceptor 没有对编号大于 N 的 Prepare 请求做出过响应，它就接受该提案

# Zab

 ZAB( ZooKeeper Atomic Broadcast , ZooKeeper 原子消息广播协议）协议包括两种基本的模式：崩溃恢复和消息广播

1. 当整个服务框架在启动过程中，或是当 Leader 服务器出现网络中断崩溃退出与重启等异常情况时，ZAB 就会进入恢复模式并选举产生新的 Leader 服务器。

2. 当选举产生了新的 Leader 服务器，同时集群中已经有过半的机器与该Leader服务器完成了状态同步之后，ZAB协议就会退出崩溃恢复模式，进入消息广播模式。

3. 当有新的服务器加入到集群中去，如果此时集群中已经存在一个 Leader 服务器在负责进行消息广播，那么新加入的服务器会自动进入数据恢复模式，找到 Leader 服务器，并与其进行数据同步，然后一起参与到消息广播流程中去。

以上其实大致经历了三个步骤：

1.崩溃恢复：主要就是 Leader 选举过程

2.数据同步：Leader 服务器与其他服务器进行数据同步

3.消息广播：Leader 服务器将数据发送给其他服务器

说明：zookeeper 章节对该协议有详细描述。

# Raft
与 Paxos 不同 Raft 强调的是易懂（Understandability），Raft 和 Paxos 一样只要保证 n/2+1节点正常就能够提供服务；raft把算法流程分为三个子问题：选举（Leader election）、日志复制（Log replication）、安全性（Safety）三个子问题。

## 角色
 Raft 把集群中的节点分为三种状态：Leader、 Follower 、Candidate，理所当然每种状态负责的任务也是不一样的，Raft 运行时提供服务的时候只存在 Leader 与 Follower 两种状态；
 
### Leader（领导者-日志管理）
负责日志的同步管理，处理来自客户端的请求，与 Follower 保持这 heartBeat 的联系；

### Follower（追随者-日志同步）
刚启动时所有节点为Follower状态，响应Leader的日志同步请求，响应Candidate的请求，把请求到 Follower 的事务转发给 Leader；

### Candidate（候选者-负责选票）
负责选举投票，Raft 刚启动时由一个节点从 Follower 转为 Candidate 发起选举，选举出Leader 后从 Candidate 转为 Leader 状态；

### Term（任期）

在 Raft 中使用了一个可以理解为周期（第几届、任期）的概念，用 Term 作为一个周期，每个 Term 都是一个连续递增的编号，每一轮选举都是一个 Term 周期，在一个 Term 中只能产生一个 Leader；当某节点收到的请求中 Term 比当前 Term 小时则拒绝该请求。

## 选举（Election）
### 选举定时器
Raft 的选举由定时器来触发，每个节点的选举定时器时间都是不一样的，开始时状态都为Follower

某个节点定时器触发选举后 Term 递增，状态由 Follower 转为 Candidate，向其他节点发起 RequestVote RPC 请求，这时候有三种可能的情况发生：
 
1：该 RequestVote 请求接收到 n/2+1（过半数）个节点的投票，从 Candidate 转为 Leader，向其他节点发送 heartBeat 以保持 Leader 的正常运转。
 
2：在此期间如果收到其他节点发送过来的 AppendEntries RPC 请求，如该节点的 Term 大则当前节点转为 Follower，否则保持 Candidate 拒绝该请求。
 
3：Election timeout 发生则 Term 递增，重新发起选举
 
在一个 Term 期间每个节点只能投票一次，所以当有多个 Candidate 存在时就会出现每个Candidate发起的选举都存在接收到的投票数都不过半的问题，这时每个 Candidate 都将 Term递增、重启定时器并重新发起选举，由于每个节点中定时器的时间都是随机的，所以就不会多次存在有多个 Candidate同时发起投票的问题。

在 Raft 中当接收到客户端的日志（事务请求）后先把该日志追加到本地的 Log 中，然后通过heartbeat 把该 Entry 同步给其他 Follower，Follower 
接收到日志后记录日志然后向 Leader 发送ACK，当 Leader 收到大多数（n/2+1）Follower 的 ACK 信息后将该日志设置为已提交并追加到本地磁盘中，通知客户端并在下个 heartbeat 中 Leader 将通知所有的 Follower 将该日志存储在自己的本地磁盘中。

## 安全性（Safety）
安全性是用于保证每个节点都执行相同序列的安全机制如当某个 Follower 在当前 Leader commit Log 时变得不可用了，稍后可能该 Follower 又会被选举为 Leader，这时新 Leader 可能会用新的Log 覆盖先前已 committed 的 Log，这就是导致节点执行不同序列；Safety 就是用于保证选举出来的 Leader 一定包含先前 commited Log 的机制；

选举安全性（Election Safety）：每个 Term 只能选举出一个 Leader

Leader 完整性（Leader Completeness）：这里所说的完整性是指 Leader 日志的完整性，Raft 在选举阶段就使用 Term的判断用于保证完整性：当请求投票的该 Candidate 的 Term 较大或 Term 相同 Index 更大则投票，该节点将容易变成 leader。


## raft 协议和 zab 协议区别
### 相同点

 采用 quorum 来确定整个系统的一致性,这个 quorum 一般实现是集群中半数以上的服务器,

 zookeeper 里还提供了带权重的 quorum 实现.

 都由 leader 来发起写操作.

 都采用心跳检测存活性

leader election 都采用先到先得的投票方式

### 不同点

 zab 用的是 epoch 和 count 的组合来唯一表示一个值, 而 raft 用的是 term 和 index

 zab 的 follower 在投票给一个 leader 之前必须和 leader 的日志达成一致,而 raft 的 follower则简单地说是谁的 term 高就投票给谁

 raft 协议的心跳是从 leader 到 follower, 而 zab 协议则相反

 raft 协议数据只有单向地从 leader 到 follower(成为 leader 的条件之一就是拥有最新的 log)

而 zab 协议在 discovery 阶段, 一个 prospective leader 需要将自己的 log 更新为 quorum 里面最新的 log,然后才好在 synchronization 阶段将 quorum 里的其他机器的 log 都同步到一致.

# NWR
N：在分布式存储系统中，有多少份备份数据

W：代表一次成功的更新操作要求至少有 w 份数据写入成功

R： 代表一次成功的读数据操作要求至少有 R 份数据成功读取

NWR值的不同组合会产生不同的一致性效果，当W+R>N 的时候，整个系统对于客户端来讲能保证强一致性。而如果 R+W<=N，则无法保证数据的强一致性。以常见的 N=3、W=2、R=2 为例：

N=3，表示，任何一个对象都必须有三个副本（Replica），W=2 表示，对数据的修改操作（Write）只需要在 3 个 Replica 中的 2 个上面完成就返回，R=2 表示，从三个对象中要读取到 2个数据对象，才能返回。

如果R+W>N,则读取操作和写入操作成功的数据一定会有交集（如图中的Node2），这样就可以保证一定能够读取到最新版本的更新数据，数据的强一致性得到了保证。在满足数据一致性协议的前提下，R或者W设置的越大，则系统延迟越大，因为这取决于最慢的那份备份数据的响应时间。

当R+W<=N，无法保证数据的强一致性

因为成功写和成功读集合可能不存在交集，这样读操作无法读取到最新的更新数值，也就无法保证数据的强一致性。


# Gossip
Gossip 算法又被称为反熵（Anti-Entropy），熵是物理学上的一个概念，代表杂乱无章，而反熵就是在杂乱无章中寻求一致，这充分说明了 Gossip 的特点：在一个有界网络中，每个节点都随机地与其他节点通信，经过一番杂乱无章的通信，最终所有节点的状态都会达成一致。每个节点可能知道所有其他节点，也可能仅知道几个邻居节点，只要这些节可以通过网络连通，最终他们的状态都是一致的，当然这也是疫情传播的特点。

# 一致性 Hash
一致性哈希算法(Consistent Hashing Algorithm)是一种分布式算法，常用于负载均衡。Memcached client 也选择这种算法，解决将 key-value 均匀分配到众多 Memcached server 上的问题。

它可以取代传统的取模操作，解决了取模操作无法应对增删 Memcached Server 的问题(增删 server 会导致同一个 key,在 get 操作时分配不到数据真正存储的 server，命中率会急剧下降)。

## 一致性 Hash 特性
 平衡性(Balance)：平衡性是指哈希的结果能够尽可能分布到所有的缓冲中去，这样可以使得所有的缓冲空间都得到利用。

 单调性(Monotonicity)：单调性是指如果已经有一些内容通过哈希分派到了相应的缓冲中，又有新的缓冲加入到系统中。哈希的结果应能够保证原有已分配的内容可以被映射到新的缓冲中去，而不会被映射到旧的缓冲集合中的其他缓冲区。容易看到，上面的简单求余算法hash(object)%N 难以满足单调性要求。

 平滑性(Smoothness)：平滑性是指缓存服务器的数目平滑改变和缓存对象的平滑改变是一致的。

## 一致性 Hash 原理
### 建构环形 hash 空间：
1. 考虑通常的 hash 算法都是将 value 映射到一个 32 为的 key 值，也即是 0~2^32-1 次方的数值空间；我们可以将这个空间想象成一个首（ 0 ）尾（ 2^32-1 ）相接的圆环。

### 把需要缓存的内容(对象)映射到 hash 空间
2. 接下来考虑 4 个对象 object1~object4 ，通过 hash 函数计算出的 hash 值 key 在环上的分布

### 把服务器(节点)映射到 hash 空间
3. Consistent hashing 的基本思想就是将对象和 cache 都映射到同一个 hash 数值空间中，并且使用相同的 hash 算法。一般的方法可以使用 服务器(节点) 机器的 IP 地址或者机器名作为hash 输入。

### 把对象映射到服务节点
4. 现在服务节点和对象都已经通过同一个 hash 算法映射到 hash 数值空间中了，首先确定对象hash值在环上的位置，从此位置沿环顺时针“行走”，第一台遇到的服务器就是其应该定位到的服务器

### 考察 cache 的变动
5. 通过 hash 然后求余的方法带来的最大问题就在于不能满足单调性，当 cache 有所变动时，cache 会失效。

5.1 移除 cache：考虑假设 cache B 挂掉了，根据上面讲到的映射方法，这时受影响的将仅是那些沿 cache B 逆时针遍历直到下一个 cache （ cache C ）之间的对象。

5.2 添加 cache：再考虑添加一台新的 cache D 的情况，这时受影响的将仅是那些沿 cacheD 逆时针遍历直到下一个 cache 之间的对象，将这些对象重新映射到 cache D 上即可。

### 虚拟节点
hash 算法并不是保证绝对的平衡，如果 cache 较少的话，对象并不能被均匀的映射到 cache 上，为了解决这种情况， consistent hashing 引入了“虚拟节点”的概念，它可以如下定义：

虚拟节点（ virtual node ）是实际节点在 hash 空间的复制品（ replica ），一实际个节点对应了若干个“虚拟节点”，这个对应个数也成为“复制个数”，“虚拟节点”在 hash 空间中以 hash值排列。

仍以仅部署 cache A 和 cache C 的情况为例。现在我们引入虚拟节点，并设置“复制个数”为 2 ，这就意味着一共会存在 4 个“虚拟节点”， cache A1, cache A2 代表了 cache A； cache C1,cache C2 代表了 cache C 。此时，对象到“虚拟节点”的映射关系为：

    objec1->cache A2 ； objec2->cache A1 ； objec3->cache C1 ； objec4->cache C2 ；

因此对象 object1 和 object2 都被映射到了 cache A 上，而 object3 和 object4 映射到了 cache C 上；平衡性有了很大提高。

引入“虚拟节点”后，映射关系就从 { 对象 -> 节点 } 转换到了 { 对象 -> 虚拟节点 } 。查询物体所在 cache 时的映射关系如下图 所示。












