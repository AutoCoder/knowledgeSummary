

1、RDD,DAG,Stage怎么理解？

	RDD弹性分布式数据集, 是对分布式系统中数据的一个抽象，比如HDFS， RDD支持数据变换接口，如常用的filter, map等等，在变换的过程中，RDD的数据并不立即发生实际变化（Lazily transform），而是保存了数据的依赖关系，直到要求RDD进行动作（Action）时。RDD会从全局的角度来优化Transform的运行过程，从而节省时间。RDD的cache操作将数据的中间结果保存在内存中，方便下次使用。RDD的Action操作将数据的运算结果进行统计和返回。常见的如count 和 collect.

2、宽依赖 窄依赖怎么理解？

	窄依赖， 子rdd仅仅依赖于父rdd的特定的partition，宽依赖， 子rdd依赖于父rdd所有的partition，会触发shuffle，导致大量的数据在系统中传输。

3、Stage是基于什么原理分割task的？

	stage是一个顺序执行的阶段，每个stage下会产生task集， 一般来说，一个 rdd 有多少个 partition，就会有多少个 task，因为每一个 task 只是处理一个 partition 上的数据 。在同一个stage 中 所有的task 结束后才能根据DAG 依赖执行下一个stage 中的task， stage之间是顺序执行的。

4、血统的概念（lineage）

5、Task的概念

6、容错方法

	RDD实现了基于Lineage的容错机制。RDD的转换关系，构成了compute chain，可以把这个compute chain认为是RDD之间演化的Lineage。**在部分计算结果丢失时，只需要根据这个Lineage重算即可。** 
	
	Task失败的话会在executor重试，local模式并不会重试

7、粗粒度和细粒度

	这个是针对实时计算来说的，spark streaming是一种粗粒度的近似实时。

8、Spark优越性

9、Spark为什么快

	在多个mapreduce 复杂job的情况下， mapreduce shuffle后必须io落盘，但是spark可以将中间数据cache在memory上，另外一个就是spark的内置算子更丰富。另外一个spark的executor只会启动一次，对于task是开线程进行处理的，而mapreduce启动一个task就会启动一次JVM

10、Transformation和action是什么？区别？举几个常用方法

	action算子：**reduce**，**collect**，**count**，**first**，**take**，**top**，**takeOrdered**，**countByKey**，**collectAsMap**，**lookup**，**aggregate**，**fold**，**saveAsFile**，**saveAsSequenceFile**
	
	transformation算子： **map**, **filter**, **groupby**, **flatmap**

11、RDD怎么理解

12、spark 作业提交流程是怎么样的，client和 cluster 有什么区别，各有什么作用

	首先启动driver程序，action触发job，初始化sparkcontext，提交job，DAGscheduler构建有向无环图，然后划分stage，每个stage根据rdd的partition数目决定task个数，产生task集，然后由taskscheduler分配task到executor上执行。

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20161109171133811) 

	DAGScheduler： 实现将Spark作业分解成一到多个Stage，每个Stage根据RDD的Partition个数决定Task的个数，然后生成相应的Task set放到TaskScheduler中。
	
	TaskScheduler：实现Task分配到Executor上执行。

13、spark on yarn 作业执行流程，yarn-client 和 yarn cluster 有什么区别

	我观察到的是yarn-client 和yarn cluster的log输出不一样，yarn client的log更有价值一些，一般调试状态我都用yarn client

#### 								client mode

![client mode](https://img-blog.csdn.net/20170609165010221?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvVHJpZ2w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 								cluster mode

![cluster mode](https://img-blog.csdn.net/20170609183743824?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvVHJpZ2w=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

一般来说，如果提交任务的节点（即Master）和Worker集群在同一个网络内，此时client mode比较合适

如果提交任务的节点和Worker集群相隔比较远，就会采用cluster mode来最小化Driver和Executor之间的网络延迟

14、spark streamning 工作流程是怎么样的，和 storm 比有什么区别

	把Spark Streaming的输入数据按照batch size（如1秒）分成一段一段的数据（DStream），DStream是rdd的模板，代表了一系列连续的rdd。 

![DStreamå°RDDè¿ç¨è§£æ_Python](http://aliyunzixunbucket.oss-cn-beijing.aliyuncs.com/jpg/50312a0982ab0a5b326af632998468d3.jpg?x-oss-process=image/resize,p_100/auto-orient,1/quality,q_90/format,jpg/watermark,image_eXVuY2VzaGk=,t_100,g_se,x_0,y_0) 

	它的算子和rdd一样，其内部generatedRDDs 保存了每个BatchDuration时间生成的RDD对象实例。DStream的依赖构成了RDD依赖关系 

15、spark sql 你使用过没有，在哪个项目里面使用的

	sparksql 的前身是Shark

16、spark 机器学习和 spark 图计算接触过没，能举例说明你用它做过什么吗？

17、spark rdd 是怎么容错的，基本原理是什么？

18.  数据倾斜如何解决？

数据倾斜常见于各种shuffle操作，比如groupby，reducebyKey。因为数据的key本身分布不均匀，或者key的设置不合理。数据倾斜会使得shuffle后并发度不够，有些task的运行时间过长，占用内存过多，导致task失败。

![åçæ°æ®å¾æ](https://img-blog.csdn.net/20160725174623608) 

解决的方式是:

1. hive/ODPS 预处理一下，让数据不倾斜
2. 过滤掉这部分key， 如果需要每次作业执行时，动态判定哪些key的数据量最多然后再进行过滤，那么可以使用sample算子对RDD进行采样，然后计算出每个key的数量，取数据量最多的key过滤掉即可。 
3. 两阶段聚合，第一阶段在有些key上加上随机前缀，第二阶段，再把前缀去掉，然后再聚合

19 . spark-submit 如何指定executor个数？

假设集群有6个Node, 每个Node 有16 核（core), 64G RAM。 

每个node需要1个核和1G内存运行 hadoop Daemons。所以对于每个Node来说只有15个核可用，63G内存可用。

研究发现每个executor最多5个核。所以每个node最多 15 /5 = 3个executor。所以总共设定为 6 * 3 = 18 个executor。

每个executor 的内存分配为 63 /3 = 21G, 另外 spark.yarn.driver.memoryOverhead  一般 为 max(0.07 * spark.executor.memory, 384M), 所以每个executor分配的内存为 21G / 1.07 = 19.6G

20. 每个executor多少内存，是怎么决定的？

Executor占用的内存分为两部分：ExecutorMemory和MemoryOverhead，由这两个参数决定。

**ExecutorMemory**为JVM进程的Java堆区域， **MemoryOverhead**是JVM进程中除Java堆以外占用的空间大小，包括方法区（永久代）、Java虚拟机栈、本地方法栈、JVM进程本身所用的内存、直接内存（Direct Memory）等。 通过spark.yarn.executor.memoryOverhead设置，单位MB。spark.yarn.executor.memoryOverhead ：默认值为executorMemory * 0.10

21. 如何spark写UDAF

    ```scala
    def setupUDAFs(sqlCtx: SQLContext) = {
        class Avg extends UserDefinedAggregateFunction {
        // Input type
        def inputSchema: org.apache.spark.sql.types.StructType =
        StructType(StructField("value", DoubleType) :: Nil)
        def bufferSchema: StructType = StructType(
        StructField("count", LongType) ::
        StructField("sum", DoubleType) :: Nil
        )
        // Return type
        def dataType: DataType = DoubleType
        def deterministic: Boolean = true
        def initialize(buffer: MutableAggregationBuffer): Unit = {
        buffer(0) = 0L
        buffer(1) = 0.0
        }
        def update(buffer: MutableAggregationBuffer,input: Row): Unit = {
        buffer(0) = buffer.getAs[Long](0) + 1
        buffer(1) = buffer.getAs[Double](1) + input.getAs[Double](0)
        }
        def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
        buffer1(0) = buffer1.getAs[Long](0) + buffer2.getAs[Long](0)
        buffer1(1) = buffer1.getAs[Double](1) + buffer2.getAs[Double](1)
        }
        def evaluate(buffer: Row): Any = {
        buffer.getDouble(1) / buffer.getLong(0)
        }
        }
        // Optionally register
        val avg = new Avg
        sqlCtx.udf.register("ourAvg", avg)
    }
    ```

22. spark的checkpoint

    checkpoint 会切除前面算子的rdd 依赖， 而 cache 是将数据暂存在一个具体的位置

    （1）一块是在spark core中对RDD做checkpoint，将RDD数据保存到可靠存储（如HDFS）以便数据恢复；
    通过将计算代价较大的 RDD checkpoint 一下，当下游 RDD 计算出错时，可以直接从 checkpoint 过的 RDD 那里读取数据继续算。
    （2）应用在spark streaming中，使用checkpoint用来保存DStreamGraph以及相关配置信息，以便在Driver崩溃重启的时候能够接着之前进度继续进行处理（如之前waiting batch的job会在重启后继续处理）。

    ```
    sparkContext.setCheckpointDir()
    trainset_df.checkpoint()
    ```