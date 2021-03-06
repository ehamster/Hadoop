关于Spark
==============
命令
------------------
```bash
awk	"Aho, Weinberger and Kernigan", Bell Labs, 1970s. Interpreted programming language for text processing.
awk -F	(see above) + Set the field separator.
cat	Display the contents of a file at the command line, is also used to copy and or append text files into a document. Named after its function to con-cat-enate files.
cd	Change the current working directory. Also known as chdir (change directory).
cd /	Change the current directory to root directory.
cd ..	Change the current directory to parent directory.
cd ~	Change the current directory to your home directory.
cp	Make copies of files and directories.
cp -r	Copy directories recursively.
cut	Drop sections of each line of input by bytes, characters, or fields, separated by a delimiter (the tab character by default).
cut -d -f	-d is for delimiter instead of tab character, -f select only those fields (ex.: “cut -d “,“ -f1 multilined_file.txt” - will mean that we select only the first field from each comma-separated line in the file)
du	Estimate (and display) the file space usage - space used under a particular directory or files on a file system.
df	Display the amount of available disk space being used by file systems.
df -h	Use human readable format.
free	Display the total amount of free and used memory (use vm_stat instead on MacOS).
free -m	Display the amount of memory in megabytes.
free -g	Display the amount of memory in gigabytes.
grep	Process text and print any lines which match a regular expression ("global regular expression print")
head	Print the beginning of a text file or piped data. By default, outputs the first 10 lines of its input to the command line.
head -n	Output the first n lines of input data (ex.: “head -5 multilined_file.txt”).
kill	Send a signal to kill a process. The default signal for kill is TERM (which will terminate the process).
less	Is similar to more, but has the extended capability of allowing both forward and backward navigation through the file.
ls	List the contents of a directory.
ls -l	List the contents of a directory + use a long format, displaying Unix file types, permissions, number of hard links, owner, group, size, last-modified date and filename.
ls -lh	List the contents of a directory + print sizes in human readable format. (e.g. 1K, 234M, 2G, etc.)
ls -lS	Sort by file size
man	Display the manual pages which provide documentation about commands, system calls, library routines and the kernel.
mkdir	Create a directory on a file system ("make directory")
more	Display the contents of a text file one screen at a time.
mv	Rename files or directories or move them to a different directory.
nice	Run a command with a modified scheduling priority.
ps	Provide information about the currently running processes, including their process identification numbers (PIDs) ("process status").
ps a	Select all processes except both session leaders and processes not associated with a terminal.
pwd	Abbreviated from "print working directory", pwd writes the full pathname of the current working directory.
rm	Remove files or directories.
rm -r	Remove directories and their contents recursively.
sort	Sort the contents of a text file.
sort -r	Sort the output in the reverse order. Reverse means - to reverse the result of comparsions
sort -k	-k or --key=POS1[,POS2] Start a key at POS1 (origin 1), end it at POS2 (default end of the line) (ex.: “sort -k2,2 multilined_file.txt”).
sort -n	Compare according to string numerical value.
tail	Print the tail end of a text file or piped data. Be default, outputs the last 10 lines of its input to the command line.
tail -n	Output the last n lines of input data (ex.: “tail -2 multilined_file.txt”).
top	Produce an ordered list of running processes selected by user-specified criteria, and updates it periodically.
touch	Update the access date and or modification date of a file or directory or create an empty file.
tr	Replace or remove specific characters in its input data set ("translate").
tr -d	Delete characters, do not translate.
vim	Is a text editor ("vi improved"). It can be used for editing any kind of text and is especially suited for editing computer programs.
wc	Print a count of lines, words and bytes for each input file ("word count")
wc -c	Print only the number of characters.
wc -l	Print only the number of lines.
```

Hadoop Distributed Files System write files
-----------------------
```bash
1. client request , name node validate whether you have right
2. client requests a list of datanode to put a fraction of files
3. client sends packet of files to closest data node
4. later data node on the pipeline will transfer copy of the packet
5. the packet stores in all datanode, data node send acknowledge packet back
6. if one data node wrong. mark it bad and organize the pipeline again
```
Block and replica
--------------------------
```bash
Block in name node contain info about replica
replica in data node
Replica states:
1. Finalized: replica的写入完成了
2.RBW : replica being written to，replica打开文件的最后一个block
block of file is appending : NameNode信息可能和DataNode里不match
3.RWR：
RWR ： replica Waitting to be Recovered
写入过程中datanode挂了，replica会从 RBW -> RWR，说明数据需要被恢复
4.RUR Replica under Reconvery Replica正在恢复
5.Temporary： 临时的replica会因为复制或者集群平衡而存在，若复制失败，所在DataNode会重启，temporary Replica被删除。
对client不可见
```

Block State:
------------------
* 1. Under_ Construction
  Create or reopen block, last block of the file
* 2.Under_Recovery:
  file lease expire, Recovery开始后,Block: Under_contruction -> Under_Recovery
* 3. Committed:
  When a client successfuly request close a file or create a new block,
一些replica已经是finalized 不是所有的
* 4.Complete：
  All replicas are in Finalized. All block of a file are completed. can close the file.

Recovery process
------------------
```bash
1.Block Recovery:
2.Lease Recovery:
Block Recovery是Lease Recovery一部分
获取包含文件最后一个 block 的所有 DataNodes。
指定其中一个 DataNode 作为主导恢复的节点。
主导节点向其他节点请求获得它们上面存储的 replica 信息。
主导节点收集了所有节点上的 replica 信息后，就可以比较计算出各节点上不同 replica 的最小长度。
主导节点向其他节点发起更新，将各自 replica 更新为最小长度值，保持各节点 replica 长度一致。
所有 DataNode 都同步后，主导节点向 NameNode 报告更新一致后的最终结果。
NameNode 更新文件 block 元数据信息，收回该文件租约，并关闭文件。
其中 3～6 步就属于 block recovery 的处理过程

3.replica recovery
4.pipeline recovery
```
pipeline 写入包括三个阶段：
---------------------------
```bash
pipeline setup：Client 发送一个写请求沿着 pipeline 传递下去，最后一个 DataNode 收到后发回一个确认消息。Client 收到确认后，pipeline 设置准备完毕，可以往里面发送数据了。
data streaming：Client 将一个 block 拆分为多个 packet 来发送（默认一个 block 64MB，太大所以需要拆分）。Client 持续往 pipeline 发送 packet，在收到 packet ack 之前允许发送 n 个 packet，n 就是 Client 的发送窗口大小（类似 TCP 滑动窗口）。
close：Client 在所有发出的 packet 都收到确认后发送一个 Close 请求， 
pipeline 上的 DataNode 收到 Close 后将相应 replica 修改为 FINALIZED 状态，并向 NameNode 发送 block 报告。NameNode 将根据报告的 FINALIZED 状态的 replica 数量是否达到最小副本要求来改变相应 block 状态为 COMPLETE。
Pipeline recovery 可以发生在这三个阶段中的任意一个，只要在写入过程中一个或多个 DataNode 遭遇网络或自身故障。我们来分别分析下。


从 pipeline setup 错误中恢复
在 pipeline 准备阶段发生错误，分两种情况：

新写文件：Client 重新请求 NameNode 分配 block 和 DataNodes，重新设置 pipeline。
追加文件：Client 从 pipeline 中移除出错的 DataNode，然后继续。
从 data streaming 错误中恢复
当 pipeline 中的某个 DataNode 检测到写入磁盘出错（可能是磁盘故障），它自动退出 pipeline，关闭相关的 TCP 连接。
当 Client 检测到 pipeline 有 DataNode 出错，先停止发送数据，并基于剩下正常的 DataNode 重新构建 pipeline 再继续发送数据。
Client 恢复发送数据后，从没有收到确认的 packet 开始重发，其中有些 packet 前面的 DataNode 可能已经收过了，则忽略存储过程直接传递到下游节点。
从 close 错误中恢复
到了 close 阶段才出错，实际数据已经全部写入了 DataNodes 中，所以影响很小了。Client 依然根据剩下正常的 DataNode 重建 pipeline，让剩下的 DataNode 继续完成 close 阶段需要做的工作。

以上就是 pipeline recovery 三个阶段的处理过程，这里还有点小小的细节可说。 
当 pipeline 中一个 DataNode 挂了，Client 重建 pipeline 时是可以移除挂了的 DataNode，也可以使用新的 DataNode 来替换。这里有策略是可配置的，称为 DataNode Replacement Policy upon Failure，包括下面几种情况：

NEVER：从不替换，针对 Client 的行为
DISABLE：禁止替换，DataNode 服务端抛出异常，表现行为类似 Client 的 NEVER 策略
DEFAULT：默认根据副本数要求来决定，简单来说若配置的副本数为 3，如果坏了 2 个 DataNode，则会替换，否则不替换
ALWAYS：总是替换
```

H D F S commands
-------------------------
```bash
1.hdfs dfs -help
2.hdfs dfs -usage<name>
3.hdfs dfs -du -h /data/wiki  查看file大小和所有replica消耗大小
4.hdfs dfs -df -h /data/wiki  分布系统剩余空间
5.hdfs dfs -mkdir -p deep/nested/path
6.hdfs dfs -rm -r deep
7.hdfs dfs -touchz a.txt
8.local -> HDFS
hdfs dfs -put <source location> <HDFS>
9. hdfs dfs -cat a.txt | head -4
hdfs dfs -cat a.txt | tail -4
10. hdfs dfs -cp a.txt ab.txt
11. hdfs dfs -get a*
12. hdfs dfs -getmerge a* b.txt 变成一个local file
13. 复制：
hdfs dfs -setrep -w 1 a.txt  /复制一次
14.hdfs fsck /data/wiki -files -blocks -locations
15.hdfs dfs -find /data/wiki -name "*part*"
```

scala
–––––
```bash
1.string.strip

等于scala的 s.trim()

2.根据list过滤dataframe
df=df.filter(df.col("home").isin(alist:_*))
如果要实现not in.加感叹号
df=df.filter(!df.col("home").isin(alist:_*))
3.合并df,他只是按照列的顺序，不管列名一不一样

df=df.unionAll(df2)

4.查看进度

yarn application -list
```

5.写出成分区表
不要用saveastable和它生成的表
用这个,先人工建表。
df.write.partitionBy("cp","ld").insertInto("tb")

6.时间戳转日期

df=df.withColumn("a",from_unixtime(df("a"),"yyyy–MM–dd HH:mm:ss"))

7.column重命名
df=df.drop("a").withColumnRenamed("b","a")

8.改变列的顺序
val column:Array[String] = df.columns
val reorder:Array[String] = Array("a","c","b")
df = df.select(reorder.head,reorder.tail:_*)

9.spark-shell --master yarn
