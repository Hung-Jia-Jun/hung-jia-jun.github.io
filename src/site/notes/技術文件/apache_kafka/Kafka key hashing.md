---
{"dg-publish":true,"permalink":"/技術文件/apache_kafka/Kafka key hashing/","tags":["kafka"],"created":"2025-12-18T21:27:34.277+08:00","updated":"2025-12-18T21:27:34.277+08:00"}
---


Producer 預設的 Partition  邏輯使用 murmur2 進行邏輯分區，需要輸入 key，再根據 key 去切分此筆 message 該去哪個 partition，這帶來一個好處，只要是相同的 key，那他的分區位置就是可預測的，並且 kafka 保證分區內的 message 是有序性的，這點用於序列化資料是很重要的。
> [!CAUTION]
> 若使用 kafka key 有序性分區，當新增 partition 時，會打破原本的分區規則

若因業務需求無法擴張 partition 數量，例如交易資料需要有序性，可以增加 broker 數量
此論點依據為
- Leader partition 會重新分散到更多 broker
	假設：
		- 3 partitions
		- 3 brokers
	- 每個 broker 當 1 個 leader
	現在變成：
		- 3 partitions
		- 6 brokers
- Leader 還是 3 個沒錯，但 follower replicas 會被分散