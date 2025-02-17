实战项目地址：https://github.com/JianMin-Xie/hello-zookeeper  

# 分布式的一致性问题
分布式的一致性问题是指在分布式环境中引入数据复制机制后，不同数据节点之间可能出现的，并无法依靠计算机应用程序自身解决的数据不一致情况。简单地讲，数据一致性就是指在对一个数据副本进行更新的同时，必须确保也能更新到其他副本，否则不同副本之间的数据将不再一致。

# 一致性级别
* 强一致性：要求系统写入什么，读出来的也会是什么。

* 弱一致性：约束了系统在写入成功后，不承诺立即可以读到写入的值，也不具体承诺多久之后数据能够达到一致性，但会尽可能保证到某个时间级别（比如秒级别）后，数据能够达到一致状态。弱一致性还可以再进行细分：
  * 会话一致性：在同一个客户端会话中可以读到一致的值，但其他会话不能保证
  * 用户一致性：在同一个用户中可以读到一致的值，但其他用户不能保证

* 最终一致性：是弱一致性的一个特例。系统会保证在一定时间内，能够达到一个数据一致的状态。

# 分布式事务
分布式事务是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于分布式系统的不同节点之上。

典型的场景：
一个跨银行的转账操作涉及调用两个异地的银行服务，其中一个是本地银行提供的取款服务，另一个则是目标银行提供的存款服务，这两个服务本身是无状态而且互相独立的，共同构成了一个完整的分布式事务。如果本地银行取款成功，但是因为某种原因存款服务失败了，那么必须回滚到存款前的状态，否则用户会发现自己的钱不翼而飞了。

# CAP 定理
CAP 定理告诉我们，一个分布式系统不可能同时满足一致性、可用性和分区容错性这三个基本需求，只能同时满足其中的两项。

## 一致性（Consistency）
在分布式环境中，一致性是指数据在多个副本之间是否保持一致的特性。
## 可用性（Availability）
可用性是指系统提供的服务必须一直处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内返回结果。
## 分区容错性（Partition tolerance）
分区容错性约束了一个分布式系统需要具有以下特性：分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障。

# BASE 理论
BASE 理论是对 CAP 定理中一致性和可用性权衡的结果，核心思想是即使无法做到强一致性，但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性。

## 基本可用（Basically Available）
基本可用是指分布式系统在出现不可预知的故障的时候，允许损失部分可用性。--但是不等价于系统不可用。
* **响应时间上的损失：** 正常情况下，一个在线搜索引擎需要在0.5秒之内返回给用户相应的查询结果，但由于出现故障（比如系统部分机房发生断电和断网故障），查询结果的响应时间增加到了1~2秒。
* **功能上的损失：** 正常情况下，在一个电子商务网站上进行购物，消费者几乎可以顺利的完成每一笔订单，但是在一些节日大促购物高峰的时候，由于消费者的购物行为激增，为了保护购物系统的稳定性，部分消费者可能会被引导到一个降级的页面。

## 软状态（Soft state）
软状态是指允许系统的数据存在中间状态，并认为该中间状态不会影响到系统的整体可用性，即允许系统在不同节点之间的数据副本之间进行数据同步的过程存在延时。

## 最终一致性（Eventually consistent）
最终一致性强调的是系统中的所有数据副本，在经过一定时间的同步后，最终能够达到一个一致的状态。

# 一致性协议
## 2PC
2PC，二阶段提交，通过引入协调者（Coordinator）来协调参与者的行为，并最终决定这些参与者是否要真正执行事务。

将事务的提交过程分成了两个阶段来进行处理，执行流程如下：   
**阶段一：提交事务请求（投票阶段）**   
1.事务询问。  
2.执行事务。  
3.各参与者向协调者反馈事务询问的响应。  

**阶段二：执行事务提交**  
协调者会根据各参与者的反馈情况来决定最终是否可以进行事务提交操作，正常情况下，包含以下两种可能：  
第一种可能，**执行事务提交**  
  假如协调者从所有参与者获得的反馈都是yes响应，那么会执行事务提交。  
  1.发送事务提交  
  2.事务提交  
  3.反馈事务提交结果  
  4.完成事务  
另外一种可能，**中断事务**  
  假如任何一个参与者向协调者反馈了No响应，或者在等待超时之后，协调者无法接收到所有参与者的反馈响应，那么就会中断事务。  
  1.发送回滚事务  
  2.事务回滚  
  3.反馈事务回滚结果  
  4.中断事务   
**2PC交互流程如下：**   
![](https://github.com/JianMin-Xie/Learning-Note/blob/master/pic/2PC交互流程.jpg)  
## 2PC的优缺点：
优点：原理简单，实现方便。  
缺点：同步阻塞、单点问题、数据不一致、太过保守。

## 3PC
3PC，三阶段提交，是2PC的改进版，将二阶段提交协议的“提交事务请求”过程一分为二，形成了由 CanCommit、PreCommit 和 doCommit 三个阶段组成的事务处理协议，其协议设计如下：  
![](https://github.com/JianMin-Xie/Learning-Note/blob/master/pic/3PC流程.jpg)  
**阶段一：CanCommit**   
1.事务询问   
2.各参与者向协调者反馈事务询问的响应。  

**阶段二：PreCommit**  
协调者会根据各参与者的反馈情况来决定最终是否可以进行事务的 PreCommit 操作，正常情况下，包含以下两种可能：  
第一种可能，**执行事务预提交**  
  假如协调者从所有参与者获得的反馈都是yes响应，那么会执行事务预提交。  
  1.发送预提交请求
  2.事务预提交
  3.各参与者向协调者反馈事务询问的响应

第二种可能，**中断事务**  
  假如任何一个参与者向协调者反馈了No响应，或者在等待超时之后，协调者无法接受所有参与者的反馈响应，那么会执行中断事务。 
  1.发送中断请求
  2.中断事务

**阶段三：doCommit**
该阶段将进行真正的事务提交，会存在以下两种可能：  
第一种可能，**执行提交**  
  假如协调者从所有参与者获得的反馈都是yes响应，那么会执行事务提交。  
  1.发送事务提交  
  2.事务提交  
  3.反馈事务提交结果  
  4.完成事务  
另外一种可能，**中断事务**  
  假如任何一个参与者向协调者反馈了No响应，或者在等待超时之后，协调者无法接收到所有参与者的反馈响应，那么就会中断事务。  
  1.发送回滚事务  
  2.事务回滚  
  3.反馈事务回滚结果  
  4.中断事务   

## 3PC的优缺点：
优点：相较于二阶段提交协议，三阶段最大的优点就是降低了参与者的阻塞范围，并且能够在出现单点故障后继续达成一致。  
缺点：在参与者接受了preCommit消息后，如果出现网络分区，此时协调者所在的节点无法和参与者进行正常的网络通信。

## Paxos 算法
用于达成共识性问题，即对多个节点产生的值，该算法能保证只选出唯一一个值。

主要有三类节点：

* 提议者（Proposer）：提议一个值；
* 接受者（Acceptor）：对每个提议进行投票；
* 告知者（Learner）：被告知投票的结果，不参与投票过程。

### 执行过程
规定一个提议包含两个字段：[n, v]，其中 n 为序号（具有唯一性），v 为提议值。  

**Prepare 阶段**  
下图演示了两个 Proposer 和三个 Acceptor 的系统中运行该算法的初始过程，每个 Proposer 都会向所有 Acceptor 发送 Prepare 请求。  
![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/1a9977e4-2f5c-49a6-aec9-f3027c9f46a7.png)  
当 Acceptor 接收到一个 Prepare 请求，包含的提议为 [n1, v1]，并且之前还未接收过 Prepare 请求，那么发送一个 Prepare 响应，设置当前接收到的提议为 [n1, v1]，并且保证以后不会再接受序号小于 n1 的提议。  

如下图，Acceptor X 在收到 [n=2, v=8] 的 Prepare 请求时，由于之前没有接收过提议，因此就发送一个 [no previous] 的 Prepare 响应，设置当前接收到的提议为 [n=2, v=8]，并且保证以后不会再接受序号小于 2 的提议。其它的 Acceptor 类似。   
![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/fb44307f-8e98-4ff7-a918-31dacfa564b4.jpg)

如果 Acceptor 接收到一个 Prepare 请求，包含的提议为 [n2, v2]，并且之前已经接收过提议 [n1, v1]。如果 n1 > n2，那么就丢弃该提议请求；否则，发送 Prepare 响应，该 Prepare 响应包含之前已经接收过的提议 [n1, v1]，设置当前接收到的提议为 [n2, v2]，并且保证以后不会再接受序号小于 n2 的提议。

如下图，Acceptor Z 收到 Proposer A 发来的 [n=2, v=8] 的 Prepare 请求，由于之前已经接收过 [n=4, v=5] 的提议，并且 n > 2，因此就抛弃该提议请求；Acceptor X 收到 Proposer B 发来的 [n=4, v=5] 的 Prepare 请求，因为之前接收到的提议为 [n=2, v=8]，并且 2 <= 4，因此就发送 [n=2, v=8] 的 Prepare 响应，设置当前接收到的提议为 [n=4, v=5]，并且保证以后不会再接受序号小于 4 的提议。Acceptor Y 类似。  
![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/2bcc58ad-bf7f-485c-89b5-e7cafc211ce2.jpg)

**Accept 阶段**  
当一个 Proposer 接收到超过一半 Acceptor 的 Prepare 响应时，就可以发送 Accept 请求。

Proposer A 接收到两个 Prepare 响应之后，就发送 [n=2, v=8] Accept 请求。该 Accept 请求会被所有 Acceptor 丢弃，因为此时所有 Acceptor 都保证不接受序号小于 4 的提议。

Proposer B 过后也收到了两个 Prepare 响应，因此也开始发送 Accept 请求。需要注意的是，Accept 请求的 v 需要取它收到的最大提议编号对应的 v 值，也就是 8。因此它发送 [n=4, v=8] 的 Accept 请求。  
![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/9b838aee-0996-44a5-9b0f-3d1e3e2f5100.png)

**Learn 阶段**  
Acceptor 接收到 Accept 请求时，如果序号大于等于该 Acceptor 承诺的最小序号，那么就发送 Learn 提议给所有的 Learner。当 Learner 发现有大多数的 Acceptor 接收了某个提议，那么该提议的提议值就被 Paxos 选择出来。  
![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/bf667594-bb4b-4634-bf9b-0596a45415ba.jpg)

## 小结
这三种协议都从不同方面不同程度地解决了分布式数据一致性问题。其中二阶段提交协议解决分布式事务的原子性问题，保证了分布式事务的多个参与者要么都执行成功，要么都执行失败。但是仍然存在同步阻塞、无限期等待、和“脑裂”等问题。三阶段提交协议则是在二阶段提交协议的基础上，添加了PreCommit过程，从而避免了二阶段提交协议中的无限等待问题。而Paxos算法引入了“过半”的理念，通俗地讲就是少数服从多数的原则。同时Paxos算法支持分布式节点之间角色的转换，这极大的避免的分布式单点问题的出现，因此Paxos既解决了无限等待问题，也解决了“脑裂”问题。

# ZooKeeper
ZooKeeper 是一个分布式协调服务，是分布式数据一致性的解决方案。分布式应用可以基于它实现数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

# ZooKeeper 的基本概念
## 集群角色
通常在分布式系统中，构成一个集群的每一台机器都有自己的角色，最典型的集群模式就是 Master/Slave 模式（主备模式），在这种模式中，处理所有写操作的机器称为 Master 机器，通过异步复制获取最新数据，并提供读服务的机器称为 Slave 机器。

而在 ZooKeeper 中，没有沿用传统的 Master/Slave 概念，而是引入了 Leader、Follower 和 Observer 三种角色。ZooKeeper 集群通过 Leader 选举过程来选定一台被称为“Leader”的机器，Leader 服务器为客户端提供读和写服务，Follower 和 Observer 都能够提供读服务，唯一的区别在于，Observer 不参与 Leader选举过程，也不参与写操作的“过半写成功”策略，因此 Observer 可以在不影响写性能的情况下提升集群的读性能。
## 会话（Session）
Session 是指客户端会话，客户端启动的时候，首先会与服务器建立一个 TCP 连接，从第一次连接建立开始，客户端会话的生命周期就开始了，通过这个连接，客户端能够通过心跳检测与服务器保持有效的会话，也能向 Zookeeper 服务器发送请求并接受响应，同时还能通过该连接接收来自服务器的 Watch 事件通知。

## 数据节点（Znode）
在谈到分布式的时候，通常说的“节点”是指组成集群的每一台机器。然而，在 Zookeeper 中，“节点”分为两类，第一类同样是指构成集群的机器，称之为机器节点；第二类则是指数据模型中的数据单元，我们称之为数据节点——ZNode。

## 版本
对应每个 ZNode,ZooKeeper 都会为其维护一个叫做 Stat 的数据结构，Stat 中记录了这个 ZNode 的三个数据版本，分别是 version（当前 ZNode 的版本），cversion（当前 ZNode 子节点的版本）和 aversion（当前 ZNode 的 ACL 版本）。

## Watcher
Watcher（事件监听器），ZooKeeper 允许用户在指定节点上注册一些 Watcher，并且在一些特定事件触发的时候，ZooKeeper 服务端会将事件通知到感兴趣的客户端上去。

## ACL
采用ACL（Access Control Lists）策略，类似于 UNIX 文件系统的权限控制。

# ZooKeeper 的 ZAB 协议
## ZAB 协议
事实上，ZooKeeper 并没有完全采用 Paxos 算法，而是使用了一种称为 ZooKeeper Atomic Broadcast（ZAB，ZooKeeper 原子消息广播协议）的协议作为其数据一致性的核心算法。

ZAB 协议支持崩溃恢复，是专门为 ZooKeeper 设计的协议。

在 ZooKeeper 中，主要依赖 ZAB 协议来实现分布式数据一致性，基于该协议，ZooKeeper 实现了一种主备模式的系统架构来保持集群中各副本之间数据的一致性。

## ZAB 协议与 Paxos 算法的联系与区别
* 两者都存在一个类似于 Leader 的角色，由其负责协调多个 Follower 进程的运行。  
* Leader 进程都会等待超过半数的 Follower 做出正确的反馈后，才会将一个提案进行提交。  
* 在 ZAB 协议中，每个 Proposal 中都包含了一个 epoch 值，用来代替当前的 Leader 周期，在 Paxos 算法中，同样存在这样的一个标识，只是名字变成了 Ballot。  

在 Paxos 算法设计的基础上，ZAB 协议额外添加了一个不同阶段。  

总的来讲，ZAB 协议和 Paxos 算法的本质区别在于，两者的设计目标不太一样。ZAB 协议主要用于构建一个高可用的分布式数据主备系统，例如 ZooKeeper，而 Paxos 算法则是用于构建一个分布式的一致性状态机系统。

# 使用 ZooKeeper
## 部署与运行
## 客户端脚本
## Java 客户端 API 使用
## 开源客户端
### ZkClient
ZkClient 在 ZooKeeper 原生 API 接口之上进行了包装，是一个更易用的 ZooKeeper 客户端。  
依赖：  
```
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.6</version>
</dependency>

<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
</dependency>
```
这里举一个简单的例子，如创建会话：  
```
// 使用ZkClient来创建一个ZooKeeper客户端
public class Create_Session_Sample {
    public static void main(String[] args) throws IOException, InterruptedException {
    	ZkClient zkClient = new ZkClient("192.168.102.140:2181", 5000);
    	System.out.println("ZooKeeper session established.");
    }
}
```
创建节点、删除节点、读取数据和更新数据等操作的代码可以在 https://github.com/JianMin-Xie/hello-zookeeper 中自行阅读。  
### Curator
除了封装了底层细节之外，Curator 还在 ZooKeeper 原生 API 的基础上进行了包装，提供了一套易用性和可读性更强的 Fluent 风格的客户端 API 框架。  

除此之外，Curator 中还提供了 ZooKeeper 各种应用场景（Recipe，如共享锁服务、Master 选举机制和分布式计数器等）的抽象封装。  

依赖：  
```
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.2.0</version>
</dependency>
```
这里举一个简单的例子，如创建会话：  
```
//使用curator来创建一个ZooKeeper客户端
public class Create_Session_Sample {
    public static void main(String[] args) throws Exception{
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        CuratorFramework client =
        CuratorFrameworkFactory.newClient("192.168.102.140:2181",
        		5000,
        		3000,
        		retryPolicy);
        client.start();
        Thread.sleep(Integer.MAX_VALUE);
    }
}
```
创建节点、删除节点、读取数据和更新数据等操作的代码可以在 https://github.com/JianMin-Xie/hello-zookeeper 中自行阅读。 
#### 典型的使用场景
Curator 不仅提供了便利的 API 接口，而且提供了一些典型场景的使用参考。这些使用参考都在 recipes 包中。  

依赖：  
```
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.2.0</version>
</dependency>
```
**事件监听**  
Curator 引入了 Cache 来实现对 ZooKeeper 服务端事件的监听。Cache 是 Curator 中对事件监听的包装。Cache 分两类监听类型：节点监听和子节点监听。  

**NodeCache** 用于监听指定 ZooKeeper 数据节点本身的变化。

**PathChildrenCache** 用于监听指定 ZooKeeper 数据节点的子节点变化情况。

**Master 选举**  
选举思路：选择一个根节点，例如/master_select，多台机器同时向该节点创建一个子节点/master_select/lock，利用 ZooKeeper 的特性，最终只有一台机器能够创建成功，成功的那台机器就作为 Master。

Curator 也是基于这个思路。

**分布式锁**  
在分布式环境中，为了保证数据的一致性，经常在程序的某个运行点（例如，减库存操作或者流水号生成等）需要进行同步控制。

**分布式计数器**  
指定一个 ZooKeeper 数据节点作为计数器，多个应用实例在分布式锁的控制下，通过更新该数据节点的内容来实现技术功能。

**分布式 Barrier**  
Barrier 是一种用来控制多线程之间同步的经典方式，在 JDK 中也自带了 CyclicBarrier 实现。

**工具类**   
* ZKPaths：提供了一些简单的 API 来构建 ZNode 路径、递归创建和删除节点等。
* EnsurePath：提供了一种能够确保数据节点的机制。





