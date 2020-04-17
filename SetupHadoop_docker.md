1.镜像spark1.5 hadoop2.7.7 jdk 1.8 centos 7 hive 1.2.2 hbase 2.1 使用方法
-------------------------

```bash
1.docker run -itd --privileged 名字:v1 /usr/sbin/init  
docker exec -it ....
2.进入之后先source /etc/profile
3.开启hadoop
start-all.sh
jps查看是不是
18465 DataNode
19094 NodeManager
19499 Jps
18700 SecondaryNameNode
18958 ResourceManager
18302 NameNode

4.开启hive 和 hiveserver2
hive --service metastore &
就可以使用hive了
hiveserver2 &
就可以使用beeline -u "jdbc:hive2://127.0.0.1:10000"
jps查看
18465 DataNode
19798 RunJar
19094 NodeManager
19559 RunJar
18700 SecondaryNameNode
20044 Jps
18958 ResourceManager
18302 NameNode
多了两个runjar
5. spark shell
6.start-hbase.sh
然后hbase 
```

安装JDK1.8或者以上版本。这里安装jdk1.6.0_45。 
--------------------------
```bash
下载地址：http://www.oracle.com/technetwork/java/javase/downloads/index.html 
1，下载jdk1.6.0_45-linux-x64.gz，解压到/usr/lib/jdk1.6.0_45。 
2，在/root/.bash_profile中添加如下配置：


export JAVA_HOME=/usr/lib/jdk1.6.0_45
export PATH=$JAVA_HOME/bin:$PATH
3，使环境变量生效，#source ~/.bash_profile 
4，安装验证# java -version 
java version “1.6.0_45” 
Java(TM) SE Runtime Environment (build 1.6.0_45-b06) 
Java HotSpot(TM) 64-Bit Server VM (build 20.45-b01, mixed mode)
```
配置SSH无密码登陆
----------------
```bash

$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
验证ssh，# ssh localhost 
不需要输入密码即可登录。
```
安装mysql
---------------
```bash
wget http://repo.mysql.com/mysql57-community-release-el7-10.noarch.rpm
rpm -Uvh mysql57-community-release-el7-10.noarch.rpm
yum install  -y  mysql-community-server

> vi /etc/my.cnf # 最后一行增加 skip-grant-tables
> service mysqld start
> mysql
mysql > use mysql;
mysql > update user set authentication_string = password("root") where user='root';
mysql > exit;
> vi /etc/my.cnf # 去掉最后一行
> service mysqld stop
> service mysqld start
> mysql -uroot -proot
mysql > alter user 'root'@'localhost' identified by 'root';
mysql > flush privileges;
mysql > exit;

mysql > create user 'hadoop'@'localhost' identified by 'hadoop';
mysql > grant all privileges on *.* to hadoop;
mysql > create database hive;
```


安装Hadoop2.7.7
--------------
```bash
1，下载Hadoop2.7.7
下载地址：http://mirrors.hust.edu.cn/apache/hadoop/common/stable2/hadoop-2.7.7.tar.gz


2，解压安装 
1），复制 hadoop-2.7.7.tar.gz 到/root/hadoop目录下， 
然后#tar -xzvf hadoop-2.7.7.tar.gz 解压，解压后目录为：/root/hadoop/hadoop-2.7.7 
2），在/root /hadoop/目录下，建立tmp、hdfs/name、hdfs/data目录，执行如下命令 
#mkdir /root/hadoop/tmp 
#mkdir /root/hadoop/hdfs 
#mkdir /root/hadoop/hdfs/data 
#mkdir /root/hadoop/hdfs/name


3），设置环境变量，#vi ~/.bash_profile


# set hadoop path
export HADOOP_HOME=/root /hadoop/hadoop-2.6.0
export PATH=$PATH:$HADOOP_HOME/bin
4)，使环境变量生效，$source ~/.bash_profile


3，Hadoop配置 
进入$HADOOP_HOME/etc/hadoop目录，配置 hadoop-env.sh等。涉及的配置文件如下： 
hadoop-2.6.0/etc/hadoop/hadoop-env.sh 
hadoop-2.6.0/etc/hadoop/yarn-env.sh 
hadoop-2.6.0/etc/hadoop/core-site.xml 
hadoop-2.6.0/etc/hadoop/hdfs-site.xml 
hadoop-2.6.0/etc/hadoop/mapred-site.xml 
hadoop-2.6.0/etc/hadoop/yarn-site.xml


1）配置hadoop-env.sh


# The java implementation to use.
#export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME=/usr/lib/jdk1.6.0_45
2）配置yarn-env.sh


#export JAVA_HOME=/home/y/libexec/jdk1.6.0/
export JAVA_HOME=/usr/lib/jdk1.6.0_45
3）配置core-site.xml 
添加如下配置：


<configuration>
 <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
    <description>HDFS的URI，文件系统://namenode标识:端口号</description>
</property>


<property>
    <name>hadoop.tmp.dir</name>
    <value>/root/hadoop/tmp</value>
    <description>namenode上本地的hadoop临时文件夹</description>
</property>
</configuration>
4），配置hdfs-site.xml 
添加如下配置


<configuration>
<!—hdfs-site.xml-->
<property>
    <name>dfs.name.dir</name>
    <value>/root/hadoop/hdfs/name</value>
    <description>namenode上存储hdfs名字空间元数据 </description> 
</property>


<property>
    <name>dfs.data.dir</name>
    <value>/root/hadoop/hdfs/data</value>
    <description>datanode上数据块的物理存储位置</description>
</property>


<property>
    <name>dfs.replication</name>
    <value>1</value>
    <description>副本个数，配置默认是3,应小于datanode机器数量</description>
</property>
</configuration>
5），配置mapred-site.xml 
添加如下配置：


<configuration>
<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
</property>
</configuration>
6），配置yarn-site.xml 
添加如下配置：


<configuration>
<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
</property>
<property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>${yarn.resourcemanager.hostname}:8099</value>
</property>
</configuration>
4，Hadoop启动 
1）格式化namenode


$ bin/hdfs namenode –format
2）启动NameNode 和 DataNode 守护进程


$ sbin/start-dfs.sh
3）启动ResourceManager 和 NodeManager 守护进程


$ sbin/start-yarn.sh
5，启动验证 
1）执行jps命令，有如下进程，说明Hadoop正常启动


# jps
54679 NameNode
54774 DataNode
15741 Jps
9664 Master
55214 NodeManager
55118 ResourceManager
54965 SecondaryNameNode
2）在浏览器中输入：http://datanode-4:8099/ 即可看到YARN的ResourceManager的界面。注意：默认端口是8088，这里我设置了yarn.resourcemanager.webapp.address为：${yarn.resourcemanager.hostname}:8099
```
接下在配置hive
--------------
```bash
1.hive-env.sh
HADOOP_HOME=
HIVE_CONF_DIR=

2.hive-site.xml
<property>
<name>javax.jdo.option.ConnectionUserName</name>
<value>hadoop</value>
</property>
<property>
<name>javax.jdo.option.ConnectionPassword</name>
<value>123!@#qweQWE</value>
</property>
<property>
<name>javax.jdo.option.ConnectionURL</name>
<value>jdbc:mysql://127.0.0.1:3306/hive?createDatabaseIfNotExist=true</value>
</property>
<property>
<name>javax.jdo.option.ConnectionDriverName</name>
<value>com.mysql.jdbc.Driver</value>
</property>
3.初始化metastore

> schematool -dbType mysql -initSchema
```
然后装spark
------------
```bash
先把hive conf里的 hive-site.xml 复制到spark的conf里
1.进入cd spark-2.3.2-bin-hadoop2.7/conf，修改文件
cp conf/spark-env.sh.template conf /spark-env.sh
cp conf/slaves.template conf/slaves
2.打开修改spark-env.sh文件

export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export CLASSPATH=$CLASSPATH:/usr/local/hive/lib
export SCALA_HOME=/usr/local/scala
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export HIVE_CONF_DIR=/usr/local/hive/conf
export SPARK_CLASSPATH=$SPARK_CLASSPATH:/usr/local/hive/lib/mysql-connector-java-5.1.40-bin.jar

JAVA_HOME：Java安装目录 
SCALA_HOME：Scala安装目录 
SPARK_MASTER_IP：spark集群的Master节点的ip地址 
SPARK_WORKER_MEMORY：每个worker节点能够最大分配给exectors的内存大小 
SPARK_WORKER_CORES：每个worker节点所占有的CPU核数目 
SPARK_WORKER_INSTANCES：每台机器上开启的worker节点的数目

spark-shell里试一试能不能连上hive:
Scala> import org.apache.spark.sql.Row
Scala> import org.apache.spark.sql.SparkSession
 
Scala> case class Record(key: Int, value: String)
 
// warehouseLocation points to the default location for managed databases and tables
Scala> val warehouseLocation = "spark-warehouse"
 
Scala> val spark = SparkSession.builder().appName("Spark Hive Example").config("spark.sql.warehouse.dir", warehouseLocation).enableHiveSupport().getOrCreate()
 
Scala> import spark.implicits._
Scala> import spark.sql
//下面是运行结果
scala> sql("SELECT * FROM sparktest.student").show()


3.修改slaves文件
加入localhost
```
安装zookeeper
------------
```bash
1.tar -zxvf zookeeper-3.4.13.tar.gz -C /app
2.复制配置文件并修改名称为zoo.cfg

# mv zoo_sample.cfg zoo.cfg
3.试一下能不能开
bin/zkServer.sh
然后关了。不然好像hbase开不了
```

安装hbase
----------
```bash
1.tar -zxvf hbase-2.1.0-bin.tar.gz /app
2.安装单机版很简单，我们只需要配置JDK的路径即可，我们将JDK的路径配置到conf/下的hbase.env.sh中
3.编辑hbase-site.xml文件，在<configuration>标签中添加如下内容
<property>
  <property>
       <name>hbase.rootdir</name>
       <value>hdfs:///home/xlc/hbase</value>
  </property>
  <property>
       <name>hbase.zookeeper.property.dataDir</name>
       <value>/home/xlc/zookeeper</value>
  </property>
  <property>
        <name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
  </property>
</property>

4.# SET HBASE_enviroment 

HBASE_HOME=/app/hbase-2.1.0
export PATH=$PATH:$HBASE_HOME/bin

5.start-hbase.sh
有HMaster
hbase shell进入
```
可能出现的问题
----------------
```bash
park查询hive表时可能会出现找不到表或者视图的情况，该情况是由于spark未知hive的配置。

解决方案为：

在hive的配置目录conf下，找到hive的配置文件hive-site.xml

将该文件复制到spark的conf目录下

重新运行spark shell或者通过spark-submit运行文件

```


```bash
新版镜像多了spark为2.1.1，spark版本可以随时变换，只需要修改配置文件，但是1.*和2.*需要修改hive的配置文件不一样
 spark升级到spark2以后，原有lib目录下的大JAR包被分散成多个小JAR包，原来的spark-assembly-*.jar已经不存在，所以hive没有办法找到这个JAR包。

解决办法是：修改bin目录下的hive文件

1 vim hive
找到下面内容

# add Spark assembly jar to the classpath
if [[ -n "$SPARK_HOME" ]]
then
  sparkAssemblyPath=`ls ${SPARK_HOME}/lib/spark-assembly-*.jar`
  CLASSPATH="${CLASSPATH}:${sparkAssemblyPath}"
fi
将紫色部分改为:sparkAssemblyPath=`ls ${SPARK_HOME}/jars/*.jar`
```
2.ubuntu
----------------------
```bash
和centos不太一样
1.假如忘记mysql密码
ps -ef | grep -i mysql  //检查mysql是否运行

service mysqld stop
修改配置文件 /etc/my.cnf
最后加一行skip-grant-tables
重启服务 service mysqld start
进入  mysql -uroot
不用密码了

然后修改密码
use mysql;
update mysql.user set authentication_string=password('root_password') where user='root';
flush privileges;
exit
把刚刚加的最后一行删除
重启服务就好了
```
