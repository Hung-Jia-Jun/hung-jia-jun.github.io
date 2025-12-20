---
{"dg-publish":true,"permalink":"/技術文件/apache_kafka/Consumer offset reset 行為/","tags":["kafka","java"],"created":"2025-12-16T21:16:09.446+08:00","updated":"2025-12-17T22:43:36.045+08:00"}
---

# 情境
---
consumer 預期會從 kafka 持續讀取 log，但如果 consumer crash，kafka 會保存 commited offset 7 天

也說明，若 consumer 停機超過 7 天，之前消費的位置將會被重置

# 參數
---
- auto.offset.reset=latest: 會從 kafka topic 最末端讀取 log
- auto.offset.reset=earliest: 從最早的地方開始讀 log
- auto.offset.reset=none: 若沒有 offset 資訊，將會拋出 exception

# 重播 log 給 consumer
---
步驟如下:
1. 關閉該 consumer group 底下所有的 consumer
2. 使用 kafka-consumer-groups 重置你想重置的 offset 位置
3. 重啟 consumer


# Java code

透過 addShutdownHook 偵測 shutdown event
``` java
...
// get a reference  
final Thread mainThread = Thread.currentThread();  
  
Runtime.getRuntime().addShutdownHook(new Thread(){  
    public void run(){  
        log.info("Detected a shutdown event");  
        consumer.wakeup();  
  
        try {  
            mainThread.join();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    }  
});
...
try(openSearchClient; consumer){  
    boolean indexExists = openSearchClient.indices().exists(new GetIndexRequest(indexName), RequestOptions.DEFAULT);  
    if (!indexExists){  
        // we need to create the index on opensearch if it doesn't exist already  
        CreateIndexRequest createIndexRequest = new CreateIndexRequest(indexName);  
        openSearchClient.indices().create(createIndexRequest, RequestOptions.DEFAULT);  
        log.info("The wikimedia index has been created");  
    } else {  
        log.info("The wikimedia index already exists");  
    }  
  
    while (true){  
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(3000));  
  
        int recordCount = records.count();  
        log.info("Received " + recordCount + " record(s)");  
        BulkRequest bulkRequest = new BulkRequest();  
        for (ConsumerRecord<String, String> record : records){  
            try{  
                String id = extractId(record.value());  
                // send the record into opensearch  
                IndexRequest indexRequest = new IndexRequest(indexName)  
                        .source(record.value(), XContentType.JSON)  
                        .id(id);  
                //IndexResponse indexResponse = openSearchClient.index(indexRequest, RequestOptions.DEFAULT);  
                bulkRequest.add(indexRequest);  
            } catch (Exception e){  
  
            }  
        }  
        if (bulkRequest.numberOfActions() > 0){  
            BulkResponse bulkResponse = openSearchClient.bulk(bulkRequest, RequestOptions.DEFAULT);  
            log.info("Inserted " + bulkResponse.getItems().length + " record(s).");  
            try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
  
        }  
  
        consumer.commitAsync();  
        log.info("offset have been committed!");  
    }  
} catch (WakeupException e){  
    log.info("Consumer is starting to shutdown");  
} catch (Exception e){  
    log.error("unexpected exception: ", e);  
} finally {  
    consumer.close(); // close the consumer, this will also commit offest to kafka.  
    openSearchClient.close();  
    log.info("The consumer is now gracefully shut down");  
}
```