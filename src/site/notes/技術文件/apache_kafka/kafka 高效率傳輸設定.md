---
{"tags":["kafka"],"dg-publish":true,"dg-home":false,"dg-created_time":"2025-12-22","dg-updated_time":"2025-12-22","permalink":"/技術文件/apache_kafka/kafka 高效率傳輸設定/","dgPassFrontmatter":true,"created":"2025-12-22","updated":"2025-12-22"}
---

訂閱 kafka 消息
```
kafka-console-consumer --bootstrap-server 127.0.0.1:19092 --topic wikimedia.recentchange
```

高效率傳輸時，可考慮以下設定
1. max.in.flight.requests.per.connection
	1. 每個 producer 在 broker 回覆 ack 前，最多送幾筆訊息出去
	2. 若設定 = 1
		1. 訊息只會一筆一筆發，會降低效率，但好處是，若訊息需要嚴格的排序(有新增 sort key)，那很重要
2. linger.ms
	1. 等待一段 linger.ms，在此期間收到的消息都放在自己的暫存區，若 broker 批處理(batch.size)在 linger.ms 到達之前填滿，則立即批處理暫存區內的訊息，否則達到 linger.ms 才進行批處理
3. compression.type
	1. 批處理參數，用於 broker 端壓縮訊息使用的算法(e.g. lz4、zstd、gzip...etc)
4. batch.size
	1. 批處理的單筆 message 大小，若超過，則立即處理該訊息
