#### 方法一（优先）
1. 在kafka ingestion的配置文件中添加consumerProperties：
"consumerProperties": {
"bootstrap.servers":"localhost:9092",
"security.protocol":"SASL_PLAINTEXT",
"sasl.kerberos.service.name":"kafka"
    }
    

2. 准备jaas.conf并添加到配置文件：

jaas.conf的内容：
```
KafkaClient {
   com.sun.security.auth.module.Krb5LoginModule required
   useKeyTab=true
   keyTab="/usr/keytab/xiet.keytab"
   principal="xiet@BETA.COM";
};
```

quickstart/tutorial/conf/druid/_common/common.runtime.properties
```
druid.indexer.runner.javaOpts=-server -Xmx2g -Duser.timezone=UTC -Dfile.encoding=UTF-8 -Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager  -Djava.security.auth.login.config=/etc/kafka/kafka_client_kerberos_jaas.conf -Djava.security.krb5.conf=/etc/krb.conf -Djavax.security.auth.useSubjectCredsOnly=false -Djava.security.krb5.kdc=server.myhostname.com -Djava.security.krb5.realm=MYREALM.COM
```

/Users/mac/druid/druid/quickstart/tutorial/conf/druid/overlord/jvm.config
/Users/mac/druid/druid/quickstart/tutorial/conf/druid/middlemanager/jvm.config

```
-Djava.security.auth.login.config=/etc/kafka/kafka_client_kerberos_jaas.conf
```
 
参考：https://groups.google.com/forum/#!topic/druid-user/W2SiPnNsy0U
https://blog.csdn.net/weixin_35852328/article/details/83617829


#### 方法二
在kafka brokers新建一个PLAINTEXT listener
,Druid 通过这个PLAINTEXT listener，不需要Kerberos验证

https://community.hortonworks.com/questions/224740/how-can-i-set-druid-to-ingest-streaming-data-from.html