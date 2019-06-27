# A master URL must be set in your configuration错误

在运行Spark的测试程序SparkPi时，点击运行，出现了如下错误：
Exception in thread "main" org.apache.spark.SparkException: A master URL must be set in your configuration
at org.apache.spark.SparkContext.<init>(SparkContext.Scala:185)
at SparkPi$.main(SparkPi.scala:12)
at SparkPi.main(SparkPi.scala)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.Java:57)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:606)
at com.intellij.rt.execution.application.AppMain.main(AppMain.java:134)
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
从提示中可以看出找不到程序运行的master，此时需要配置环境变量。
传递给spark的master url可以有如下几种：
local 本地单线程
local[K] 本地多线程（指定K个内核）
local[*] 本地多线程（指定所有可用内核）
spark://HOST:PORT 连接到指定的 Spark standalone cluster master，需要指定端口。
mesos://HOST:PORT 连接到指定的 Mesos 集群，需要指定端口。
yarn-client客户端模式 连接到 YARN 集群。需要配置 HADOOP_CONF_DIR。
yarn-cluster集群模式 连接到 YARN 集群。需要配置 HADOOP_CONF_DIR。

点击edit configuration，在左侧点击该项目。在右侧VM options中输入“-Dspark.master=local”，指示本程序本地单线程运行，再次运行即可。
或
val spark = SparkSession.builder.master("local").appName("NaiveBayesExample").getOrCreate()

============================================================================================================================================================
Please increase heap size using the --driver-memory option or spark.driver.memory in Spark configura
VM ooptions：-Xms256m -Xmx1024m
============================================================================================================================================================


# spark sql 执行读写 hive映射hbase，找不到类org.apache.hadoop.hive.hbase.HBaseSerDe
## 解决
引入相关类库
参考 https://blog.csdn.net/zh350229319/article/details/82802175

# 报错
org.apache.hadoop.hive.hbase.HiveHBaseTableOutputFormat cannot be cast to org.apache.hadoop.hive.ql.io.HiveOutputFormat
## 解决
修复改问题，重新打包
参考 https://github.com/apache/spark/pull/18127


# spark java添加依赖jar

        /*String[] jars = {"D:\\opt\\apps\\hive\\lib\\hbase-annotations-1.1.1.jar",
                "D:\\opt\\apps\\hive\\lib\\hbase-client-1.1.1.jar",
                "D:\\opt\\apps\\hive\\lib\\hbase-common-1.1.1.jar",
                "D:\\opt\\apps\\hive\\lib\\hbase-hadoop2-compat-1.1.1.jar",
                "D:\\opt\\apps\\hive\\lib\\hbase-hadoop-compat-1.1.1.jar",
                "D:\\opt\\apps\\hive\\lib\\hbase-protocol-1.1.1.jar",
                "D:\\opt\\apps\\hive\\lib\\hbase-server-1.1.1.jar",
                "D:\\opt\\apps\\hive\\lib\\hive-hbase-handler-2.3.2.jar",
                "D:\\opt\\apps\\hive\\lib\\htrace-core-3.1.0-incubating.jar"
        };
        SparkConf sparkConf = new SparkConf().setJars(jars);*/
        // .config(sparkConf)
        final SparkSession ss = SparkSession.builder().appName("niudatastats.platform_usrpf_tag").master("local[*]").config("hive.metastore.uris", hive_url).enableHiveSupport()
                .getOrCreate();
        /*ss.sparkContext().addJar("D:\\opt\\apps\\hive\\lib\\hbase-annotations-1.1.1.jar");
        ss.sparkContext().addJar("D:\\opt\\apps\\hive\\lib\\hbase-client-1.1.1.jar");
        ss.sparkContext().addJar("D:\\opt\\apps\\hive\\lib\\hbase-common-1.1.1.jar");
        ss.sparkContext().addJar("D:\\opt\\apps\\hive\\lib\\hbase-hadoop2-compat-1.1.1.jar");
        ss.sparkContext().addJar("D:\\opt\\apps\\hive\\lib\\hbase-hadoop-compat-1.1.1.jar");
        ss.sparkContext().addJar("D:\\opt\\apps\\hive\\lib\\hbase-protocol-1.1.1.jar");
        ss.sparkContext().addJar("D:\\opt\\apps\\hive\\lib\\hbase-server-1.1.1.jar");
        ss.sparkContext().addJar("D:\\opt\\apps\\hive\\lib\\hive-hbase-handler-2.3.2.jar");
        ss.sparkContext().addJar("D:\\opt\\apps\\hive\\lib\\htrace-core-3.1.0-incubating.jar");*/

