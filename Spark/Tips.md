```bash
Docker： 启动container：  sudo docker run -t -i IMAGEPORT
list变为DS scala> val ds = List("abcdef", "abcd", "cdef", "mnop").toDS
0.当需要做group的时候，reduceByKey 比 groupby更好，优先配合pair rdd使用
0.1将rdd转为pairrdd
val pair_rdd = df3.rdd.map(row => (row.getString(0),row.getString(1)))
//df里就俩字段 都是 string 所以这么写
0.2 根据第一个字段，group 第二个字段 
val rdd2 = pair_rdd.reduceByKey((x,y) => x + "\n" + y)

1. 读取文件作为DataSet
scala> val textFile = spark.read.textFile("README.md")

2. Filter 函数   是一种 transformer
匿名函数的一种写法，因为在scala中函数可以作argument：
  因为这里filter的参数要求是 (Int => Boolean)
    val x = t.filter(argument => function body)
  
    所以可以这么写
    val x = t.filter(e => e%2 == 0)  #t是DataSet
                                     #选取符合的数

3. map 函数 是一种 transformer
也是匿名函数写法
假设map函数要求的 parameter 是(String => Int)   go from List[A] => List[B]. This is called lifting
val s : List[String] = List("abc", "a","ab")
val i : List[Int] = s.map(x => x.length)
上面是最简单的写法，你也可以写成
s.map((x : String) => x.length)

或者

val sl : (String => Int) = {
  x => x.length
}
val i = s.map(sl)

4. reduce function (op function)  (A, A) => A

List( 1, 2, 3, 4 ).reduce( (x,y) => x + y )
TextFile
 "hello Tyth"
 "cool example, eh?"
 "goodbye"

TextFile.map(line => line.split(" ").size)
 2
 3
 1
TextFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)  

5.flatMap()
Map transforms an RDD of length N into another RDD of length N. 
It is similar to Map, but FlatMap allows returning 0, 1 or more elements from map function.

flatMap() 
we call flatMap to transform a Dataset of lines to a Dataset of words
t2 = t.flatMap(line => line.split(""))



6.groupByKey（）  
t.flatMap(line => line.split(" ")).groupByKey(l=>l。substring(0,3)).count().collect()

以所有words前计算所有word前三个字母为key，统计个数。
反正就是类似的作用~~


Caching:  重复运行的放进去跑

      t.cache()   #之后跑就快了？？
      
      
7.从hive里面读取数据，得到的dataframe。dataframe和rdd都可以用rdd的transformer和action

val conf = new SparkConf().setAppName("SPARKHIVE").setMaster("local[*]")
val sc = new SparkContext(conf)
val hiveContext = new HiveContext(sc)
var df = hiveContext.sql("select msisdn,concat_ws('|',collect_set(col1),collect_set(col2)) as values from tb_input2 
where msisdn is not NULL group by msisdn")
//这里collect_set是会收集全部行，想不重复可以选其他的。
//concat_ws 类似于group_concat   用于合并多行

8.Ice接受mappartitions的数据问题:

一次action就会执行所有transformation操作，所以小心不要执行多次action
如果数据少，val processedDf = df.repartition(1).mapPartitions(sendNlp)
加一个repartition(1)设置partition为1个，这样就不会有空partition也穿到服务端了



9.将RDD存入hive  
1)先将rDD转被为DF
import org.apache.spark.sql.types.{StructField, StructType}

import hiveContext.implicits._
rdd.cache() //前面的transformer操作不需要再做了，已经将rdd存在内存了
val newrdd = rdd.map(x => Row(x.val1, x.val2))
val schemaString = "msisdn,value"
val fields = schemaString.split(",").map(fieldName => StructField(fieldName, StringType, nullable = true))
val schema = StructType(fields)
var peopleDF = hiveContext.createDataFrame(newrdd, schema)
peopleDF.show()
createDataFrame属于sqlContext，而hiveContext继承了SqlContext

2)将df输出为表
两种方法：a. peopleDF.registerTemTable("tmp_input")
            hiveContext.sql("create table tb_input_final2 as select * from tmp_input")
            
            
        b. peopleDF.write.mode(SaveMode.Append).saveAsTable("tb_input_final")
        
因为只有要输出内容得到数值的操作都是action,所以上面两种都会执行一边之前的transformation操作

为了避免`，可以将要用到的rdd.cache()，存到内存里，就不需要执行前面的操作了

关于spark join 表格

1.读取两个表
var df = hiveContext.sql("select * from tb1")
2.join先广播小表
val df2 = sc.broadcast(df2).value
var result = df.join(df2, df("numbers")=== df2("n"))
3.去重复列
result = result.drop(df2("n"))

4.如果是想合并列的话
val separator = ","
result.select(df("numbers",concat_ws(separator, df("value"), df2("value")).cast(StringType).as("values")).show()

```
关于spark资源问题
----
```bash
0.默认情况下，yarn 可分配核 = 机器核 x 1.5，yarn 可分配内存 = 机器内存 x 0.8

1.partition数量最好是 num_exectors和executor_cores的倍数
因为两个变量乘等于能同时处理的分区数量，为了不浪费资源，最好是倍数关系

2. 可以先用这个查看表的大小
show create table
取得地址然后查看大小
hadoop fs -du -s -h /user/hive/warehouse/
 
3.各种参数
2种模式： yarn-client用于调试  yarn-cluster用于生产环境
项目提交的机器和worker是物理连接的时候用client
远程连接的时候用cluster
区别在于driver运行的位置，是和spark-submit那个在一起还是在cluster里

driver-memory - driver使用的内存
num-executors - 创建多少个executors
executor-memory - 单个executor最大内存
executor-cores - 各个 executor 使用的并发线程数目，也即每个 executor 最大可并发执行的 Task 数目


```
