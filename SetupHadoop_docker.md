```bash
安装JDK1.6或者以上版本。这里安装jdk1.6.0_45。 
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


三，配置SSH无密码登陆


$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
验证ssh，# ssh localhost 
不需要输入密码即可登录。


四，安装Hadoop2.6 
1，下载Hadoop2.6 
下载地址：http://mirrors.hust.edu.cn/apache/hadoop/common/stable2/hadoop-2.6.0.tar.gz


2，解压安装 
1），复制 hadoop-2.6.0.tar.gz 到/root/hadoop目录下， 
然后#tar -xzvf hadoop-2.6.0.tar.gz 解压，解压后目录为：/root/hadoop/hadoop-2.6.0 
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
