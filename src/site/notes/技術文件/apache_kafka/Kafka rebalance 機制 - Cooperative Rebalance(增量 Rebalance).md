---
{"tags":["kafka"],"dg-publish":true,"dg-created_time":"2025-12-22","dg-updated_time":"2025-12-22","permalink":"/技術文件/apache_kafka/Kafka rebalance 機制 - Cooperative Rebalance(增量 Rebalance)/","dgPassFrontmatter":true,"created":"2025-12-22","updated":"2025-12-22"}
---

# 背景故事
---
kafka 在新版本引入Cooperative Rebalance(增量 Rebalance)機制
在過去版本中，只要 consumer 斷線，就會觸發 rebalance，會停止所有 consumer 的訂閱流
Cooperative Rebalance 就是為了解決這個問題
以下是具體流程
![Pasted image 20251129194411.png](/img/user/images/Pasted%20image%2020251129194411.png)

停止 consumer 1 對 Partition 2 的訂閱
![Pasted image 20251129200033.png](/img/user/images/Pasted%20image%2020251129200033.png)


過一段時間， kafka 會重新 Assign Partition 2 給 Consumer 2(new consumer)
![Pasted image 20251129200130.png](/img/user/images/Pasted%20image%2020251129200130.png)


> [!IMPORTANT]  Cooperative Rebalance 好處
> 其他 Consumer 還是可以持續訂閱，不會中斷

## How to use?
kafka consumer 可以設定 partition.assignment.strategy