Spark

基本概念

- RDD：弹性分布式数据集

- Task：具体执行的任务。分为 ShuffleMapTask 和 ResultTask两种，两者分别对应与 Hadoop 中的 Map 和 Reduce。

  action算子：**reduce**，**collect**，**count**，**first**，**take**，**top**，**takeOrdered**，**countByKey**，**collectAsMap**，**lookup**，**aggregate**，**fold**，**saveAsFile**，**saveAsSequenceFile**

  transformation算子： **map**, **filter**, **groupby**, **flatmap**

- Job：用户提交的作业。

- Stage：Job分成的阶段。根据依赖划分 Stage ，每次shuffle以后划分成一个大的stage

- Partition：数据分区。一个RDD数据可以被划分成多个 Partition。

- NarrowDependency：窄依赖，即子 RDD 依赖于 RDD 中固定的 Partition。分为OneToOneDependency 和 RangeDependency 两种。

- ShuffleDependency：shuffle依赖，宽依赖。子 RDD 对父 RDD 中的所有 Partition 都有依赖。

- DAG：有向无环图。反映各 RDD 之间的依赖关系。DAGscheduler简单来说就是负责任务的逻辑调度，负责将作业拆分成不同阶段的具有依赖关系的多批任务 

Code 理解：

```scala
val src_table = "aa_location_test"

val srcpartitionName = "dt=2018-03-02"
val project = "lbs_tmc_qa"
val numPartition1 = 2000

/* 
	从ODPS指定的table和partition中读取数据组织成RDD, 并且将这个RDD 划分为 2000 个partition
*/
val odpsData = iOdpsOps.readTable(project, src_table, srcpartitionName, tran2Tp, numPartition1) //stage 0

/* 
	为ResultTask，NarrowDependency,不依赖于其他partition, RRD2 依赖为 2000个 partition， 一个partition对应一个Task，所以这里产生了2000个Task
*/
val RDD2 = odpsData.filter(tp => (tp.speed>0))  //stage 0

/* 
	为ShuffleMapTask，ShuffleDependency，假设RDD odpsData 划分为2000个分区，这个ShuffleMapTask 将依赖所有的分区, 经过shuttle RDD3的partition数量已经改变了,为numPartition2  
*/
val RDD3 = RDD2.groupBy(tp => tp.device_id) //stage 1

/* 
	为ResultTask，NarrowDependency,不依赖于其他partition, RDD4的partition数量不变 
*/
val RDD4 = RDD3.map(x=>x._2.toArray) //stage 1

/*
	这两行均为ResultTask，NarrowDependency,不依赖于其他partition, RDD4的partition数量不变
*/
val tripRDD = tp_groups.flatMap(TrackUtils.splitTrip).filter(x=>(x._1.size>min_trip_size)) //stage 1
val resultRDD = tripRDD.map(track_func).filter(x=>x.nonEmpty).map(line=>line.split("=")).filter(x=>x.size==3) //stage 1

/* 
	将resultRDD cache在内存中，因为下面要以它作为parent RDD 生成三个child RDD
	persist 可以把RDDcache在disk中 
*/
resultRDD.persist(MEMONY_AND_DISK_SER)  //stage 1
val linkStrRDD = resultRDD.flatMap(x=>x(0).split(";"))  //stage 1
val linkWeightRDD = resultRDD.flatMap(x=>x(1).split(";"))  //stage 1
val nodeWeightRDD = resultRDD.flatMap(x=>x(2).split(";"))  //stage 1

/*
	action 对child RDD 进行打印, 之前的都是transformation，是lazy的并没有真正触发计算
*/
linkStrRDD.foreach(x=> { println("---"); println(x(0)); println(x(1))})  //stage 2
linkWeightRDD.foreach(x=> { println("---"); println(x(0)); println(x(1))})
nodeWeightRDD.foreach(x=> { println("---"); println(x(0)); println(x(1))})

/* 
	unpersist 必须要调用，否则会outofmemory，另外必须要action之后调用，否则过早的清楚缓存，会使得后面的子RDD无法利用其缓存
*/
resultRDD.unpersist()


```

