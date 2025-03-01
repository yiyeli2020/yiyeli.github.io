---
title: Zookeeper学习笔记（概念，协议，投票机制）

date: 2021-12-05 16:12:12

categories: 2021年12月

tags: [Linux, Zookeeper]

---

Zookeeper （动物园管理员）是一个分布式协调服务，可用于服务发现，分布式锁，分布式领导选举，配置管理等。

<!-- more -->


[TOC]

摘自《Java高并发核心编程卷1：NIO、Netty、Redis、ZooKeeper》1.3　分布式利器ZooKeeper [^1]

# 什么是ZooKeeper

最早起源于雅虎公司研究院的一个研究小组。当时，研究人员发现，在雅虎内部很多大型的系统需要依赖一个类似的系统进行分布式协调，但是这些系统往往存在分布式单点问题，所以雅虎的开发人员就试图开发一个通用的无单点问题的分布式协调框架。

此框架的命名过程也是非常有趣的。在项目初期给这个项目命名时，准备和很多项目一样，按照雅虎公司的惯例使用动物的名字来命名（例如著名的Pig项目）。在探讨取什么名字的时候，研究院的首席科学家Raghu Ramakrishnan开玩笑说：“再这样下去，我们这儿就变成动物园了。”此话一出，大家纷纷表示新框架就叫动物园管理员吧，于是ZooKeeper（动物园管理员）诞生了。而ZooKeeper正好是用来协调分布式环境的不同节点的，形象地说，可以理解为协调各个以动物命名的分布式组件，所以ZooKeeper也就“名副其实”了。

# ZooKeeper的优势

ZooKeeper的核心优势是实现了分布式环境的数据一致性，简单地说：每时每刻我们访问ZooKeeper的树结构时，不同的节点返回的数据都是一致的。也就是说，对ZooKeeper进行数据访问时，无论是什么时间，都不会引起“脏读”“幻读”“不可重复读”问题。

“脏读”“幻读”“不可重复读”是数据库事务的概念，当然，ZooKeeper也可以被理解为一种简单的分布式数据库。“脏读”是指一个事务中访问到了另外一个事务未提交的数据。“不可重复读”是指在一个事务内根据同一个条件对数据进行多次查询，但是结果却不一致，原因是其他事务对该数据进行了修改。“幻读”是指当两个完全相同的查询执行时，第二次查询所返回的结果集和第一次查询所返回的结果集不相同，原因也是另外一个事务新增、删除了第一个事务结果集中的数据。

### “不可重复读”和“幻读”的区别

是：“不可重复读”关注的重点在于记录的更新操作，对同样的记录，再次读取后发现返回的数据值不一样了；“幻读”关注的重点在于记录新增或者删除操作（数据条数发生了变化），同样的条件第一次和第二次查询出来的记录数不一样。

ZooKeeper对不同系统环境的支持都很好，在绝大多数主流的操作系统上都能够正常运行，如GNU/Linux、Sun Solaris、Win32以及MacOS等。但是，ZooKeeper官方文档中特别强调，由于FreeBSD系统的JVM实现对Java的NIOSelector（选择器）支持得不是很好，因此不建议在FreeBSD系统上部署ZooKeeper生产服务器。可以说，ZooKeeper提供的是分布式系统中非常底层且必不可少的基本功能，如果开发者自己来实现这些功能且达到高吞吐、低延迟，同时还要保持一致性和可用性，实际上是非常困难的。借助ZooKeeper提供的这些功能，开发者就可以轻松地在ZooKeeper之上构建自己的各种分布式系统。

# Zookeeper 概念
Zookeeper 是一个分布式协调服务，可用于服务发现，分布式锁，分布式领导选举，配置管理等。
Zookeeper 提供了一个类似于 Linux 文件系统的树形结构（可认为是轻量级的内存文件系统，但
只适合存少量信息，完全不适合存储大量文件或者大文件），同时提供了对于每个节点的监控与
通知机制。
# Zookeeper 角色
Zookeeper 集群是一个基于主从复制的高可用集群，每个服务器承担如下三种角色中的一种
## Leader

1. 一个 Zookeeper 集群同一时间只会有一个实际工作的 Leader，它会发起并维护与各 Follwer
及 Observer 间的心跳。
2. 所有的写操作必须要通过 Leader 完成再由 Leader 将写操作广播给其它服务器。只要有超过
半数节点（不包括 observeer 节点）写入成功，该写请求就会被提交（类 2PC 协议）。
## Follower
1. 一个 Zookeeper 集群可能同时存在多个 Follower，它会响应 Leader 的心跳，
2. Follower 可直接处理并返回客户端的读请求，同时会将写请求转发给 Leader 处理，
3. 并且负责在 Leader 处理写请求时对请求进行投票。
## Observer
角色与 Follower 类似，但是无投票权。Zookeeper 需保证高可用和强一致性，为了支持更多的客
户端，需要增加更多 Server；Server 增多，投票阶段延迟增大，影响性能；引入 Observer，
Observer 不参与投票； Observers 接受客户端的连接，并将写请求转发给 leader 节点； 加入更
多 Observer 节点，提高伸缩性，同时不影响吞吐率。

# ZAB 协议
## 事务编号 Zxid（事务请求计数器+ epoch）
在 ZAB ( ZooKeeper Atomic Broadcast , ZooKeeper 原子消息广播协议） 协议的事务编号 Zxid
设计中，Zxid 是一个 64 位的数字，其中低 32 位是一个简单的单调递增的计数器，针对客户端每
一个事务请求，计数器加 1；而高 32 位则代表 Leader 周期 epoch 的编号，每个当选产生一个新
的 Leader 服务器，就会从这个 Leader 服务器上取出其本地日志中最大事务的 ZXID，并从中读取
epoch 值，然后加 1，以此作为新的 epoch，并将低 32 位从 0 开始计数。
Zxid（Transaction id）类似于 RDBMS 中的事务 ID，用于标识一次更新操作的 Proposal（提议）
ID。为了保证顺序性，该 zkid 必须单调递增。
## epoch
epoch：可以理解为当前集群所处的年代或者周期，每个 leader 就像皇帝，都有自己的年号，所
以每次改朝换代，leader 变更之后，都会在前一个年代的基础上加 1。这样就算旧的 leader 崩溃
恢复之后，也没有人听他的了，因为 follower 只听从当前年代的 leader 的命令。
## Zab 协议有两种模式-恢复模式（选主）、广播模式（同步）
Zab 协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）。当服务启动或者在领导
者崩溃后，Zab 就进入了恢复模式，当领导者被选举出来，且大多数 Server 完成了和 leader 的状
态同步以后，恢复模式就结束了。状态同步保证了 leader 和 Server 具有相同的系统状态。
## ZAB 协议 4 阶段
### Leader election（选举阶段-选出准 Leader）
1. Leader election（选举阶段）：节点在一开始都处于选举阶段，只要有一个节点得到超半数
节点的票数，它就可以当选准 leader。只有到达 广播阶段（broadcast） 准 leader 才会成
为真正的 leader。这一阶段的目的是就是为了选出一个准 leader，然后进入下一个阶段。

### Discovery（发现阶段-接受提议、生成 epoch、接受 epoch）
2. Discovery（发现阶段）：在这个阶段，followers 跟准 leader 进行通信，同步 followers
最近接收的事务提议。这个一阶段的主要目的是发现当前大多数节点接收的最新提议，并且
准 leader 生成新的 epoch，让 followers 接受，更新它们的 accepted Epoch
一个 follower 只会连接一个 leader，如果有一个节点 f 认为另一个 follower p 是 leader，f
在尝试连接 p 时会被拒绝，f 被拒绝之后，就会进入重新选举阶段。

### Synchronization（同步阶段-同步 follower 副本）
3. Synchronization（同步阶段）：同步阶段主要是利用 leader 前一阶段获得的最新提议历史，
同步集群中所有的副本。只有当 大多数节点都同步完成，准 leader 才会成为真正的 leader。
follower 只会接收 zxid 比自己的 lastZxid 大的提议。

### Broadcast（广播阶段-leader 消息广播）
4. Broadcast（广播阶段）：到了这个阶段，Zookeeper 集群才能正式对外提供事务服务，
并且 leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。
ZAB 提交事务并不像 2PC 一样需要全部 follower 都 ACK，只需要得到超过半数的节点的 ACK 就
可以了。
### ZAB 协议 JAVA 实现（FLE-发现阶段和同步合并为 Recovery Phase（恢复阶段））
协议的 Java 版本实现跟上面的定义有些不同，选举阶段使用的是 Fast Leader Election（FLE），
它包含了 选举的发现职责。因为 FLE 会选举拥有最新提议历史的节点作为 leader，这样就省去了
发现最新提议的步骤。

实际的实现将 发现阶段 和 同步合并为 Recovery Phase（恢复阶段）。所
以，ZAB 的实现只有三个阶段：Fast Leader Election；Recovery Phase；Broadcast Phase。

## 投票机制
每个 sever 首先给自己投票，然后用自己的选票和其他 sever 选票对比，权重大的胜出，使用权
重较大的更新自身选票箱。具体选举过程如下：
1. 每个 Server 启动以后都询问其它的 Server 它要投票给谁。对于其他 server 的询问，
server 每次根据自己的状态都回复自己推荐的 leader 的 id 和上一次处理事务的 zxid（系
统启动时每个 server 都会推荐自己）
2. 收到所有 Server 回复以后，就计算出 zxid 最大的哪个 Server，并将这个 Server 相关信
息设置成下一次要投票的 Server。
3. 计算这过程中获得票数最多的的 sever 为获胜者，如果获胜者的票数超过半数，则改
server 被选为 leader。否则，继续这个过程，直到 leader 被选举出来
4. leader 就会开始等待 server 连接
5. Follower 连接 leader，将最大的 zxid 发送给 leader
6. Leader 根据 follower 的 zxid 确定同步点，至此选举阶段完成。
7. 选举阶段完成 Leader 同步后通知 follower 已经成为 uptodate 状态
8. Follower 收到 uptodate 消息后，又可以重新接受 client 的请求进行服务了

目前有 5 台服务器，每台服务器均没有数据，它们的编号分别是 1,2,3,4,5,按编号依次启动，它们
的选择举过程如下：
1. 服务器 1 启动，给自己投票，然后发投票信息，由于其它机器还没有启动所以它收不到反
馈信息，服务器 1 的状态一直属于 Looking。
2. 服务器 2 启动，给自己投票，同时与之前启动的服务器 1 交换结果，由于服务器 2 的编号
大所以服务器 2 胜出，但此时投票数没有大于半数，所以两个服务器的状态依然是
LOOKING。
3. 服务器 3 启动，给自己投票，同时与之前启动的服务器 1,2 交换信息，由于服务器 3 的编
号最大所以服务器 3 胜出，此时投票数正好大于半数，所以服务器 3 成为领导者，服务器
1,2 成为小弟。
4. 服务器 4 启动，给自己投票，同时与之前启动的服务器 1,2,3 交换信息，尽管服务器 4 的
编号大，但之前服务器 3 已经胜出，所以服务器 4 只能成为小弟。
5. 服务器 5 启动，后面的逻辑同服务器 4 成为小弟。

# Zookeeper 工作原理（原子广播）
1. Zookeeper 的核心是原子广播，这个机制保证了各个 server 之间的同步。实现这个机制
的协议叫做 Zab 协议。Zab 协议有两种模式，它们分别是恢复模式和广播模式。
2. 当服务启动或者在领导者崩溃后，Zab 就进入了恢复模式，当领导者被选举出来，且大多
数 server 的完成了和 leader 的状态同步以后，恢复模式就结束了。
3. 状态同步保证了 leader 和 server 具有相同的系统状态
4. 一旦 leader 已经和多数的 follower 进行了状态同步后，他就可以开始广播消息了，即进
入广播状态。这时候当一个 server 加入 zookeeper 服务中，它会在恢复模式下启动，发
现 leader，并和 leader 进行状态同步。待到同步结束，它也参与消息广播。Zookeeper
服务一直维持在 Broadcast 状态，直到 leader 崩溃了或者 leader 失去了大部分的
followers 支持。
5. 广播模式需要保证 proposal 被按顺序处理，因此 zk 采用了递增的事务 id 号(zxid)来保
证。所有的提议(proposal)都在被提出的时候加上了 zxid。
6. 实现中 zxid 是一个 64 为的数字，它高 32 位是 epoch 用来标识 leader 关系是否改变，
每次一个 leader 被选出来，它都会有一个新的 epoch。低 32 位是个递增计数。
7. 当 leader 崩溃或者 leader 失去大多数的 follower，这时候 zk 进入恢复模式，恢复模式
需要重新选举出一个新的 leader，让所有的 server 都恢复到一个正确的状态。

# Znode 有四种形式的目录节点
1. PERSISTENT：持久的节点。
2. EPHEMERAL：暂时的节点。
3. PERSISTENT_SEQUENTIAL：持久化顺序编号目录节点。
4. EPHEMERAL_SEQUENTIAL：暂时化顺序编号目录节点。

[^1]: https://weread.qq.com/web/reader/e6d323b0723b6029e6d1c55kd3d322001ad3d9446802347