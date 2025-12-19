---
{"dg-publish":true,"permalink":"/技術文件/apache_kafka/Kafka rebalance 機制 - Eager Rebalance/","tags":["kafka"],"created":"2025-12-18T21:27:34.277+08:00","updated":"2025-12-18T21:27:34.277+08:00"}
---


# Eager Rebalance(此 rebalance 會造成世界停止)
![Pasted image 20251129194411.png](/img/user/images/Pasted%20image%2020251129194411.png)

停止 consumer 訂閱
![Pasted image 20251129194626.png](/img/user/images/Pasted%20image%2020251129194626.png)

分配完重新開始訂閱
![Pasted image 20251129194713.png](/img/user/images/Pasted%20image%2020251129194713.png)

> [!IMPORTANT]  
> 1. consumer 在 rebalance 後，不一定能再次訂閱到之前的 Partition
> 2. 在世界停止期間，所有 consumer 訂閱會停止


## Rebalance 機制詳解
---
初始狀態
![Pasted image 20251129152948.png](/img/user/images/Pasted%20image%2020251129152948.png)

新的 consumer 加入
![Pasted image 20251129153059.png](/img/user/images/Pasted%20image%2020251129153059.png)

通知 kafka cluster 的 Coordinator broker 進行 Rebalancing
![Pasted image 20251129153922.png](/img/user/images/Pasted%20image%2020251129153922.png)

Coordinator broker 通知 consumer 斷開已連線的 partition
![Pasted image 20251129191802.png](/img/user/images/Pasted%20image%2020251129191802.png)

開始 Rebalancing
![Pasted image 20251129191900.png](/img/user/images/Pasted%20image%2020251129191900.png)

Coordinator broker 隨機選出一個 consumer 做為 Leader Consumer
![Pasted image 20251129192518.png](/img/user/images/Pasted%20image%2020251129192518.png)


Leader Consumer 從 Coordinator broker 取得 consumer group info
![Pasted image 20251129192715.png](/img/user/images/Pasted%20image%2020251129192715.png)

**Leader Consumer** 的責任是：
- 收集所有消費者的訂閱資訊 (subscription)。
- 根據分配策略 (Range, RoundRobin, Sticky 等) 計算 **partition → consumer 的映射**。
- 將分配結果提交給 Coordinator。


**Coordinator** 再把這個分配結果下發給所有消費者，完成 rebalance。
![Pasted image 20251129193028.png](/img/user/images/Pasted%20image%2020251129193028.png)

完成 rebalance
![Pasted image 20251129193100.png](/img/user/images/Pasted%20image%2020251129193100.png)

