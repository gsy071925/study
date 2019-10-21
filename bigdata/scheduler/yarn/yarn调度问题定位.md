# 当emr不能使用hostname注意事项
对所有工程中的文件，如hbase-site.xml、hdfs-site.xml等文件，包括打包到jar中的文件，也需要filter为具体的ip地址，而不是master之类的hostname;

# yarn错误，更详细日志查看
/usr/local/service/hadoop/bin/yarn logs -applicationId {applicationId} -appOwner {user}  
示例：  
/usr/local/service/hadoop/bin/yarn logs -applicationId application_1566869763239_0621 -appOwner www  
/usr/local/service/hadoop/bin/yarn logs -applicationId application_1569485841152_0597 -appOwner www  
