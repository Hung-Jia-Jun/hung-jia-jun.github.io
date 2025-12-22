---
{"tags":["kafka"],"dg-publish":true,"dg-created_time":"2025-12-22","dg-updated_time":"2025-12-22","permalink":"/技術文件/apache_kafka/Kafka Topic Availability/","dgPassFrontmatter":true,"created":"2025-12-22","updated":"2025-12-22"}
---

# Producer ACK 機制說明
1. ACK = 0
	- Case: 發送速度最快，但資料丟失風險最大
	- 只要發送成功就繼續發送下一筆
2. ACK = 1
	- Case: 剛剛好的發送速度，適合日常一般資料使用，有部分資料丟失風險
	- Leader Partition 回傳 ACK 就算成功
3. ACK = all (搭配)
	- Case: 希望資料不要丟失，可以考慮這個設置
	- 需要 Partition 副本組內指定 min.insync.replicas 數量的 replica ACK，訊息才會 ACK
		- min.insync.replicas = 1 -> Leader ACK 就成功
		- min.insync.replicas = 2 -> Leader + replica 兩個 ACK 就成功
![Pasted image 20251207231419.png](/img/user/images/Pasted%20image%2020251207231419.png)
