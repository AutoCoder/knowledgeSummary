1. hadoop和spark的都是并行计算，那么他们有什么相同和区别

   两者都是用mr模型来进行并行计算，hadoop的一个作业称为job，job里面分为map task和reduce task，每个task都是在自己的进程中运行的，当task结束时，进程也会结束

   spark用户提交的任务成为application，一个application对应一个sparkcontext，app中存在多个job，每触发一次action操作就会产生一个job

   这些job可以并行或串行执行，每个job中有多个stage，stage是shuffle过程中DAGSchaduler通过RDD之间的依赖关系划分job而来的，每个stage里面有多个task，组成taskset有TaskSchaduler分发到各个executor中执行，executor的生命周期是和app一样的，即使没有job运行也是存在的，所以task可以快速启动读取内存进行计算

   hadoop的job只有map和reduce操作，表达能力比较欠缺而且在mr过程中会重复的读写hdfs，造成大量的io操作，多个job需要自己管理关系

   spark的迭代计算都是在内存中进行的，API中提供了大量的RDD操作如join，groupby等，而且通过DAG图可以实现良好的容错

2. Hive中存放是什么？

   表。 
   存的是和hdfs的映射关系，hive是逻辑上的数据仓库，实际操作的都是hdfs上的文件，HQL就是用sql语法来写的mr程序。



3. map-reduce (mapper, reducer, combiner)

   ​	一个MapReduce的job，在map之后，reduce之前，会有一个数据聚集的过程，即map完的数据会按照key聚集在一起，会有一个shuffle的过程，然后再进入reduce。 

      	在不同的节点上的map会将同一个key的数据传输到同一个节点上，而传输就会涉及到数据量、传输时间，combiner的作用就是在每个节点map完之后，将同一个节点上的数据，按照key输入combiner中，然后combiner可以将同一个key的数据合并或压缩，然后再传输出去。

   ​	并不是所有的job都适用combiner，只有操作满足结合律的才可设置combiner。combine操作类似于：opt(opt(1, 2, 3), opt(4, 5, 6))。如果opt为求和、求最大值的话，可以使用，但是如果是求中值的话，不适用。 

4. 如何查看文件在hadoop集群上的分布

   ```shell
   hadoop fsck '/data/uaa/commons/cdum/features/dev_feature_vec/2018-08-26/mobile/ios/VV/EPISODE_ID' -files -blocks -locations -racks 
   
   Status: HEALTHY
    Total size:    33679572392 B
    Total dirs:    1
    Total files:   125
    Total symlinks:                0
    Total blocks (validated):      247 (avg. block size 136354544 B)
    Minimally replicated blocks:   247 (100.0 %)
    Over-replicated blocks:        0 (0.0 %)
    Under-replicated blocks:       0 (0.0 %)
    Mis-replicated blocks:         0 (0.0 %)
    Default replication factor:    3
    Average block replication:     3.0
    Corrupt blocks:                0
    Missing replicas:              0 (0.0 %)
    Number of data-nodes:          1418
    Number of racks:               109
   FSCK ended at Thu Aug 30 10:04:56 CST 2018 in 91 milliseconds
   ```

5. 如何合理设置mapper和reducer的个数
