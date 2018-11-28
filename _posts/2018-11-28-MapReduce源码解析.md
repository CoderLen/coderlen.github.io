# MapReduce源码解析

```
#MapReduce源码项目组成
-- hadoop-mapreduce-client
​   -- hadoop-mapreduce-client-app
​   -- hadoop-mapreduce-client-common
​   -- hadoop-mapreduce-client-core
​   -- hadoop-mapreduce-client-hs
​   -- hadoop-mapreduce-client-hs-plugins
​   -- hadoop-mapreduce-client-jobclient
​   -- hadoop-mapreduce-client-nativetask
​   -- hadoop-mapreduce-client-shuffle
​   -- hadoop-mapreduce-client-uploader
```





<p>
<b>YARNRunner</b>：允许JobClient运行在Yarn上，里面包含一个ResourceMgrDelegate的成员变量，这个是ResourceManager的代理类，负责与Yarn集群通信，包括Job的提交、Job状态查询等，也可以获取到集群信息。而ResourceMgrDelegate是继承了YarnClient。YARNRunner的submitJob来提交Job。
</p>

<p>
<b>YarnClient</b>：一个抽象类，提供了一个createYarnClient()静态方法获取YarnClientImpl实现类对象，提供了提交Application，获取Application状态等一系列接口。
</p>


​	



​	