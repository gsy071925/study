# 报错
 hdfs path：hive的库的路径下没有文件读写权限  /user/hive/warehouse/cn_nubia_neostore.db
## 解决办法
不确定为符号问题，将  .db   改成 *，即下面：
/user/hive/warehouse/cn_nubia_neostore.db   ->   /user/hive/warehouse/cn_nubia_neostore*