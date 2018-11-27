### MapReduce面试总结

1. Hadoop MapReduce提供了什么机制允许一个作业的各个任务间共享只读文件？尝试描述该机制的工作原理。

   DistributedCache，即分布式缓存，原理是借助HDFS实现数据分发功能，实际上是一个磁盘（不是内存）缓存实现，具体可参考我的书《Hadoop技术内幕：深入解析MapReduce架构设计与实现原理》第五章

2. 请描述MapReduce中shuffle阶段的工作流程？如何优化shuffle阶段？

3. 一个MapReduce作业的Map Task数目是如何决定的？一般情况下，在Hadoop中，一个处理文本文件的作业的Map Task是如何计算的？

   由InputFormat决定。默认情况下采用TextInputFormat，MapTask个数由block size、min inputsplit和max inputsplit决定。

4. 什么叫数据本地性？Hadoop采用了哪些机制提高任务的数据本地性？

   数据本地性是指优先将任务调度到数据所在的节点上，这样可避免跨网络读取数据，从而提高任务运行效率。数据本地性分为节点本地性（node locality）和机架本地性（rack locality）两种。提高数据本性的方法有：delay scheduling（有相关论文，可自行查找）、增加数据副本数等。

5. 描述Hadoop MapReduce的容错机制

6. MapReduce中Partitioner的作用是什么，哪些情况下需要自定义Parititioner？

   Partitioner的作用是决定Map Task产生的数据记录交给哪个Reduce Task处理。默认实现是：（key）mod R，其中R是Reduce Task个数。一般情况下，当需要按照key的一部分（不是全部，比如key的前三个字节）进行partition，或者按照key范围进行partition时，需要自定义Partitioner。

7. MapReduce中Combiner的作用是什么，它一般用于哪些场景？举出一个不能使用Combiner的例子。

   Combiner位于Map Task中，相当于local reducer，通常跟Reducer逻辑一样，作用是对Map Task数据进行局部汇总或者规约，以减少数据输出量。

    