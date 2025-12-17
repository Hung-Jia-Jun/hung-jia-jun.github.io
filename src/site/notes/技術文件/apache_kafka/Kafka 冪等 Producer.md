---
{"tags":["kafka"],"dg-publish":true,"permalink":"/技術文件/apache_kafka/Kafka 冪等 Producer/","dgPassFrontmatter":true}
---


# 冪等定義
冪等 = 同一個操作，執行一次或執行多次，結果都一樣。
不論你做 1 次、10 次、100 次，
系統的最終狀態必須一模一樣。

kafka 重複 message 問題
若 ack 消息回傳期間網路中斷
Producer 就不會收到 ACK 通知，就會觸發 Retry 機制
造成訊息重複發送的情況
![Pasted image 20251208002229.png](/img/user/images/Pasted%20image%2020251208002229.png)