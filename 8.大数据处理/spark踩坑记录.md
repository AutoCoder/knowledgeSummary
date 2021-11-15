[Spark排错与优化](https://blog.csdn.net/lsshlsw/article/details/49155087)



**坑1**：hive表结构存在，但是表数据被删掉了，那么hive查询这张表不会报错但是spark sql 查询这张表会报错

**坑2**：sparkSession.sql(sql_str) 产出一个巨大的dataframe， 数据量非常大，但是出来的默认分区是200. 导致后续的并行度不够。 用repartition进行分区，这个也非常耗时。

stackoverflow 解决方案1： 

```scala
sparkSession.conf.set("spark.sql.shuffle.partitions", 64) //设置分区数
```

```scala
val result = sqlContext.sql("""
   // SET spark.sql.shuffle.partitions = 64 //和上面类似
   select * from (
     select *,random(64) as rand_part from bt_st_ent
   ) cluster by rand_part""")                            //将输出按照分区cluster
```

具体可以参考 [deepsence]( https://deepsense.ai/optimize-spark-with-distribute-by-and-cluster-by/) 的介绍

stackoverflow 解决方案2：

> spark.sql.files.maxPartitionBytes 134217728 (128 MB) The maximum number of bytes to pack into a single partition when reading files.

但是这个 [链接](https://stackoverflow.com/questions/44061443/how-does-spark-sql-decide-the-number-of-partitions-it-will-use-when-loading-data?rq=1)提到 这种方案是解决spark.read.json("examples/src/main/resources/people.json") 这种直接读取的case的，对于sql操作hive table 不起作用

**spark.sql.shuffle.partitions**和**spark.defalut.parallelism** 的区别：

​	 一个是在spark sql文档里的，描述上说（Configures the number of partitions to use when shuffling data for joins or aggregations.）默认200。

​	另一个是spark文档里的参数(Default number of partitions in RDDs returned by transformations like join, reduceByKey, and parallelize when not set by user.)。

​	我所理解的是当使用当spark sql且两者都有设置，shuffle.partitions会起作用，如果不是spark sql的shuffle，需要defalut.parallelism才起作用）

**坑3**： OOM 

1. driver OOM， 不要collect
2. executor OOM
   1. 可以增加内存总量spark.executor.memory
   2. 增加并行度partition
   3. 注意executor memory 和 storage memory的比例，老版本有spark.storage.memoryFraction = 0.6 by default， 也就是只有40%执行内存。新版本运行时会自动相互借用调整。尽量不要用cache和persist，会消耗内存

参考[链接](https://blog.csdn.net/yhb315279058/article/details/51035631)

**坑4**：数据倾斜

当groupby 以后，进行后续action出现错误，可能是OOM了。于是我加了repartition(8000)后依然报错，查看每个task的输入，其实不比之前的步骤多，但是依然失败。于是我又repartition(16000), 这次还在跑repartition的时候直接就报错了。

推测，有异常的key，导致groupby 记录异常多，所以才能解释为什么增加repartition反而提前出错了，因为一个repartition连一个key都放不下

参考[链接1](https://www.cnblogs.com/jxhd1/p/6528517.html)

**坑5**：GC时间过久导致，运行过慢

由于内存占用过多，导致频繁GC，且每次GC的量也很小，所以GC时间过久，按照一个blog的说法，GC线程会让Task线程暂停下来

**坑6**：在对string类别特征做onehot时transformer可能会引起数据丢失

```scala
val index_transformers: Array[org.apache.spark.ml.PipelineStage] = stringColumns.map(
    cname => new StringIndexer()
    .setInputCol(cname)
    .setOutputCol(s"${cname}_index")
    .setHandleInvalid("skip")  #skip 会丢掉这个字段为NULL的row，避免这种情况用keep
)

val one_hot_encoders: Array[org.apache.spark.ml.PipelineStage] = stringColumns.map(
    cname => new OneHotEncoder()
    .setInputCol(s"${cname}_index")
    .setOutputCol(s"${cname}_vec")
)
```

某些空值率特别高的字段做onehot时，用skip风险很高

**坑7** ： spark从2.1.1升级到2.3.1, scala从2.10升级到2.11。 会导致有些代码编译报错，如*“No TypeTag available for String”，“No TypeTag available for Option[Int]”*

解决方案：

```xml
-- pom.xml中加入依赖
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-reflect</artifactId>
    <version>2.12.0-M4</version>
</dependency>
```

**坑8**：stringIndexer不适合用来做device_id的编号， 因为device_Id有过多的distinct值， spark的stringIndexer是用来做category特征编号的。

```scala
-- 来自 stackoverflow，这种方式会在driver上创建巨大的map，可能会导致OOM
With 15M distinct values, you are actually creating a huge Map on the driver and it runs out of memory... A straightforward workaround would be to increase the memory of the driver. If you use spark-submit you can use `--driver-memory 16g`. You can also use the `spark.driver.memory`property in the config file

-- 正确的做法
uid_rdd = data\
    .select("user")\
    .distinct()\
    .rdd.map(lambda x : x["user"])\
    .zipWithIndex()
uid_map = spark.createDataFrame(uid_rdd, ["user", "user_idx"])
```

**坑9**： pivot在使用过程中，很容易出现OOM

把不同的distinct value分段来做pivot，然后再full outer join 起来

