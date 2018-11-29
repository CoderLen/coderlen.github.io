<h2>MapReduce源码解析</h2>

```
#MapReduce源码项目组成
-- hadoop-mapreduce-client
   -- hadoop-mapreduce-client-app
   -- hadoop-mapreduce-client-common
   -- hadoop-mapreduce-client-core
   -- hadoop-mapreduce-client-hs
   -- hadoop-mapreduce-client-hs-plugins
   -- hadoop-mapreduce-client-jobclient
   -- hadoop-mapreduce-client-nativetask
   -- hadoop-mapreduce-client-shuffle
   -- hadoop-mapreduce-client-uploader
```

<p>
<b>YARNRunner</b>：允许JobClient运行在Yarn上，里面包含一个ResourceMgrDelegate的成员变量，这个是ResourceManager的代理类，负责与Yarn集群通信，包括Job的提交、Job状态查询等，也可以获取到集群信息。而ResourceMgrDelegate是继承了YarnClient。YARNRunner的submitJob来提交Job。
</p>

<p>
<b>YarnClient</b>：一个抽象类，提供了一个createYarnClient()静态方法获取YarnClientImpl实现类对象，提供了提交Application，获取Application状态等一系列接口。
</p>


<h3>Job提交流程</h3>

1. Job类里面的waitForCompletion()的方法里，调用了submit()方法去提价Job。submit()方法里面通过JobSubmitter的submitJobInternal(Job job, Cluster cluster)来提交任务。Job包含了Job相关的配置，Cluster则是运行环境的配置。
2. Cluster有一个ClientProtocol类型的成员变量client，这个ClientProtocol其实是一个接口，里面定义了很多Job和Task相关的接口，其中就有提交作业的方法submitJob()。ClientProtocol有两个实现类LocalJobRunner和YARNRunner。
3. YARNRunner的submitJob()有三个参数，JobId, JobSubmitDir, Credentials。通过JobSubmitDir和Credentials，就可以创建ApplicationSubmissionContext对象，它封装了要运行的Application的相关信息，例如ApplicationId, ApplicationName, Queue，资源请求，Job优先级等信息。然后就可以利用ResourceManager的代理类ResourceMgrDelegate来提交Application。实际就是通过YarnClient来提交Application。
4. YarnClient的实现类YarnClientImpl类里，再调用submitApplication方法处理ApplicationSubmissionContext。首先是用SubmitApplicationRequest封装appContext成一个request，再通过ApplicationClientProtocol的submitApplication方法提交request。最后通过一个while循环，监控Application状态，包括提交任务超时监控等。
5. ApplicationClientProtocol，是客户端跟ResourceManager之间的一个协议，用来提交或终止Jobs，获取Application信息，集群信息，节点，队列，ACLs等信息，它也是一个接口类，在YarnClient调用serviceStart()方法时，使用ClientRMProxy动态代理生成。其中一个实现类是ApplicationClientProtocolPBClientImpl。



​	