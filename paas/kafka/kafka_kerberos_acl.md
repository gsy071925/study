kerberos kafka acl 集成部署文档

# 环境

kafka 2.11_0.10.0.0
zookeeper 3.4.8

# 0.其他参考资料
https://blog.csdn.net/x1172031988/article/details/82852326
http://www.cnblogs.com/dongxiao-yang/p/7131626.html
http://www.orchome.com/500
https://wenku.baidu.com/view/d0390418842458fb770bf78a6529647d26283415.html?from=search

# 1.java security
参考链接：
https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html

# 2.kerberos 安装
参考链接：
https://blog.csdn.net/huanqingdong/article/details/84979110
https://docs.oracle.com/cd/E24847_01/html/819-7061/setup-9.html

## 2.1.server
yum install krb5-libs krb5-server krb5-workstation 

### 创建kdc数据库
kdb5_util create -r niudatastats.nubia.com -s
kdb5_util create -r NUBIA.COM -s

### 启动kdc/kadmin/kprop服务
service krb5kdc stop
service krb5kdc start

service kadmin stop
service kadmin start

service kprop stop
service kprop start

## 2.2.client
yum install krb5-libs krb5-workstation


# 3.环境部署

## 3.1.java中kafka的kerberos参数设置
参考链接：https://blog.csdn.net/TXBSW/article/details/82768955


## 3.2.zookeeper
### 3.2.1导出keytab
kadmin.local -q "addprinc zookeeper/fzdev-228"
kadmin.local -q "ktadd -k /root/zookeeper.keytab zookeeper/fzdev-228@NUBIA.COM"
cp /root/zookeeper.keytab {zk_home}/conf
### 3.2.2配置文件修改
vim {zk_home}/conf/zoo.cfg
------------------------------------------------------------------------------
authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider
requireClientAuthScheme=sasl
jaasLoginRenew=3600000
------------------------------------------------------------------------------

vim {zk_home}/conf/zk_server_jaas.conf
------------------------------------------------------------------------------
Server{
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    useTicketCache=false
    serviceName="zookeeper"
    keyTab="/data/bigdata/apps/zookeeper_cluster/conf_krb/kafka.keytab"
    principal="zookeeper/fzdev-228@NUBIA.COM";
};
------------------------------------------------------------------------------

vim {zk_home}/conf/java.env
------------------------------------------------------------------------------
export JVMFLAGS="-Djava.security.auth.login.config={zk_home}/conf/zk_server_jaas.conf"
------------------------------------------------------------------------------

### 3.2.3启动
kinit 步骤可选
kinit -k -t {zk_home}/conf/zookeeper.keytab zookeeper/fzdev-228
./bin/zkServer.sh start ./conf/zoo.cfg

## 3.3.kafka
### broker
vi {kafka_home}/config/server.properties
------------------------------------------------------------------------------
# acl
authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
super.users=User:root;User:kafka;User:hadoop;

#kerberos
listeners=SASL_PLAINTEXT://10.206.19.181:9092
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=GSSAPI
sasl.enabled.mechanisms=GSSAPI,PLAIN
sasl.kerberos.service.name=kafka
principal.to.local.class=kafka.security.auth.KerberosPrincipalToLocal
------------------------------------------------------------------------------

vi {kafka_home}/config/jaas.conf
------------------------------------------------------------------------------
KafkaServer {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    useTicketCache=false
    serviceName="kafka"
    keyTab="/data/kafka/kafka.keytab"
    principal="kafka/datecenter1@NUBIA.COM";

    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="admin"
    user_admin="admin"
    user_www="nubia123";
};
KafkaClient {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    keyTab="/data/kafka/kafka.keytab"
    storeKey=true
    useTicketCache=false
    serviceName="kafka"
    principal="kafka/datecenter1@NUBIA.COM";
};
Client {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    keyTab="/data/zookeeper/zookeeper.keytab"
    storeKey=true
    useTicketCache=false
    serviceName="zookeeper"
    principal="zookeeper/datecenter1@NUBIA.COM";
};
------------------------------------------------------------------------------

vi {kafka_home}/config/client_jaas.conf  可选
------------------------------------------------------------------------------
KafkaClient{
com.sun.security.auth.module.Krb5LoginModule required
useTicketCache=true;
};
------------------------------------------------------------------------------


vi {kafka_home}/bin/kafka-run-class.sh
------------------------------------------------------------------------------
# JVM performance options
if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
  #KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+DisableExplicitGC -Djava.awt.headless=true"
  KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+DisableExplicitGC -Djava.awt.headless=true -Djava.security.krb5.conf=/etc/krb5.conf -Djava.security.auth.login.config=/data/program/kafka/config/jaas.conf -Dzookeeper.sasl.client.username=zookeeper"
fi
------------------------------------------------------------------------------

vim {kafka_home}/config/producer.properties/consumer.properties
------------------------------------------------------------------------------
#kerberos
security.protocol=SASL_PLAINTEXT
sasl.mechanism=GSSAPI
sasl.kerberos.service.name=kafka
------------------------------------------------------------------------------

# 验证

## 命令行验证
设置环境变量

注意这样配置环境变量只针对当前的进程有效
export KAFKA_OPTS="-Djava.security.auth.login.config=/data/program/kafka/config/clisnt_jaas.conf"
echo $KAFKA_OPTS
-Djava.security.auth.login.config=/usr/local/kafka_client/jaas.conf


-Djava.security.krb5.conf=/etc/krb5.conf -Djava.security.auth.login.config=/data/bigdata/apps/kafka_cluster/config/krb_kafka_server_jaas.conf -Dzookeeper.sasl.client.username=zookeeper


测试Producer
kafka-console-producer --broker-list 192.xxx.xxx.xx:9092,192.xxx.xxx.xx:9092 --topic kerbero --producer.config /usr/local/kafka_client/client.properties
启动producer客户端过程和发消息过程没报错即代表producer客户端配置成功

测试Consumer
kafka-console-consumer --topic kerbero --from-beginning --bootstrap-server 192.xxx.xxx.xx:9092,192.xxx.xxx.xx:9092 --consumer.config client.properties
启动consumer客户端过程和接受消息过程没报错即代表客户端配置成功

### Kafka topic
./bin/krb-kafka-server-start.sh -daemon ../config/server.properties

### kafka acl
查看topic

./bin/kafka-topics.sh --zookeeper fk1:2181,fk2:2181,fk3:2181 --list
查看权限
./bin/kafka-acls.sh --authorizer-properties zookeeper.connect=fk1:2181,fk2:2181,fk3:2181 --list
授予写权限
./bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=fk1:2181,fk2:2181,fk3:2181 --add --allow-principal User:guosy --operation Write --topic test20190304
授予写权限及host
./bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=fk1:2181,fk2:2181,fk3:2181 --add --allow-host 10.206.19.75 --operation Write --topic kerberos1
授予读权限
./bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=fk1:2181,fk2:2181,fk3:2181 --add --allow-principal User:www --operation Read --topic kerberos1 --group www-group
删除权限
./bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=fk1:2181,fk2:2181,fk3:2181 --remove --allow-principal User:guosy --operation Write --topic test20190304


## java kafka
### 生成principal及导出keytab
kadmin.local -q "addprinc guosy/10.206.16.75"
kadmin.local -q "ktadd -k /root/guosy.keytab guosy/10.206.16.75@NUBIA.COM"
### acl授权
### 生产者
1.krb5.conf
2.jaas.conf
------------------------------------------------------------------------------
KafkaClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  storeKey=true
  keyTab="E:\\workspace\\bigdata\\stat-open-stream\\stat-open-stream-server\\src\\main\\resources\\guosy_emr.keytab"
  principal="guosy/10.206.16.75@NUBIA.COM";
};
------------------------------------------------------------------------------
3.kafka properties
------------------------------------------------------------------------------
    private Producer<String, String> createProducer() {
        Properties props = new Properties();
        props.put("bootstrap.servers", KafkaConfig.EMR_KRB_BOOTSTRAP_SERVERS);
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("security.protocol", "SASL_PLAINTEXT");
        props.put("sasl.mechanism", "GSSAPI");
        props.put("sasl.kerberos.service.name", "kafka");
        return new org.apache.kafka.clients.producer.KafkaProducer<String, String>(props);
    }
------------------------------------------------------------------------------
4.krb5.conf及jaas.conf放置工程的resources下面，或者指定绝对路径
------------------------------------------------------------------------------
    public static void main(String[] args) {
        System.setProperty("java.security.krb5.conf", Thread.currentThread().getContextClassLoader().getResource("krb5.conf").getPath());
        System.setProperty("java.security.auth.login.config", Thread.currentThread().getContextClassLoader().getResource("jaas.conf").getPath());
    }
------------------------------------------------------------------------------

### 消费者
操作基本同生成生产者
1.acl中需要增加--group {group.id}

# 其他
## 常用Kerberos指令
https://www.cnblogs.com/xiashiwendao/p/8322235.html

添加principal
kadmin.local -q "addprinc zookeeper/fzdev-228"
kadmin.local -q "ktadd -k /root/zookeeper.keytab zookeeper/fzdev-228@NUBIA.COM"

kadmin.local -q "addprinc kafka/fzdev-228"
kadmin.local -q "ktadd -k /root/kafka.keytab kafka/fzdev-228@NUBIA.COM"

删除
kadmin.local -q "delete_principal kafka/fzdev-228"
修改密码
kadmin.local -q "change_password -pw {pwd} kafka/fzdev-228"

验证kafka.keytab
kinit -kt kafka.keytab kafka/fzdev-228
klist

查看keytab秘钥表
klist -kte /var/kerberos/krb5kdc/keytab/root.keytab

kadmin


## 查看zk日志方法
java -cp slf4j-api-1.6.1.jar:../zookeeper-3.4.8.jar org.apache.zookeeper.server.LogFormatter  /data/bigdata/log/zookeeper/node1/version-2/log.500000001


# 问题整理
## krb5kdc[31403](info): TGS_REQ (4 etypes {18 17 16 23}) 10.206.19.183: PROCESS_TGS: authtime 0,  <unknown client> for <unknown server>, Clock skew too great
使用kerberos的client端的时钟需要与kdc一致或误差较小

## krb5kdc[28606](info): TGS_REQ (4 etypes {18 17 16 23}) 10.206.16.75: LOOKING_UP_SERVER: authtime 0,  zookeeper/10.206.16.75@NUBIA.COM for <unknown server>, Server not found in Kerberos database
