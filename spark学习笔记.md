# 1. spark学习笔记总结

## 1.1. 什么是RDD？

1、RDD是Spark提供的核心抽象，全称为ResillientDistributed Dataset，即**弹性**分布式数据集。

2、RDD在抽象上来说是一种元素集合，包含了数据。它是被分区的，分为多个分区，每个分区分布在集群中的不同节点上，从而让RDD中的数据可以被并行操作。（分布式数据集）

3、RDD通常通过Hadoop上的文件，即HDFS文件或者Hive表，来进行创建；有时也可以通过应用程序中的集合来创建。

4、RDD最重要的特性就是，提供了容错性，可以自动从节点失败中恢复过来。即如果某个节点上的RDDpartition，因为节点故障，导致数据丢了，那么RDD会自动通过自己的数据来源重新计算该partition。这一切对使用者是透明的。

5、RDD的数据默认情况下存放在内存中的，但是在内存资源不足时，Spark会自动将RDD数据写入磁盘。（弹性 ==灵活）

![什么是RDD](/Users/apple/Idea/workspace/FAQ/image/什么是RDD.png)

## 1.2. Spark架构

![spark架构](/Users/apple/Idea/workspace/FAQ/image/spark架构.png)

## 1.3. Historyserver配置



如果spark记录下了一个作业生命周期内的所有事件，那么就会在该作业执行完成之后，我们进入其webui时，自动用记录的数据

重新绘制作业的web ui。

有3个属性我们可以设置

**spark-defaults.conf**

[spark.eventLog.enabled  true]()

spark.eventLog.dir      hdfs://192.168.32.110:9000/spark-events

spark.eventLog.compress true

**spark-env.sh**

[exportSPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080-Dspark.history.retainedApplications=250-Dspark.history.fs.logDirectory=hdfs://192.168.32.110:9000/spark-events"]()

务必预先创建好hdfs://192.168.0.103:9000/spark-events目录

而且要注意，spark.eventLog.dir与spark.history.fs.logDirectory指向的必须是同一个目录

因为spark.eventLog.dir会指定作业事件记录在哪里，spark.history.fs.logDirectory会指定从哪个目录中去读取作业数据

启动HistoryServer: ./sbin/start-history-server.sh

访问地址: 192.168.0.103:18080



### 1.3.1. RDD的创建方式

   进行Spark核心编程时，首先要做的第一件事，就是创建一个初始的RDD。该RDD中，通常就代表和包含了Spark应用程序的输入源数据。然后在创建了初始的RDD之后，我们接着进行各种算子操作。

大体上有两种方式创建RDD：

### 第一种方式：通过读取文件

A:通过读取HDFS上文件，创建RDD。  sc.textFile(“hdfs://”) 

B:通过读取本地文件，创建RDD  sc.textFile(“”);

### 第二种方式：通过并行化的方式创建RDD

其实这种方式就是通过我们自己去模拟数据

val str=Array(“you      jump”,”i     jumps”)

val list=Array(1,2,3,4,5,6)

val listrdd=sc.parallelize(list);

### 1.3.2. Transformation和action原理

Spark支持两种RDD操作：transformation和action。transformation操作会针对已有的RDD创建一个新的RDD；而action则主要是对RDD进行最后的操作，比如遍历、reduce、保存到文件等，并可以返回结果给Driver程序。

例如，map就是一种transformation操作，它用于将已有RDD的每个元素传入一个自定义的函数，并获取一个新的元素，然后将所有的新元素组成一个新的RDD。而reduce就是一种action操作，它用于对RDD中的所有元素进行聚合操作，并获取一个最终的结果，然后返回给Driver程序。

transformation的特点就是lazy特性。lazy特性指的是，如果一个spark应用中只定义了transformation操作，那么即使你执行该应用，这些操作也不会执行。也就是说，transformation是不会触发spark程序的执行的，它们只是记录了对RDD所做的操作，但是不会自发的执行。只有当transformation之后，接着执行了一个action操作，那么所有的transformation才会执行。Spark通过这种lazy特性，来进行底层的spark应用执行的优化，避免产生过多中间结果。

action操作执行，会触发一个spark job的运行，从而触发这个action之前所有的transformation的执行。这是action的特性。![transformation和action原理剖析](/Users/apple/Idea/workspace/FAQ/image/transformation和action原理剖析.png)

### 1.3.3. map，filter，flatMap算子

Map:

对调用map的RDD数据集中的每个element都使用func，然后返回一个新的RDD,这个返回的数据集是分布式的数据集

Filter:

对调用filter的RDD数据集中的每个元素都使用func，然后返回一个包含使func为true的元素构成的RDD

flatMap:

和map差不多，但是flatMap生成的是多个结果



## 1.4. join[,cogroup](),union算子

- Cogroup:[这个实现根据两个要进行合并的两个RDD操作,生成一个CoGroupedRDD的实例,这个RDD的返回结果是把相同的key中两个RDD分别进行合并操作,最后返回的RDD的value是一个Pair的实例,这个实例包含两个Iterable的值,第一个值表示的是RDD1中相同KEY的值,第二个值表示的是RDD2中相同key的值.]。

## 1.5. sample, Aggregate,aggregateBykey算子

Sample:

 [对RDD中的集合内元素进行采样，第一个参数withReplacement是true表示有放回取样，false表示无放回。第二个参数表示比例]

 Aggregate:即聚合操作

```
import org.apache.spark.{SparkConf, SparkContext}

object AggregateTest {

  def main(args:Array[String]) = {

    // 设置运行环境
    val conf = new SparkConf().setAppName("Aggregate Test").setMaster("spark://master:7077").setJars(Seq("E:\\Intellij\\Projects\\SimpleGraphX\\SimpleGraphX.jar"))
    val sc = new SparkContext(conf)

    var data = List(2,5,8,1,2,6,9,4,3,5)
    var res = data.par.aggregate((0,0))(
      // seqOp
      (acc, number) => (acc._1+number, acc._2+1),
      // combOp
      (par1, par2) => (par1._1+par2._1, par1._2+par2._2)
    )

    println(res)

    sc.stop
  }

}

```

acc即(0,0)，number即data，seqOp将data的值累加到Tuple的第一个元素，将data的个数累加到Tuple的第二个元素。由于没有分区，所以combOp是不起作用的，这个例子里面即使分区了，combOp起作用了，结果也是一样的

运行结果：

```
(45,10)
```

aggregateByKey算子操作原理

![aggregateByKey原理剖析](/Users/apple/Idea/workspace/FAQ/image/aggregateByKey原理剖析.png)

aggregateByKey在kv对的RDD中，，按key将value进行分组合并，合并时，将每个value和初始值作为seq函数的参数，进行计算，返回的结果作为一个新的kv对，然后再将结果按照key进行合并，最后将每个分组的value传递给combine函数进行计算（先将前两个value进行计算，将返回结果和下一个value传给combine函数，以此类推），将key与计算结果作为一个新的kv对输出。

```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by Administrator on 2017/6/13.
  */
object AggregateByKeyTest {

  def main(args:Array[String]) = {

    // 设置运行环境
    val conf = new SparkConf().setAppName("AggregateByKey Test").setMaster("spark://master:7077").setJars(Seq("E:\\Intellij\\Projects\\SimpleGraphX\\SimpleGraphX.jar"))
    val sc = new SparkContext(conf)

    val data = List((1,3),(1,2),(1,4),(2,3),(3,6),(3,8))
    val rdd = sc.parallelize(data)

    val res : RDD[(Int,Int)] = rdd.aggregateByKey(0)(
      // seqOp
      math.max(_,_),
      // combOp
      _+_
    )

    res.collect.foreach(println)
    sc.stop
  }

}
```



根据Key值的不同，可以分为3个组：

(1)  (1,3),(1,2),(1,4)；

(2)  (2,3)；

(3)  (3,6),(3,8)。

这3个组分别进行seqOp，也就是(K,V)里面的V和0进行math.max()运算，运算结果和下一个V继续运算，以第一个组为例，运算过程是这样的：

0, 3 => 3

3, 2 => 3

3, 4 => 4

所以最终结果是(1,4)。combOp是对把各分区的V加起来，由于这里并没有分区，所以实际上是不起作用的。

运行结果：

```
(2,3)
(1,4)
(3,8)
```

如果生成RDD时分成3个区：

```
val rdd = sc.parallelize(data,3)
```

运行结果就变成了：

```
(3,8)
(1,7)
(2,3)
```

这是因为一个分区返回(1,3)，另一个分区返回(1,4)，combOp将这两个V加起来，就得到了(1,7)。



## 1.6. RDD持久化

RDD持久化

将数据通过操作持久化（或缓存）在内存中是Spark的重要能力之一。当你缓存了一个RDD，每个节点都缓存了RDD的所有分区。这样就可以在内存中进行计算。这样可以使以后在RDD上的动作更快（通常可以提高10倍）。

你可以对希望缓存的RDD通过使用persist或cache方法进行标记。它通过动作操作第一次在RDD上进行计算后，它就会被缓存在节点上的内存中。Spark的缓存具有容错性，如果RDD的某一分区丢失，它会自动使用最初创建RDD时的转换操作进行重新计算。

另外，RDD可以被持久化成不同的级别。比如，可以允许你存储在磁盘，内存，甚至是序列化的**Java**对象（节省空间），备份在不同的节点上，或者存储在基于内存的文件系统Tachyon上。通过向persist()方法传递StorageLevel对象来设置。cache方法是使用默认级别[`StorageLevel`]()`.MEMORY_ONLY`的方法。

选持久化方案建议：

1：优先选择[MEMORY_ONLY]()，如果可以用内存缓存所有的数据，那么也就意味着我的计算是纯内存的计算，速度当然快。

2：MEMORY_ONLY 缓存不了所有的数据，MEMORY_ONLY_SER 把数据实现序列化然后进行存储。这样也是纯内存操作，速度也快，只不过需要耗费一点cpu资源需要反序列化。

3：可以选用带2这种方式。恢复速度的时候可以使用备份。不需要重新计算

4：能不能使用DISK的，就不使用DISK，有时候从磁盘读，还不如从新计算一次。



### 1.7. 共享变量（广播变量，累加变量）

![广播变量原理](/Users/apple/Idea/workspace/FAQ/image/广播变量原理.png)



![累加变量](/Users/apple/Idea/workspace/FAQ/image/累加变量.png)



## 1.8	Spark on YARN模式（cluster,client）

```

./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn \
  --deploy-mode cluster \  # can be client for client mode
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000
Mapreduce在YARN上面的运行的详细过程

```





![yarn-cluster](/Users/apple/Idea/workspace/FAQ/image/yarn-cluster.png)

![yarn运行任务的过程](/Users/apple/Idea/workspace/FAQ/image/yarn运行任务的过程.png)

 

建议：

1：调试程序的时候，建议使用client模式。使用client模式的时候打印出来的信息非常

```
详细，有利于我们调试程序。
2：如果我们调试完成以后，建议使用cluster模式提交任务。
```

## 1.9 窄依赖和宽依赖

在RDD中将依赖分成了两种类型：窄依赖和宽依赖，**窄依赖是指父****RDD的每个分区都只被子RDD一个分区使用**。相应的，那么宽依赖就是指**父RDD的分区被多个子RDD的分区所依赖。**

## 2.0 spark 依赖包冲突的问题
job.spark.config=--conf spark.driver.extraClassPath=guava-18.0.jar,jsr166e-1.1.0.jar,t-digest-3.0.jar,hppc-0.7.1.jar  --conf spark.executor.extraClassPath=guava-18.0.jar,jsr166e-1.1.0.jar,t-digest-3.0.jar,hppc-0.7.1.jar --conf spark.driver.userClassPathFirst=true





