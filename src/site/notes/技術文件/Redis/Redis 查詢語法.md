---
{"tags":["Redis"],"dg-publish":true,"dg-created_time":"2026-03-24","dg-updated_time":"2026-03-24","dg-draft":false,"permalink":"/技術文件/Redis/Redis 查詢語法/","dgPassFrontmatter":true,"created":"2026-03-24","updated":"2026-03-24"}
---

## Redis 操作紀錄

### 1. 建立連線 (Login)
使用帳戶憑證登入遠端 Redis 伺服器：
```
redis-cli -h HOST_ADDRESS -p 6380 --user ACCOUNT_NAME -a PASSWORD_STRING
```

### 2. 檢查 Hash 欄位存活時間 (HTTL)
查詢指定 Hash 表內某個欄位的剩餘生存時間：
```
redis> HTTL RESOURCE_KEY:TENANT_ID FIELDS 1 FIELD_ID_A
1) (integer) 282
```

### 3. 掃描 Hash 內容 (HSCAN)
列出該 Hash 表下的所有欄位與資料內容：
```
redis> hscan RESOURCE_KEY:TENANT_ID 0 MATCH *
1) "0"
2) 1) "FIELD_ID_A"
   3) "{\"data_field\": ...}"
```

### 4. 正則匹配查詢 (Pattern Match)
使用前綴過濾特定的欄位 ID
```
redis> hscan RESOURCE_KEY:TENANT_ID 0 MATCH FIELD_PREFIX*
1) "0"
2) 1) "FIELD_ID_A"
   3) "{\"data_field\": ...}"
```

### 5. 移除鍵值 (HDEL)
從 Hash 表中刪除指定欄位，並驗證結果：
```
redis> hdel RESOURCE_KEY:TENANT_ID "FIELD_ID_A"
(integer) 1

# 再次掃描確認，回傳空陣列表示已成功移除
redis> hscan RESOURCE_KEY:TENANT_ID 0 MATCH FIELD_PREFIX*
1) "0"
2) (empty array)
```

### 6. 全域掃描與過濾 (SCAN)
掃描符合規則的獨立 Key 列表：
```
redis> SCAN 0 MATCH RESOURCE_PREFIX:TENANT_ID* COUNT 10
1) "1664"
2) 1) "RESOURCE_PREFIX:TENANT_ID:RULE_UUID_1"
   3) "RESOURCE_PREFIX:TENANT_ID:RULE_UUID_2"
   ...
```

### 7. 查詢獨立 Key 的 TTL
```
redis> ttl "RESOURCE_PREFIX:TENANT_ID:RULE_UUID_1"
(integer) 287
```