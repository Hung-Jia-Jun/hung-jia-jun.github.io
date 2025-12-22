---
{"tags":["kafka"],"dg-publish":true,"dg-created_time":"2025-12-22","dg-updated_time":"2025-12-22","permalink":"/技術文件/apache_kafka/Kafka consumer group 與 rebalance 機制/","dgPassFrontmatter":true,"created":"2025-12-22","updated":"2025-12-22"}
---


Topic 可以切分多個 Partition
![Pasted image 20251129134453.png](/img/user/images/Pasted%20image%2020251129134453.png)

一個 consumer 可以訂閱多個 Partition
![Pasted image 20251129134625.png](/img/user/images/Pasted%20image%2020251129134625.png)

每個 consumer 都可以訂閱 Topic 內所有的 partition
![Pasted image 20251129135104.png](/img/user/images/Pasted%20image%2020251129135104.png)

多個 Consumer 可以組成一個 Consumer group
![Pasted image 20251129134529.png](/img/user/images/Pasted%20image%2020251129134529.png)

同個 Consumer group 內共享 Topic 內所有的 Partition
一個 consumer 就會訂閱所有的 Partition
![Pasted image 20251129135735.png](/img/user/images/Pasted%20image%2020251129135735.png)

兩個 consumer 分散訂閱 Partition
![Pasted image 20251129135906.png](/img/user/images/Pasted%20image%2020251129135906.png)

三個 consumer 就會一個 consumer 訂閱一個 partition，均分所有的 Partition
![Pasted image 20251129140037.png](/img/user/images/Pasted%20image%2020251129140037.png)

若 consumer 數量 > Partition 數量，多餘的 Consumer 就會 Disable
![Pasted image 20251129140258.png](/img/user/images/Pasted%20image%2020251129140258.png)


## 以 UI 介面為例
Consumer group: `my-java-application`
![Pasted image 20251129140444.png](/img/user/images/Pasted%20image%2020251129140444.png)

訂閱了一個 Topic，並且擁有兩個 Member
![Pasted image 20251129140641.png](/img/user/images/Pasted%20image%2020251129140641.png)

並且從 Assigned partitions 可以得知，總共有三個 Partition
client id 為 `b52ac033-cf4b-49d2-8a13-78ea6b4e1cf1` 的 consumer 分到 1 個 partition
client id 為 `5545bc61-e3fd-4f2f-a0d7-fc7d77d1ddcc` 的 consumer 分到 2 個 partition
![Pasted image 20251129140811.png](/img/user/images/Pasted%20image%2020251129140811.png)

查詢 consumer group 裡面的 member 目前訂閱哪個 Topic，與每個 member 訂閱的 Partition 有哪些
```
$ kafka-consumer-groups --bootstrap-server localhost:19092 --group my-java-application --describe
```
![Pasted image 20251129141307.png](/img/user/images/Pasted%20image%2020251129141307.png)

## 案例實作
Topic: demo_java
Partition: 3
```
$ kafka-topics.sh --bootstrap-server localhost:19092 --topic demo_java --describe                           
```
![Pasted image 20251129141715.png](/img/user/images/Pasted%20image%2020251129141715.png)
### Case 1: 只有一個訂閱者
CONSUMER-ID: consumer-my-java-application-1-c478a360-25f4-4cd8-8a13-158f0b960d71 (訂閱三個 partition)
![Pasted image 20251129141509.png](/img/user/images/Pasted%20image%2020251129141509.png)

## Case 2: 兩個訂閱者
consumer-my-java-application-1-a2c4de74-999b-4898-a0e3-b2f165a37a75 訂閱 id: 0,1 partition
 consumer-my-java-application-1-c478a360-25f4-4cd8-8a13-158f0b960d71 訂閱 id: 2 partition
![Pasted image 20251129141835.png](/img/user/images/Pasted%20image%2020251129141835.png)

## Case 3: 三個訂閱者(一個 Partition 對應一個 consumer)
![Pasted image 20251129142121.png](/img/user/images/Pasted%20image%2020251129142121.png)

## Case 4: 四個訂閱者(訂閱者數量 > Partition 數量)
Consumer 0 未被分配到任一個 Partition
![Pasted image 20251129142635.png](/img/user/images/Pasted%20image%2020251129142635.png)

# Rebalance 機制
每次 Rebalance 都需要 Broker 參與，但不是每台 Broker 都需要參與 Rebalance 的行為，只有 Coordinator Broker 才需要
透過以下指令查詢 Coordinator Broker 
```
$ kafka-consumer-groups.sh --bootstrap-server localhost:19092 --group my-java-application --describe --state 
```
![Pasted image 20251129144503.png](/img/user/images/Pasted%20image%2020251129144503.png)
以這次案例來說，group: my-java-application 的 Coordinator Broker 是 localhost:19092 

觀察每次新建 consumer 都會產出的 Log
```bash
Request joining group due to: group is already rebalancing
Revoke previously assigned partitions demo_java-1

(Re-)joining group

Successfully joined group with generation Generation{generationId=34, memberId='consumer-my-java-application-1-a2c4de74-999b-4898-a0e3-b2f165a37a75', protocol='range'}

Notifying assignor about the new Assignment(partitions=[demo_java-2])
Adding newly assigned partitions: demo_java-2
Setting offset for partition demo_java-2 to the committed offset FetchPosition{offset=959, offsetEpoch=Optional[1], currentLeader=LeaderAndEpoch{leader=Optional[localhost:19092 (id: 0 rack: null)], epoch=absent}}
```

| Log                                                                     | 解釋                                                       |
| ----------------------------------------------------------------------- | -------------------------------------------------------- |
| **Request joining group due to: group is already rebalancing**          | Group 在 rebalance，consumer 需要重新加入 group。                 |
| **Revoke previously assigned partitions demo_java-1**                   | Coordinator 通知 consumer 放棄目前持有的 partition，準備進入下一輪分配。     |
| **(Re-)joining group**                                                  | consumer 正在重新加入 group，等待 coordinator 分配。                 |
| **Successfully joined group with generation...**                        | consumer 正式加入 group，獲得 generationId 與 memberId。          |
| **Notifying assignor about the new Assignment(...)**                    | consumer client 收到 coordinator 的分配結果，更新要訂閱的 partition    |
| **Adding newly assigned partitions: demo_java-2**                       | 新一輪 rebalance 分配到了 demo_java-2。                          |
| **Setting offset for partition demo_java-2 to the committed offset...** | consumer 讀取起點設定為 partition 目前已 commit 的 offset（本例為 959）。 |
> [!TIP]
> Generation 用於追蹤目前 Group 的狀態，避免舊的成員提交 offset，只有新成員能提交 Partition offset

