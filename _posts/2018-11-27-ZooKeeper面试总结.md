---
layout:     post
title:      "ZooKeeper面试总结"
author:     "CoderLen"
header-img: "img/in-post/zookeeper-bg.jpg"
tags:
    - 面试
    - 大数据
    - zookeeper
---


## ZooKeeper面试总结

1. ZooKeeper是什么？

   分布式的协同服务。

2. ZooKeeper提供了什么？

   1. 文件系统
   2. 通知机制

3. ZooKeeper文件系统

   ZooKeeper提供了一个多层级的节点命名空间，节点称为znode。与文件系统不同的是，这些节点都是可以设置关联的数据，而文件系统中只有文件节点可以存放数据而目录节点不行。ZooKeeper为了保证高吞吐和低延迟，在内存中维护了这个树状的目录结构，这种特性使得ZooKeeper不能用于存放大量的数据，每个节点的存放数据的上限为1M。

4. 四种类型的znode

   1. PERSISTENT-持久化目录节点

      客户端与ZooKeeper断开连接后，该节点依旧存在

   2. PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点

      客户端与ZooKeeper断开连接后，该节点依旧存在，只是ZooKeeper给该节点名称进行了顺序编号

   3. EPHEMERAL-临时目录节点

      客户端与ZooKeeper断开连接后，该节点被删除

   4. EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点

      客户端与ZooKeeper断开连接后，该节点被删除，只是ZooKeeper给该节点名称进行顺序编号

   5. ZooKeeper通知机制

      client端对某个znode建立一个watcher时间，当该znode发生变化时，这些client会受到zk的通知，然后client可以根据znode变化来做出业务上的改变等。

   6. ZooKeeper可以做什么？

      1. 命名服务

         命名服务是指通过指定的名字来获取资源或服务的地址，利用zk创建一个全局的路径，即时唯一的路径，这个路径就可以作为一个名字，指向集群中机器或者提供服务的地址，又或者一个远程的对象等。

      2. 配置管理

         程序分布式的部署在不同的机器上，将程序的配置信息放在znode下，当有配置发生改变时，也就是znode发生变化时，可以通过改变zk中某个目录节点的内容，利用watcher通知给各个客户端，从而更改配置。

      3. 集群管理

         1. 是否有机器加入或退出

            所有机器约定在父目录下创建临时目录节点，然后监听父目录节点下的子节点变化。一旦有机器挂掉，该机器与ZooKeeper的连接断开，其所创建的临时目录节点也被删除，所有其他机器都收到通知：某个节点被删除了。

         2. 选举master

            例如，所有机器创建临时顺序编号目录节点，每次选取编号最小的机器作为master就好。

      4. 分布式锁

         有了ZooKeeper的一致性文件系统，锁的问题变得容易。锁服务可以分成两类，一个是保持独占，另一个是控制时序。

         1. 保持独占，我们把znode看作是一把锁，通过createznode的方式来实现。所有客户端都去创建/distribute_lock节点，最终成功创建的那个客户端也即拥有了这把锁。用完删除掉自己创建的/distribute_lock节点就释放出锁。
         2. /distribute_lock已经预先存在，所有客户端在它下面创建临时顺序编号目录节点，和master一样，编号最小的获得锁，用完删除，依次方便。

      5. 队列管理

         两种类型的队列

         1. 同步队列，当一个队列的成员都聚齐时，这个队列才可用，否则一致等待。

            在约定的目录下创建临时目录节点，监听节点数目是否是我们要求的数目。

         2. 队列按照FIFO方式进行入队和出队操作。

            和分布式锁服务中的控制时序的场景基本原理一致，入列有编号，出列按编号。创建PERSISTENT_SEQUENTIAL节点，创建成功时Watcher通知等待的队列，队列删除序列号最小的节点以消费。此场景下，znode用于消息存储，znode存储的数据就是消息队列中的消息内容，SEQUENTIAL序列号就是消息的编号，按序取出即可。由于创建的节点是持久化的，所以不必担心队列消息丢失的问题。

      6. ZooKeeper的工作原理

         ZooKeeper的核心是原子广播，这个机制保证了各个Server之间的同步。实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）。当服务启动或者Leader崩溃后，Zab就进入了恢复模式，当新的Leader被选举出来，且大多数Server完成了和Leader的状态同步以后，恢复模式就结束了。状态同步保证了Leader和Server具有相同的系统状态。

      7. ZooKeeper是如何保证事务的顺序一致性的？

         zk采用了递增的事务id来识别，所有的proposal（提议）都在被提出的时候加上了zxid，zxid实际上是一个64位数字，高32位是epoch用来标识leader是否发生了改变，如果有新的leader产生出来，epoch会自增。低32位用来递增计数。当新产生的peoposal的时候，会依据数据库的两阶段过程，首先会向其他的Server发出事务执行请求，如果超过半数的机器都能执行并且能够成功，那么就会开始执行。

      8. ZooKeeper下的Server工作状态

         1. LOOKING：当前Server不知道Leader是谁，正在搜寻；
         2. LEADING：当前Server是Leader
         3. FOLLOWING：leader已经选举出来，当前Server与之同步。

      9. ZooKeeper是如何选择主Leader的？

         当Leader崩溃或者Leader失去大多数的follower，这时zk进入恢复模式，恢复模式需要重新选举出一个新的leader，让所有的Server都恢复到一个正确的状态。

         zk的选举算法有两种：一种是基于basic paxos实现的，另外一种是基于fast paxos算法实现的。系统默认的选举算法是fast paxos。

         1. base paxos

         2. fast paxos

            在选举的过程中，某个server首先向所有的server提议自己要成为leader，当其他server收到提议以后，解决epoch和zxid的冲突，并接受对方的提议，然后向对方发送接受提议完成的消息，重复这个流程，最后一定能选举出Leader。

            ![](https://segmentfault.com/img/bV8XeR?w=533&h=451)

      10. 






## 参考

1. https://segmentfault.com/a/1190000014479433