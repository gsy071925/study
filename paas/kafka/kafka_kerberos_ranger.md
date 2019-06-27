# 资料
http://blog.sina.com.cn/s/blog_167a8c6480102xfu6.html

# 安装集成
## kafka集成ranger
https://cwiki.apache.org/confluence/display/RANGER/Apache+Ranger+0.5+-+User+Guide#ApacheRanger0.5-UserGuide-KAFKA
https://cwiki.apache.org/confluence/display/RANGER/Ranger+installation+in+Kerberized++Environment#RangerinstallationinKerberizedEnvironment-Installing/EnablingRangerKAFKAplugin


## ranger 上面kafka配置
Service Detail:
 Service Name:kafka_228
 Active Status:Enabled
Config Properties:
 Username:kafka
 Password:nubia123
 Zookeeper Connect String:10.206.19.228:2181,10.206.19.228:2182,10.206.19.228:2183
 Add NewConfigurations:
   tag.download.auth.users:kafka
   policy.download.auth.users:kafka

### Policies
Policy Name:guuosy
Topic*:*

   
# 准备
## 删除原acl列表
./kafka-acls.sh --authorizer-properties zookeeper.connect=fzdev-228:2181,fzdev-228:2182,fzdev-228:2183 --list

./kafka-acls.sh --authorizer-properties zookeeper.connect=fk1:2181,fk2:2181,fk3:2181 --list

./kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=fzdev-228:2181,fzdev-228:2182,fzdev-228:2183 --remove --allow-principal User:www --operation Describe --operation Read --operation Write --topic test20190304

./kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=fzdev-228:2181,fzdev-228:2182,fzdev-228:2183 --remove --allow-principal User:www --operation Read --group www-group

./kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=fk1:2181,fk2:2181,fk3:2181 --remove --allow-principal User:www --operation Read --operation Write --topic kerberos1

./kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=fk1:2181,fk2:2181,fk3:2181 --remove --allow-principal User:www --operation Read --group www-group

## zookeeper
### 导出keytab
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

## 修改kafka配置

### 导出keytab
kadmin.local -q "addprinc kafka/fzdev-228"
kadmin.local -q "ktadd -k /root/kafka.keytab kafka/fzdev-228@NUBIA.COM"
cp /root/zookeeper.keytab {zk_home}/conf

vim ./config/server.properties
-------------------------------------------------------------------------------------------
#kerberos
listeners=SASL_PLAINTEXT://10.206.19.228:9072
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.enabled.mechanisms=GSSAPI,PLAIN
sasl.mechanism.inter.broker.protocol=GSSAPI
sasl.kerberos.service.name=kafka
principal.to.local.class=kafka.security.auth.KerberosPrincipalToLocal

#ranger
authorizer.class.name=org.apache.ranger.authorization.kafka.authorizer.RangerKafkaAuthorizer

auto.create.topics.enable = false
-------------------------------------------------------------------------------------------

vim ./bin/kafka-run-class.sh
-------------------------------------------------------------------------------------------
# JVM performance options
if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
  KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+DisableExplicitGC -Djava.awt.headless=true -Djava.security.krb5.conf=/etc/krb5.conf -Djava.security.auth.login.config=/data/program/kafka/config/krb_kafka_server_jaas.conf -Dzookeeper.sasl.client.username=zookeeper"
  # -Dsun.security.krb5.debug=true
fi
-------------------------------------------------------------------------------------------

vi {kafka_home}/config/krb_kafka_server_jaas.conf
------------------------------------------------------------------------------
KafkaServer {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    useTicketCache=false
    serviceName="kafka"
    keyTab="/data/bigdata/apps/kafka_cluster/config/kafka.keytab"
    principal="kafka/fzdev-228@NUBIA.COM";
};
KafkaClient {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    useTicketCache=true
    renewTicket=true
    keyTab="/data/bigdata/apps/kafka_cluster/config/client.keytab"
    principal="client/fzdev-228@NUBIA.COM";
};
Client {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    useTicketCache=false
    serviceName="zookeeper"
    keyTab="/data/bigdata/apps/kafka_cluster/config/kafka.keytab"
    principal="zookeeper/fzdev-228@NUBIA.COM";
};
------------------------------------------------------------------------------

## ranger-{version}-kafka-plugin安装
假设{ranger_path}=/opt/apps/ranger
{ranger_path}/ranger-{version}-kafka-plugin
mkdir -p /opt/apps/ranger/kafka
ln -s /data/program/kafka/config /opt/apps/ranger/kafka/config
ln -s /data/program/kafka/libs /opt/apps/ranger/kafka/libs



## 修改{ranger_path}/ranger-{version}-kafka-plugin/install.properties
-------------------------------------------------------------------------------------------
COMPONENT_INSTALL_DIR_NAME=/opt/apps/ranger/kafka
POLICY_MGR_URL=http://10.206.19.183:6080
SQL_CONNECTOR_JAR=/usr/share/java/mysql-connector-java.jar
REPOSITORY_NAME=kafka_dev

XAAUDIT.HDFS.IS_ENABLED=true
XAAUDIT.HDFS.DESTINATION_DIRECTORY=hdfs://emr-header-2:8020/ranger/audit/%app-type%/%time:yyyyMMdd%
XAAUDIT.HDFS.LOCAL_BUFFER_DIRECTORY=/data/log/ranger/kafka/audit
XAAUDIT.HDFS.LOCAL_ARCHIVE_DIRECTORY=/data/log/ranger/kafka/audit/archive
-------------------------------------------------------------------------------------------

## bin目录下面增加ranger-kafka-security.xml、ranger-kafka-audit.xml的软链
ln -a /data/program/kafka/config/ranger-kafka-security.xml /data/program/kafka/bin/ranger-kafka-security.xml
ln -a /data/program/kafka/config/ranger-kafka-audit.xml /data/program/kafka/bin/ranger-kafka-audit.xml 

## 修改ranger-kafka-audit.xml，正确设hdfs（active）的环境
-------------------------------------------------------------------------------------------
    <property>
        <name>xasecure.audit.hdfs.config.destination.directory</name>
        <value>hdfs://emr-header-2:8020/ranger/audit/%app-type%/%time:yyyyMMdd%</value>
    </property>
    <property>
        <name>xasecure.audit.destination.hdfs.dir</name>
        <value>hdfs://emr-header-2:8020/ranger/audit</value>
    </property>
-------------------------------------------------------------------------------------------

## 审计写入hdfs中，依赖数据统计emr环境，{ranger_path}/ranger-{version}-kafka-plugin/libs/ranger-kafka-plugin-impl目录下引入如下包
htrace-core-3.1.0-incubating.jar
commons-cli-1.2.jar
protobuf-java-2.5.0.jar
上面3个文件同mysql，放在/usr/share/java/目录下，然后建一个软链
ln -s /usr/share/java/htrace-core-3.1.0-incubating.jar /opt/apps/ranger/ranger-0.7.1-kafka-plugin/lib/ranger-kafka-plugin-impl/htrace-core-3.1.0-incubating.jar
ln -s /usr/share/java/commons-cli-1.2.jar /opt/apps/ranger/ranger-0.7.1-kafka-plugin/lib/ranger-kafka-plugin-impl/commons-cli-1.2.jar
ln -s /usr/share/java/protobuf-java-2.5.0.jar /opt/apps/ranger/ranger-0.7.1-kafka-plugin/lib/ranger-kafka-plugin-impl/protobuf-java-2.5.0.jar

## 给ranger的本地审计日志目录授予权限
mkdir -p /data/log/ranger/kafka/audit
chown kafka:hadoop /data/log/ranger/kafka/audit -R

## 给ranger的hdfs的审计日志目录授予权限
hadoop fs -mkdir -p /ranger/audit/kafka
hadoop fs -chown -R kafka:hadoop /ranger/audit/kafka
以及在ranger admin中，给kafka授权hdfs读写权限
add policy
PolicyName:kafka
ResourcsPath:/ranger/audit/kafka/*
SelectUser:Permissons
kafka:Read Write Execute


## 重启kafka
cd ${kafka_home}/bin

ln -s /data/program/kafka/config/ranger-kafka-security.xml /data/program/kafka/bin/ranger-kafka-security.xml
ln -s /data/program/kafka/config/ranger-kafka-audit.xml /data/program/kafka/bin/ranger-kafka-audit.xml
ln -s /data/program/kafka/config/ranger-security.xml /data/program/kafka/bin/ranger-security.xml

./kafka-server-stop.sh ../config/server.properties
./kafka-server-start.sh -daemon ../config/server.properties
tail -100f ../logs/kafkaServer.out


useradd kafka -d /home/kafka
gpasswd -a kafka hadoop

# kafka&ranger使用
## 如果kafka开启无认证和认证兼容模式，kafka需要开2个监听端口
listeners=SASL_PLAINTEXT://10.206.19.228:9093,PLAINTEXT://10.206.19.228:9092

## 由于接入到ranger后，2个端口都被会ranger拦截
具有所以权限的用户，指定PolocyConditions：将broker的节点，及emr的节点加入授予ANONYMOUS用户所以权限
然后基于kerberos，对业务进行限制


# 问题整理
## kafka启动集成ranger plugin报如下错误
[2019-03-26 17:44:32,992] DEBUG RangerPluginClassLoader.findResource(/ranger-kafka-security.xml): calling componentClassLoader.getResources() (org.apache.ranger.plugin.classloader.RangerPluginClassLoader)
[2019-03-26 17:44:32,992] DEBUG <== RangerPluginClassLoader.findResource(/ranger-kafka-security.xml): null (org.apache.ranger.plugin.classloader.RangerPluginClassLoader)
[2019-03-26 17:44:32,992] ERROR addResourceIfReadable(ranger-kafka-security.xml): couldn't find resource file location (org.apache.ranger.authorization.hadoop.config.RangerConfiguration)
[2019-03-26 17:44:32,992] DEBUG <== addResourceIfReadable(ranger-kafka-security.xml), result=false (org.apache.ranger.authorization.hadoop.config.