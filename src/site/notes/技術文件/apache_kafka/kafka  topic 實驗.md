---
{"dg-publish":true,"permalink":"/技術文件/apache_kafka/kafka  topic 實驗/","tags":["kafka"],"created":"2025-12-18T21:27:34.277+08:00","updated":"2025-12-18T21:27:34.278+08:00"}
---

#kafka

## 環境設定
docker-compose.yaml
```
version: '2.1'
services:
  zoo1:
    image: confluentinc/cp-zookeeper:7.3.2
    hostname: zoo1
    container_name: zoo1
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_SERVERS: zoo1:2888:3888

  kafka1:
    image: confluentinc/cp-kafka:7.3.2
    hostname: kafka1
    container_name: kafka1
    ports:
      - "9092:9092"
      - "29092:29092"
      - "9999:9999"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka1:19092,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092,DOCKER://host.docker.internal:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT,DOCKER:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_JMX_PORT: 9999
      KAFKA_JMX_HOSTNAME: ${DOCKER_HOST_IP:-127.0.0.1}
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.authorizer.AclAuthorizer
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
    depends_on:
      - zoo1
```


```
docker compose up  -d
```


```
docker exec -it kafka1 bash
```

刪除 JMX env
```
unset JMX_PORT && unset KAFKA_JMX_OPTS
```

建立一個 topic
```
kafka-topics --create --topic quickstart-events --bootstrap-server localhost:9092
```

  

檢查 topic 詳情

```
kafka-topics --describe --topic quickstart-events --bootstrap-server localhost:9092
```

  

建一個 producer

```
kafka-console-producer --topic quickstart-events --bootstrap-server localhost:9092
```


查看目前 topic 有多少 message, 顯示的是 offset 的值
```
$ kafka-run-class kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic quickstart-events
quickstart-events:0:16

$ kafka-run-class kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic filebeat
filebeat:0:0
```
  


建立一個 訂閱者，訂閱 quickstart-events
```
kafka-console-consumer --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```

測試 kafka consumer group  消費機制
```
kafka-console-producer --topic quickstart-events --bootstrap-server localhost:9092 --group 1
```

同一個 group 只會有一個 consumer 會消費到一個 topic 的訊息

---

顯示目前 kafka 有多少 topic
```
[appuser@kafka1 ~]$ kafka-topics --list --bootstrap-server localhost:9092
__consumer_offsets
filebeat
metricbeat
my_group2_v2
quickstart-events
```
