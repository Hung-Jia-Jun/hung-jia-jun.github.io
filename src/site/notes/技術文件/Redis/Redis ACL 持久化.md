---
{"dg-publish":true,"permalink":"/技術文件/Redis/Redis ACL 持久化/","tags":["Redis","acl"],"created":"2025-12-19T11:05:15.947+08:00","updated":"2025-12-19T11:05:15.948+08:00"}
---

# 主旨
因為在 [[技術文件/Redis/Redis stream 訂閱權限設定\|Redis stream 訂閱權限設定]] 時，無法 `acl save`
![Pasted image 20251218141616.png](/img/user/images/Pasted%20image%2020251218141616.png)

故需要設定 aclfile 才能儲存 redis acl 設定

# 環境準備
---
## 建立 redis 
---
docker compose
``` fold title=docker-compose.yml
version: "3.9"
services:
  redis:
    image: redis:7.2  # Redis 7+ 才支援 ACL 持久化
    container_name: redis_acl
    ports:
      - "6379:6379"
    volumes:
      - ./redis-data:/data         # 資料持久化
      - ./redis-acl.conf:/usr/local/etc/redis/redis-acl.conf:ro  # 自訂 ACL config
    command: ["redis-server", "/usr/local/etc/redis/redis-acl.conf"]
```

建立 redis-acl.conf 與 acl 設定檔
``` bash
touch redis-acl.conf
mkdir -p ./redis-data
touch ./redis-data/users.acl
```

在 redis-acl.conf 填入以下內容
```
# redis-acl.conf
bind 0.0.0.0
port 6379

# 指定 ACL file
aclfile /data/users.acl
```

資料夾結構
```
.
├── docker-compose.yml
├── redis-acl.conf
└── redis-data
    └── users.acl

2 directories, 3 files
```

啟動 redis
``` bash
docker compose up -d                       
```

連線到 redis
```
redis-cli
```

建立 Redis user
此使用者只能訂閱 mystream (承 [[技術文件/Redis/Redis stream 訂閱權限設定\|Redis stream 訂閱權限設定]])
```
ACL SETUSER stream_user on >YOUR_PASSWORD +XREAD +XREADGROUP +XRANGE +XINFO ~mystream
```

儲存 ACL 設定
```
acl save
```

![Pasted image 20251218211319.png](/img/user/images/Pasted%20image%2020251218211319.png)

重新啟動 redis
```
docker compose down -v && docker compose up -d 
```

![Pasted image 20251218211358.png](/img/user/images/Pasted%20image%2020251218211358.png)

登入剛建立的 ACL 帳戶
```
$ redis-cli
127.0.0.1:6379>  auth stream_user YOUR_PASSWORD
OK
127.0.0.1:6379>
```

stream_user 有持久化保存
![Pasted image 20251218211452.png](/img/user/images/Pasted%20image%2020251218211452.png)


測試未 acl save 就退出的情況
![Pasted image 20251218212007.png](/img/user/images/Pasted%20image%2020251218212007.png)

### optional - 預定義 user
---
編輯 `/redis-data/users.acl`，加入預設使用者：
```
user stream_user2 on >mypassword ~* +@all
```
![Pasted image 20251218213146.png](/img/user/images/Pasted%20image%2020251218213146.png)

啟動 redis 後也能看到剛剛設定的 stream_user2
![Pasted image 20251218213253.png](/img/user/images/Pasted%20image%2020251218213253.png)

## 結語
透過 acl save 可以將 redis-cli session 期間建立的 user 持久化保存，也能透過預先定義的 user.acl 建立 user