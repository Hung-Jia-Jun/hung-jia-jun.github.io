---
{"dg-publish":true,"permalink":"/技術文件/Redis/Redis stream 訂閱權限設定/","tags":["Redis","redis-stream","auth","acl"],"created":"2025-12-18T20:28:27.052+08:00","updated":"2025-12-18T20:28:27.052+08:00"}
---

Redis ACL
https://redis.io/docs/latest/operate/oss_and_stack/management/security/acl/

## 建立 Stream

Redis Stream 是「**lazy creation**」，也就是只要你往一個 key 寫入資料，它就會自動建立。
### 建立一個名為 `mystream` 的 Stream

``` bash
XADD mystream * message "Hello World"
```
- `*` → 自動生成 ID（timestamp-based）
- `message "Hello World"` → 欄位與值，可以一次放多個欄位
執行後，`mystream` 就存在了。
![Pasted image 20251218135834.png](/img/user/images/Pasted%20image%2020251218135834.png)

### 列出所有 stream channel
``` bash
SCAN 0 TYPE stream
```
![Pasted image 20251218140101.png](/img/user/images/Pasted%20image%2020251218140101.png)

### 建立只能訂閱 mystream 的使用者

語法如下
```bash
ACL SETUSER stream_user on >StrongPassword +XREAD +XREADGROUP +XRANGE +XINFO ~{{YOUR_STREAM_NAME}}
```

example:
```
ACL SETUSER stream_user on >YOUR_PASSWORD +XREAD +XREADGROUP +XRANGE +XINFO ~mystream
```

> [!NOTE]  
> stream_user → 新使用者名稱
on → 啟用帳號
YOUR_PASSWORD → 密碼
+XREAD +XREADGROUP +XRANGE +XINFO → 允許讀取 Stream 的命令
~my-stream → 只允許存取這個 key，其他 Stream key 都不能存取。

### 使用受 ACL 權限控管的使用者連線到 Redis
``` bash
redis-cli -u redis://stream_user:YOUR_PASSWORD@127.0.0.1:6379
```
![Pasted image 20251218140714.png](/img/user/images/Pasted%20image%2020251218140714.png)

連線後只要操作 `my-stream` 可以成功，其他 Stream key 會出現權限錯誤：

```
XREAD COUNT 1 STREAMS mystream 0   # 成功 XREAD COUNT 1 
STREAMS other-stream 0 # 失敗 -> No Permissions to access a key
```
![Pasted image 20251218140855.png](/img/user/images/Pasted%20image%2020251218140855.png)

最後記得 `acl save`  持久化保存
如何做 ACL 持久化請參閱
[[技術文件/Redis/Redis ACL 持久化\|Redis ACL 持久化]]