kafka acl 集成部署文档

# 参考资源
http://kafka.apache.org/0100/documentation.html

# 集成操作
## Security
### jdk环境支持
在{jdk_home}/jre/lib/security/java.secutiry增加下面设置
security.provider.11=org.apache.kafka.common.security.plain.PlainSaslServerProvider
或在启动程序增加如下代码段
import org.apache.kafka.common.security.plain.PlainSaslServerProvider;
PlainSaslServerProvider.initialize();

## broker设置
### server.properites
vim {kafka_home}/config/server.properites  建议复制一份
------------------------------------------------------------------------------
#acl
authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
isuper.users=User:root;User:kafka;User:hadoop

#sasl plain
#listeners=PLAINTEXT://10.206.19.228:9092,SASL_PLAINTEXT://10.206.19.228:9082
listeners=SASL_PLAINTEXT://10.206.19.228:9082
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN
------------------------------------------------------------------------------

### acl_kafka_server_jaas.conf
vim /etc/ecm/kafka-conf/acl_kafka_server_jaas.conf
------------------------------------------------------------------------------
KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="admin"
    user_admin="admin"
    user_www="www";
};
------------------------------------------------------------------------------
配置说明：user_{username}="{password}";

### kafka-server-start.sh
vim {kafka_home}/bin/kafka-server-start.sh  建议复制一份
exec $base_dir/kafka-run-class.sh $EXTRA_ARGS -Djava.security.auth.login.config=/data/bigdata/apps/kafka/config/acl_kafka_server_jaas.conf kafka.Kafka "$@"

# 操作验证
## 启动分别重启kafka broker
cd {kafka_home}
./bin/acl-kafka-server-start.sh -daemon ./bin/config/acl_server.properties

## 命令行
### 生产者
./kafka-console-producer.sh --broker-list 10.206.19.228:9092 --topic test20190304
./www-kafka-console-producer.sh --broker-list 10.206.19.228:9092,10.206.19.228:9093,10.206.19.228:9094 --topic test20190304 --producer.config ../config/www-producer.config

### 消费者 
./kafka-console-consumer.sh --zookeeper 10.206.19.228:2181 --topic test20190304 --from-beginning
./www-kafka-console-consumer.sh --zookeeper 10.206.19.228:2181 --topic test20190304 --from-beginning --consumer.config ../config/www-consumer.config

## Java程序
### 生产者
#### 使用acl授权

#### client_jaas.conf
------------------------------------------------------------------------------
KafkaClient {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="www"
  password="www";
};
------------------------------------------------------------------------------

#### properties
------------------------------------------------------------------------------
props.put("security.protocol", "SASL_PLAINTEXT");
props.put("sasl.mechanism", "PLAIN");
------------------------------------------------------------------------------

#### 程序启动系统变量设置
------------------------------------------------------------------------------
public static void main(String[] args) {
    PlainSaslServerProvider.initialize();
    // Security.addProvider(new PlainSaslServerProvider());
    System.setProperty("java.security.auth.login.config", Thread.currentThread().getContextClassLoader().getResource("client_jaas.conf").getPath());
    ....
}
------------------------------------------------------------------------------
也可加到jvm中：-Djava.security.auth.login.config={path}/client_jaas.conf

### 消费者
#### 使用acl对用户及组进行授权
#### 其他同消费者


# kafka集群权限操作指南
## acl命令
### 查看已分配权限列表
./kafka-acls.sh --list --authorizer-properties zookeeper.connect=10.206.19.228:2181,10.206.19.228:2182,10.206.19.228:2183

#### topic权限：Describe、 Write、 Read
./kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=10.206.19.228:2181,10.206.19.228:2182,10.206.19.228:2183 --add --allow-principal User:www --operation Describe --topic test20190304 --group www-group

./kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=10.206.19.228:2181,10.206.19.228:2182,10.206.19.228:2183 --add --allow-principal User:www --operation Write --topic test20190304 --group www-group

./kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=10.206.19.228:2181,10.206.19.228:2182,10.206.19.228:2183 --add --allow-principal User:www --operation Read --topic test20190304 --group www-group

可以组合命令
./kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=10.206.19.228:2181,10.206.19.228:2182,10.206.19.228:2183 --add --allow-principal User:www --operation Describe --operation Write --operation Read --topic test20190304 --group www-group

移除命令 --add =》 --remove

#### kafka-cluster命令  Create Delete Alter ClusterAction All
./kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=10.206.19.228:2181 --add --allow-principal User:www --operation All --cluster kafka-cluster

更多请输入./kafka-acls.sh看帮助说明


