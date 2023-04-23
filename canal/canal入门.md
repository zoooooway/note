# Canal 入门
> https://github.com/alibaba/canal

canal 是由 Alibaba 开源的一个基于 binlog 的增量日志组件, 其核心原理是 canal 伪装成 Mysql 的 slave, 发送 dump 协议获取 binlog, 解析并存储起来给客户端消费。

![](https://raw.githubusercontent.com/zoooooway/picgo/nom/202304172008774.png)

## 原理

首先, 先来了解一下 MySQL 的主从复制.

### MySQL 主从复制原理
> https://blog.nowcoder.net/n/b90c959437734a8583fddeaa6d102e43#%E4%B8%80%E3%80%81%E6%A6%82%E8%A6%81

MySQL 主从复制涉及到三个线程, 一个运行在主节点(log dump thread), 其余两个(I/O thread, SQL thread)运行在从节点, 如下图所示:

![](https://raw.githubusercontent.com/zoooooway/picgo/nom/202304172038699.png)


**复制的基本过程**
1. 从节点执行 `start slave` 命令后开始复制主节点的 binary log . 从节点上的 I/O thread 去连接主节点, 并请求从指定 binary log 文件的指定位置(或者从最开始的日志)之后的日志内容;
2. 主节点接收到来自从节点 I/O thread 的请求后, log dump thread 将根据请求信息读取指定 binary log 文件指定位置之后的内容, 返回给从节点. 
   >返回信息中除了日志所包含的信息之外, 还包括本次返回的信息的 binary log file 的以及 binary log position(binary log中的下一个指定更新位置);
3. 从节点的I/O进程接收到主节点发送过来的日志内容、日志文件及位置点后, 将接收到的日志内容更新到本机的 relay log(中继日志)的文件(Mysql-relay-bin.xxx)的最末端, 并将读取到的 binary log 文件名和 binary log position 保存到 master info 文件中, 下次请求主节点时会携带这些信息；
4. Slave 的 SQL thread 检测到 relay log 中新增加了内容后, 会将 relay log 的内容解析成在主节点上实际执行过 SQL 语句, 然后在本数据库中按照解析出来的顺序执行, 并在relay-log.info 中记录当前应用中继日志的文件名和位置点。

![](https://raw.githubusercontent.com/zoooooway/picgo/nom/202304172133714.png)

如上图, binary log dump 过程是长连接, 类似 TCP 进行握手建立连接后, 不断推送数据流. 
   
### Canal 原理
canal 的核心思想是伪装为 MySQL 的从节点, 利用 MySQL 主从复制原理来获取源数据库的 binary log. 
从 MySQL 的主从复制原理我们可以得知: 
* canal 支持消费的开始点取决于 binary log 文件, 因为 MySQL 主从复制时, 从节点可以指定 binary log 文件以及偏移量, 假设 binary log 只存了最近三天的, 那么最早的消费点就只能向前三天.
* 源库与目标库之间的版本不同可能会影响同步结果. 因为 binary log 解析后是在源库执行的 SQL 语句, 使用 canal 获取并解析后, 在不同版本的目标库上执行 SQL 可能会出现异常.

> canal 其他的内容是一些架构设计, 感觉没有什么深入研究的必要.
> 后续如果需要再补充吧.