# Raft
> 参阅：
> https://raft.github.io/
> http://thesecretlivesofdata.com/raft/
> https://zhuanlan.zhihu.com/p/32052223

`Raft`是一个易于理解的一致性算法。在容错性和性能方面，它与`Paxos`算法相同。不同之处在于`Raft`算法将一致性问题分解成相对独立的子问题，从而更加易于理解。这些子问题包括：
* **Leader选举（Leader election）**
* **日志同步（Log replication）**
* 安全性（Safety）
* 日志压缩（Log compaction）
* 成员变更（Membership change）
* ......


## 核心概念
### Leader election
为了保证高可用，同一服务往往会部署多个实例组成集群，这时，这些实例都可以视作一个个节点。为便于理解，我们这里将每个节点都视作数据库服务。
`Raft`中第一个需要了解的就是节点的状态。每个节点都会处于以下下三种状态之一：
* **Follower**：追随者，接受并持久化`Leader`同步的日志，在`Leader`告之日志可以提交之后，提交日志。
* **Candidate**：候选者，可以发起`Leader`选举，尝试成为`Leader`。
* **Leader**：领导者，接受客户端请求，并向`Follower`同步请求日志，当日志同步到**大多数**节点上后告诉`Follower`提交日志。
  
Raft要求系统在任意时刻最多只有一个`Leader`，正常工作期间只有`Leader`和`Follower`。`Leader`会通过`heartbeat`定期来和`Follower`进行通信。

`Raft`算法将时间分为一个个的任期（`term`），每一个`term`的开始都是`Leader`选举。在成功选举`Leader`之后，`Leader`会在整个`term`内管理整个集群。如果`Leader`选举失败，该`term`就会因为没有`Leader`而结束。

一开始，所有的节点都会初始化为`Follower`状态，如果`Follower`在一段时间内（心跳超时）没有收到`Leader`的消息（心跳），那么他们就可以成为`Candidate`，并且开始一个新的`term`并进行一轮`Leader`选举来试图让自己成为`Leader`。

值得注意的是，`Raft`有两个控制选举的超时设置。
* 选举超时：每个`Follower`等待成为`Candidate`的时间。
  选举超时是指`Follower`在未收到`Leader`的心跳后，需要等待多久才会成为`Candidate`。每个`Follower`的选举超时是随机的。减少了多个`Follower`同时开启`Leader`选举的概率。
* 心跳超时：`Leader`发送心跳的时间间隔

`Candidate`首先给自己投票，并且会向其他节点发出投票信息(`RequestVote RPC`)，让其他节点同意它成为`Leader`。`Candidate`在发送`RequestVote RPC`时，要带上自己的最后一条日志的`term`和`log index`。如果一个节点没有投过票给其他节点，并且自己的日志并没有比请求中携带的日志更新，那么它会同意投票,并且会重置自己的选举超时时间。如果一个节点也成为了`Candidate`，那么由于它给自己投了票，它就不会同意投票。
> 日志比较的原则是，如果本地的最后一条`log entry`的`term`更大，则`term`大的更新，如果`term`一样大，则`log index`更大的更新
所以会存在三种结果：
* 超过半数的节点都同意投票，那么，该`Candidate`成功当选为`Leader`。
* 在选举过程中收到了`Leader`的信息，这表示已经有`Candidate`成为`Leader`了，那么本次选举失败，节点变回`Follower`。
* 同意投票的节点没有超过半数，本次选举失败，等待选举超时后发起下一次选举。

![](https://raw.githubusercontent.com/zoooooway/picgo/master/202304112141935.png)

### Log replication
日志由有序编号（`log index`）的日志条目组成。每个日志条目包含它被创建时的任期号（`term`），和用于状态机执行的命令。
![](https://raw.githubusercontent.com/zoooooway/picgo/master/202304110944485.png)

Raft日志同步保证如下两点：
* 如果不同日志中的两个条目有着相同的索引和任期号，则它们所存储的命令是相同的。
  因为`Leader`在一个`term`内在给定的一个`log index`最多创建一条日志条目，同时该条目在日志中的位置也从来不会改变。
* 如果不同日志中的两个条目有着相同的索引和任期号，则它们之前的所有条目都是完全一样的。
  这个特性来源于：当发送一个`AppendEntries RPC`时，`Leader`会把新日志条目紧接着之前的条目的`log index`和`term`都包含在里面。如果`Follower`没有在它的日志中找到`log index`和`term`都相同的日志，它就会拒绝新的日志条目。

假设现在有三个节点A，B，C。A是`Leader`，客户端的写入请求都需要先经过A。`Leader`把客户端的请求写入首先加入到它自己的日志中，然后通过下一次`heartbeat`并行的向`Follower`发起`AppendEntries RPC`请求，将该请求复制到`Follower`的日志上。当日志成功复制到`Follower`上时，`Follower`会成功返回，当超过半数的`Follower`返回成功后，`Leader`会将此条日志置为已提交状态，并且向客户端发送响应。随后，在下一次`heartbeat`，该日志的已提交状态又会被发送给各个`Follower`。
假如某个`Follower`异常，那么`Leader`会通过`heartbeat`不断的重试`AppendEntries RPC`，直到所有`Follower`成功的复制日志。
注意此处提到的是异常，如果是`Follower`主动拒绝新的日志（比如该日志条目之前的日志尚未写入），那么`Leader`会从后向前依次尝试每个日志条目，直到找到`Follower`和`Leader`一致的位点。


一般情况下，`Leader`和Followers的日志保持一致。然而，`Leader`崩溃可能会导致日志不一致。
一个`Follower`可能会丢失掉`Leader`上的一些条目，也有可能包含一些`Leader`没有的条目，也有可能两者都会发生。丢失的或者多出来的条目可能会持续多个任期。

![](https://raw.githubusercontent.com/zoooooway/picgo/master/202304120944352.png)

**`Leader`通过强制`Follower`复制它的日志来处理日志的不一致，`Follower`上的不一致的日志会被`Leader`的日志覆盖。**

`Leader`为了使`Follower`的日志同自己的一致，`Leader`需要找到`Follower`同它的日志一致的地方，然后覆盖`Follower`在该位置之后的条目。

`Leader`会**从后往前**试，每次`AppendEntries RPC`失败后尝试前一个日志条目，直到成功找到每个`Follower`的日志一致位点，然后向后逐条覆盖`Follower`在该位置之后的条目。
















