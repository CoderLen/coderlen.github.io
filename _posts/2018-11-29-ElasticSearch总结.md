### ElasticSearch 调优总结

1. 采用G1垃圾回收机制代替默认的CMS

   ```
   JAVA_OPTS="$JAVA_OPTS -XX:UseParNewGC"
   JAVA_OPTS="$JAVA_OPTS -XX:UseConcMarkSweepGC"
   JAVA_OPTS="$JAVA_OPTS -XX:CMSInitiatingOccupancyFraction=75"
   JAVA_OPTS="$JAVA_OPTS -XX:+UseCMSInitiationgOccupancyOnly"
   ```



   ```
   JAVA_OPTS="$JAVA_OPTS -XX:+UseG1GC"
   JAVA_OPTS="$JAVA_OPTS -XX:MaxGCPauseMillis=200"
   ```

   4

2. 清理掉没用的缓存

   随着时间的推移，发现堆内存还是不够，后来才发现是cache的问题，其实集群建立的时候，我们是可以调整每个节点的缓存比例，缓存类型或者大小的。

   ```
   # 锁定内容，不让JVM写入Swapping，避免降低ES性能
   bootstrap.mlockall: true
   # 缓存的类型设置为Soft Reference，只有当内存不够的时候才进行回收
   index.cache.field.max_size: 50000
   index.cache.field.expire: 10m
   index.cache.field.type: soft
   ```

   但是如果不想重新配置节点并且重启，你可以做一个定时任务来定时清除cache

   ```
   http://localhost:9200/*/_cache/clear
   //清除所有索引的cache，如果对查询有实时性要求，慎用。
   ```

   到了晚上资源有空闲的时候，我们还能合并优化一下索引

   ```
   http://localhost:9200/*/_optimize
   ```

3. 优化ES的线程池

   cache：这是一个无限制的线程池，为每一个传入的请求创建一个线程

   fixed：这个有固定大小的线程池，大小 由size属性指定，允许你指定一个队列（使用queue_size属性指定）来保存请求，直到有一个空闲的线程来执行请求

   如果ES无法把请求放到队列里面(队列满了)，该请求将被拒绝。有很多线程池（可以使用Type属性指定要配置的线程池类型）。对性能来说，最重要的是下面几个。

   index：此线程池用于索引和删除操作。它的默认类型为fixed，size默认为可用处理器的数量，队列size默认是300

   search：此线程池用于搜索和计数请求。它的默认类型为fixed，size默认为可用处理器的数量*3，队列size默认是1000

   suggest：此线程池用于suggest的请求。它的默认类型为fixed，size默认为可用处理器的数量，队列size默认是1000

   get：用于实时的GET请求，它的默认类型为fixed，size默认为可用处理器的数量，队列size默认是1000

   bulk：用于批量操作。它的默认类型为fixed，size默认为可用处理器的数量，队列size默认是50

   percolate：用于预匹配器操作。它的默认类型为fixed，size认为可用处理器的数量，队列size默认是1000

   elasticsearch.yml可以配置

   ```
   threadpool.index.type: fixed
   threadpool.index.size: 100
   threadpool.index.queue_size: 500
   ```

   也可以用rest APi设置

   ```
   curl -XPUT 'localhost:9200/_cluster/settings' -d '{
       "transient": {
           "threadpool.index.type": "fixed",
           "threadpool.index.size": 100,
           "threadpool.index.queue_size": 500
       }
   }'
   ```

4. index过于庞大导致es经常崩溃

   按时间对数据进行分库处理

5. 调整refresh_interval数据刷新时间间隔。

   ```
   $ curl -XPUT 'http://localhost:9200/twitter/' -d '{
       "settings" : {
           "index" : {
            "refresh_interval":"60s"
           }
       }
   }'
   ```


