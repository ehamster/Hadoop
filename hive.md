Hive基础操作
==============================
创建 DB
-----------------------
```bash
CREATE DATABASE [IF NOT EXISTS] userdb;

CREATE SCHEMA userdb;

SHOW DATABASES;
```
删除DB
------------------------
DROP database if exists userdb cascade;  //加casade后db有表也会删除

使用某一个db
------------------------
use userdb;

创建普通表：
-----------------------
```bash
create table if not exists userinfo
(
	userid int,
	username string,
	cityid bigint,
	createtime date
)
row format delimited fields terminated by '\t'
stored as textfile;

//row format delimited fields terminated by '\t' 是指定列之间的分隔符
//stored as textfile是指定文件存储格式为textfile。
```
创建表一般有几种方式：
-------------------------
create table 方式：以上例子中的方式。

create table as select 方式：根据查询的结果自动创建表，并将查询结果数据插入新建的表中。

create table like tablename1 方式：是克隆表，只复制tablename1表的结构。复制表和克隆表会在下面的Hive数据管理部分详细讲解。


外部表：
------------------------------
外部表是没有被hive完全控制的表，当表删除后，数据不会被删除。

hive> create external table iislog_ext (
    >  ip string,
    >  logtime string    
    > )
    > ;

创建分区表：
------------------------
Hive查询一般是扫描整个目录，但是有时候我们关心的数据只是集中在某一部分数据上，比如我们一个Hive查询，往往是只是查询某一天的数据，这样的情况下，可以使用分区表来优化，一天是一个分区，查询时候，Hive只扫描指定天分区的数据。

普通表和分区表的区别在于：一个Hive表在HDFS上是有一个对应的目录来存储数据，普通表的数据直接存储在这个目录下，而分区表数据存储时，是再划分子目录来存储的。一个分区一个子目录。主要作用是来优化查询性能。

--创建经销商操作日志表
```bash
create table user_action_log
(
companyId INT comment   '公司ID',
userid INT comment   '销售ID',
originalstring STRING comment   'url', 
host STRING comment   'host',
absolutepath STRING comment   '绝对路径',
query STRING comment   '参数串',
refurl STRING comment   '来源url',
clientip STRING comment   '客户端Ip',
cookiemd5 STRING comment   'cookiemd5',
timestamp STRING comment   '访问时间戳'
)
partitioned by (dt string)
row format delimited fields terminated by ','
stored as textfile;
```
这个例子中，这个日志表以dt字段分区，dt是个虚拟的字段，dt下并不存储数据，而是用来分区的，实际数据存储时，dt字段值相同的数据存入同一个子目录中，插入数据或者导入数据时，同一天的数据dt字段赋值一样，这样就实现了数据按dt日期分区存储。

当Hive查询数据时，如果指定了dt筛选条件，那么只需要到对应的分区下去检索数据即可，大大提高了效率。所以对于分区表查询时，尽量添加上分区字段的筛选条件。

创建桶表
------------------------
桶表也是一种用于优化查询而设计的表类型。创建通表时，指定桶的个数、分桶的依据字段，hive就可以自动将数据分桶存储。查询时只需要遍历一个桶里的数据，或者遍历部分桶，这样就提高了查询效率。举例：

------创建订单表
```bash
create table user_leads
(
leads_id string,
user_id string,
user_phone string,
user_name string,
create_time string
)
clustered by (user_id) sorted by(leads_id) into 10 buckets 
row format delimited fields terminated by '\t' 
stored as textfile;
```
对这个例子的说明：

clustered by是指根据userid的值进行哈希后模除分桶个数，根据得到的结果，确定这行数据分入哪个桶中，这样的分法，可以确保相同userid的数据放入同一个桶中。而经销商的订单数据，大部分是根据user_id进行查询的。这样大部分情况下是只需要查询一个桶中的数据就可以了。

sorted by 是指定桶中的数据以哪个字段进行排序，排序的好处是，在join操作时能获得很高的效率。

into 10 buckets是指定一共分10个桶。

在HDFS上存储时，一个桶存入一个文件中，这样根据user_id进行查询时，可以快速确定数据存在于哪个桶中，而只遍历一个桶可以提供查询效率。


查询表：
---------------
查询库中表
```bash
show tables;
Show TABLES '*info';  --可以用正则表达式筛选要列出的表
describe userinfo; -- 查看表定义
describe formatted userinfo -- 查看表详情


修改表名

alter table userinfo rename to user_info;


添加字段：
alter table user_info add columns (provinceid int );

修改字段：
alter table user_info replace columns (userid int, username string);
修改字段，只是修改了Hive表的元数据信息（元数据信息一般是存储在MySql中），并不对存在于HDFS中的表数据做修改。

并不是所有的Hive表都可以修改字段，只有使用了native SerDe (序列化反序列化类型)的表才能修改字段



删除表：
drop table if exists user_info


加载到普通表
-----------------------

可以将本地文本文件内容批量加载到Hive表中，要求文本文件中的格式和Hive表的定义一致，包括：字段个数、字段顺序、列分隔符都要一致。

这里的user_info表的表定义是以\t作为列分隔符，所以准备好数据后，将文本文件拷贝到hive客户端机器上后，执行加载命令。

load data local inpath '/home/hadoop/userinfodata.txt' overwrite into table user_info;

local关键字表示源数据文件在本地，源文件可以在HDFS上，如果在HDFS上，则去掉local，inpath后面的路径是类似”hdfs://namenode:9000/user/datapath”这样的HDFS上文件的路径。

overwrite关键字表示如果hive表中存在数据，就会覆盖掉原有的数据。如果省略overwrite，则默认是追加数据。

加载完成数据后，在HDFS上就会看到加载的数据文件。

加载到分区表
load data local inpath '/home/hadoop/actionlog.txt' overwrite into table user_action_log 
PARTITION (dt='2017-05-26');
partition 是指定这批数据放入分区2017-05-26中。


加载到分桶表
------先创建普通临时表
create table user_leads_tmp
(
leads_id string,
user_id string,
user_phone string,
user_name string,
create_time string
)
row format delimited fields terminated by ',' 
stored as textfile;
------数据载入临时表
load data local inpath '/home/hadoop/lead.txt' overwrite into table user_leads_tmp;
------导入分桶表
set hive.enforce.bucketing = true;
insert overwrite table user_leads select * from  user_leads_tmp;
set hive.enforce.bucketing = true; 这个配置非常关键，为true就是设置为启用分桶。


导出数据
-----------------
 --导出数据，是将hive表中的数据导出到本地文件中。
insert overwrite local directory '/home/hadoop/user_info.bak2016-08-22 '
select * from user_info;
去掉local关键字，也可以导出到HDFS上。


插入数据
insert select 语句

上一节分桶表数据导入，用到从userleadstmp表向user_leads表中导入数据，用到了insert数据。

insert overwrite table user_leads select * from  user_leads_tmp;
这里是将查询结果导入到表中，overwrite关键字是覆盖目标表中的原来数据。如果缺省，就是追加数据。

如果是插入数据的表是分区表，那么就如下所示：

insert overwrite table user_leads PARTITION (dt='2017-05-26') 
select * from  user_leads_tmp;



一次遍历多次插入
from user_action_log
insert overwrite table log1 select companyid,originalstring  where companyid='100006'
insert overwrite table log2 select companyid,originalstring  where companyid='10002'


每次hive查询，都会将数据集整个遍历一遍。当查询结果会插入多个表中时，可以采用以上语法，
将一次遍历写入多个表，以达到提高效率的目的。


复制表
------------------
复制表是将源表的结构和数据复制并创建为一个新表，复制过程中，可以对数据进行筛选，列可以进行删减。

create table user_leads_bak
row format delimited fields terminated by '\t'
stored as textfile
as
select leads_id,user_id,'2016-08-22' as bakdate
from user_leads
where create_time<'2016-08-22';
上面这个例子是对user_leads表进行复制备份，复制时筛选了2016-08-22以前的数据，减少几个列，并添加了一个bakdate列。

克隆表
---------------------
克隆表时会克隆源表的所有元数据信息，但是不会复制源表的数据。

--克隆表user_leads，创建新表user_leads_like
create table user_leads_like like  user_leads;

备份表
--------------
备份是将表的元数据和数据都导出到HDFS上。

export table user_action_log partition (dt='2016-08-19')
to '/user/hive/action_log.export'
这个例子是将useractionlog表中的一个分区，备份到HDFS上，to后面的路径是HDFS上的路径。

还原表
-------------
将备份在HDFS上的文件，还原到useractionlog_like表中。

import table user_action_log_like from '/user/hive/action_log.export';


HQL语法
===================
函数列
---------------------
select companyid,upper(host),UUID(32) from user_action_log;

可以使用hive自带的函数，也可以是使用用户自定义函数。

上面这个例子upper()就是hive自带函数，UUID()就是用户自定义函数。

算数运算列

select companyid,userid, (companyid + userid) as sumint from useractionlog;

限制返回条数

类似于sql server里的top N，或者mysql里的limit。

select * from user_action_log limit 100;


Case When Then语句

select case when userid < 9 then '未登录' else userid end from user_info;

Where筛选
-----------------
```bash
操作符	说明	操作符	说明
A=B	A等于B就返回true，适用于各种基本类型	A<=>B	都为Null则返回True，其他和=一样
A<>B	不等于	A!=B	不等于
A<B	小于	A<=B	小于等于
A>B	大于	A>=B	大于等于
A Between B And C	筛选A的值处于B和C之间	A Not Between B And C	筛选A的值不处于B和C之间
A Is NULL	筛选A是NULL的	A Is Not NULL	筛选A值不是NULL的
A Like B	%一个或者多个字符_一个字符	A Not Like B	%一个或者多个字符_一个字符
A RLike B	正则匹配
```


Group By 分组
---------------
Hive不支持having语句，有对group by 后的结果进行筛选的需求，可以先将筛选条件放入group by的结果中，然后在包一层，在外边对条件进行筛选。

如果需要进行如下查询：

SELECT col1 FROM t1 GROUP BY col1 HAVING SUM(col2) > 10
可以用下面这种方式实现：
SELECT col1 FROM 
(SELECT col1, SUM(col2) AS col2sum
       FROM t1 GROUP BY col1
) t2
WHERE t2.col2sum > 10

子查询
Hive对子查询的支持有限，只允许在 select from 后面出现。比如：

---只支持如下形式的子查询
select * from (
  select userid,username from user_info i where i.userid='10595'
  ) a;

---不支持如下的子查询
select 
(select username from user_info i where i.userid=d.user_id) 
from user_leads d where d.user_id='10595';



只支持等值连接

Hive支持类似SQL Server的大部分Join操作，但是注意只支持等值连接，并不支持不等连接。原因是Hive语句最终是要转换为MapReduce程序来执行的，但是MapReduce程序很难实现这种不等判断的连接方式。

----等值连接
select lead.* from user_leads lead
left join user_info info 
on lead.user_id=info.userid;
---不等连接（不支持）
select lead.* from user_leads lead
left join user_info info 
on lead.user_id!=info.userid;
连接谓词中不支持or

Left semi join :

I am trying to get the names within table_1 that only appear in table_2.

Then a LEFT SEMI JOIN is the appropriate query to use.




Order By
----------------	
select * from user_leads order by user_id
Hive中的Order By达到的效果和SQL Server中是一样的，会对查询结果进行全局排序，但是Hive语句最终要转换为MapReduce程序放到Hadoop分布式集群上去执行，Order By这样的操作，肯定要在Map后汇集到一个Reduce上执行，如果结果数据量大，那就会造成Reduce执行相当漫长。

所以，Hive中尽量不要用Order By，除非非常确定结果集很小。

但是排序的需求总是有的，Hive中使用下面的几种排序来满足需求。

Sort By
select * from user_leads sort by user_id
这个例子中，Sort By是在每个reduce中进行排序，是一个局部排序，可以保证每个Reduce中是按照userid进行排好序的，但是全局上来说，相同的userid可以被分配到不同的Reduce上，虽然在各个Reduce上是排好序的，但是全局上不一定是排好序的。

Distribute By 和 Sort By
--Distribute By 和Sort By实例
select * from user_leads where user_id!='0' 
Distribute By cast(user_id as int) Sort by cast(user_id as int);
Distribute By 指定map输出结果怎么样划分后分配到各个Reduce上去，比如Distribute By userid，就可以保证userid字段相同的结果被分配到同一个reduce上去执行。然后再指定Sort By userid，则在Reduce上进行按照userid进行排序。

但是这种还是不能做到全局排序，只能保证排序字段值相同的放在一起，并且在reduce上局部是排好序的。

需要注意的是Distribute By 必须写在Sort By前面。

Cluster By
如果Distribute By和Sort By的字段是同一个，可以简写为 Cluster By.

select * from user_leads where user_id!='0' 
Cluster By cast(user_id as int) ;
常见全局排序需求
常见的排序需求有两种：要求最终结果是有序的、按某个字段排序后取出前N条数据。

最终结果是有序的

最终分析结果往往是比较小的，因为客户不太可能最终要的是一个超级大数据集。所以实现方式是先得到一个小结果集，然后在得到最终的小结果集上使用order by 进行排序。

select * from (
select user_id,count(leads_id) cnt from user_leads  
where user_id!='0' 
group by user_id
) a order by a.cnt;
这个语句让程序首先执行group by语句获取到一个小结果集，group by 过程中是不指定排序的，然后再对小结果集进行排序，这样得到的最终结果是全局排序的。

取前N条

select a.leads_id,a.user_name from (
  select leads_id,user_name  from user_leads  
distribute by length(user_name)  sort by length(user_name) desc limit 10
 ) a order by length(a.user_name) desc limit 10;
这个语句是查询username最长的10条记录，实现是先根据username的长度在各个Reduce上进行排序后取各自的前10个，然后再从10*N条的结果集里用order by取前10个。

这个例子一定要结合MapReduce的执行原理和执行过程才能很好的理解，所以这个最能体现：看Hive语句要以MapReduce的角度看。

hive到hdfs
`````
把表格保存到hdfs
insert overwrite directory '/data/sms'
select * from sms;
从hdfs下载到本地
hdfs dfs -get /data/s /homr/s
读取表格从hdfs
load data inpath '/data/s.txt' into table sms

时间戳转int
-----

select unix_timestamp(sd_time) from tb where unix_timestamp(sd_time)<=
