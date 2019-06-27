# 报错
Error: java.lang.RuntimeException: java.lang.ClassNotFoundException: Class cn.nubia.nps.statistic.job.TodayMsgTotalTrendJob$MyMapper not found
        at org.apache.hadoop.conf.Configuration.getClass(Configuration.java:2195)
        at org.apache.hadoop.mapreduce.task.JobContextImpl.getMapperClass(JobContextImpl.java:186)
        at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:745)
        at org.apache.hadoop.mapred.MapTask.run(MapTask.java:341)
        at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:164)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:422)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1657)
        at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:158)
Caused by: java.lang.ClassNotFoundException: Class cn.nubia.nps.statistic.job.TodayMsgTotalTrendJob$MyMapper not found
        at org.apache.hadoop.conf.Configuration.getClassByName(Configuration.java:2101)
        at org.apache.hadoop.conf.Configuration.getClass(Configuration.java:2193)
        ... 8 more
## 背景
工程编译没有将class打成jar
## 解决方式
将工程编译时打成jar

# 报错
server.NIOServerCnxnFactory: Too many connections from /10.24.37.60 - max is 300
##　解决方式
hbase-site.xml增加如下篇配置
<property>
  <name>hbase.zookeeper.property.maxClientCnxns</name>
  <value>600</value>
</property>