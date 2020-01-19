1.组成
------------------
```bash

1.Hbase Master
可以存在多个，但是只有一个在工作。协调Region Server，给Region分配 Region

2.Region server
包括了多个Region，region才是实际放数据的。Region Server提供读写功能

3.Zookeeper
保证Hbase master在运行，注册region region server等作用


2.重点
1.数据分布式保存在HDFS上
2.以 key-value的形式
3.key 是sorted，所以 aaaa距离 zzzz很远

```
