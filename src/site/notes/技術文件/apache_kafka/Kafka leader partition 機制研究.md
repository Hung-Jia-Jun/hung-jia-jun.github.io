---
{"tags":["kafka"],"dg-publish":true,"dg-created_time":"2025-12-22","dg-updated_time":"2025-12-22","permalink":"/技術文件/apache_kafka/Kafka leader partition 機制研究/","dgPassFrontmatter":true,"created":"2025-12-22","updated":"2025-12-22"}
---


# leader partition 概念
每個 Topic 可以有多個 Partition
每個 Partition 都是一個 Partition 副本組
每個 Partition 副本組可以有多個 replication
每個 Partition 副本組會決定誰是 leader partition
![Pasted image 20251207175715.png](/img/user/images/Pasted%20image%2020251207175715.png)


topic = 多個 partition 副本組的集合
每個 Partition 都會有一個 Leader 與零個或多個 followers(副本)
kafka 分片分配規則
1. 在同個 topic 中，每個 partition 副本組的 leader 分片會儘量分散到每個 Broker
2. 一台 broker 可以同時擔任多個 partitions 副本組的 leader
![Pasted image 20251207175340.png](/img/user/images/Pasted%20image%2020251207175340.png)
