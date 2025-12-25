---
{"tags":["ceph","cephadm","#s3"],"linklist":["[[link.tech.ceph]]"],"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-25","dg-updated_time":"2025-12-25","permalink":"/技術文件/ceph/Ceph cluster 資料寫入篇 - ceph S3/","dgPassFrontmatter":true,"created":"2025-12-25","updated":"2025-12-25"}
---

## 前言
續前篇 [[技術文件/ceph/搭建 Ceph cluster\|搭建 Ceph cluster]] 結果，已經有一個 ceph cluster 可以使用了，現在要做的是將資料存入 ceph S3 內。

這邊會選用 Ceph S3 做範例的原因是因為 Ceph S3 和 AWS S3 都是基於 S3 API 標準進行物件儲存，兩者程式碼間可以無痛轉移。

## 差異
- **AWS S3 (Simple Storage Service)**
    - 完全由 AWS 管理的公有雲服務，使用者無需關注基礎設施的運維。
- **Ceph S3**
    - 開源的分散式儲存系統，可部署在私有雲、公有雲或混合雲中。
    - Ceph 提供了與 S3 API 兼容的物件儲存介面，通過 **RADOS Gateway (RGW)** 實現。

## 相同
**1. 使用 S3 API**
- **相同點**：
    - 都支持 **AWS S3 API 標準**，如 **PUT、GET、DELETE** 請求操作。
    - 客戶端可以使用相同的 S3 工具（如 `boto3`）與 Ceph S3 和 AWS S3 通訊。
    - 支援 RESTful API 進行物件存取和操作。
**2. 存儲模型**
- **相同點**：
    - 都基於 **桶（Bucket）** 和 **物件（Object）** 的存儲結構設計。
    - 支援桶內物件的層級目錄結構（邏輯上的文件夾）。

## Demo
### 搭建 rgw 節點
建立 rgw 節點 token
```
docker exec ceph-mon ceph auth get client.bootstrap-rgw -o /var/lib/ceph/bootstrap-rgw/ceph.keyring
```

啟動 rgw 節點
```
docker run -d --privileged=true --name ceph-rgw --network ceph-network --ip 172.20.0.15 -e CLUSTER=ceph -e RGW_NAME=ceph-rgw -p 7480:7480 -v /var/lib/ceph/:/var/lib/ceph/ -v /etc/ceph:/etc/ceph -v /etc/localtime:/etc/localtime:ro ceph/daemon:latest-luminous rgw
```

### 建立使用者
連線到 ceph-rgw 主機建立 testuser
```
docker exec ceph-rgw radosgw-admin user create --uid="testuser" --display-name="testuser"
```

查看剛剛建立的 testuser
```
docker exec ceph-rgw radosgw-admin   user info --uid testuser
```
![Pasted image 20241126234037.png](/img/user/images/Pasted%20image%2020241126234037.png)

填入剛剛顯示的 access_key 與 secret_key 欄位的值到 python script
```
import os
import boto
import boto.s3.connection

# S3 連線憑證
access_key = 'ACCESS_KEY'
secret_key = 'SECRET_KEY'

# 連接到 S3 儲存服務
conn = boto.connect_s3(
	aws_access_key_id=access_key,
	aws_secret_access_key=secret_key,
	host='172.20.0.15',  # 替換為自建的 ceph RGW 節點 IP
	port=7480,  # 替換為自建的 ceph RGW 節點 Port
	is_secure=False,  # 禁用 HTTPS
	calling_format=boto.s3.connection.OrdinaryCallingFormat(),
)

# 儲存桶內物件的操作
def manage_bucket_operations():
	bucket_name = "my-new-bucket-1"
	local_download_dir = "./downloaded_files"  # 本機下載目錄

	print("建立儲存桶...")
	bucket = conn.create_bucket(bucket_name)

	# 上傳檔案到儲存桶
	file_path = "example.txt"
	print("上傳檔案...")
	key = bucket.new_key("example.txt")
	key.set_contents_from_filename(file_path)
	print("檔案已上傳: example.txt")

	# 顯示儲存桶內所有檔案
	print("顯示儲存桶內所有檔案...")
	for key in bucket.list():
		print(f"檔案名稱: {key.name}")

	# 下載儲存桶內容到本機
	print("下載儲存桶內容到本機...")
	if not os.path.exists(local_download_dir):
		os.makedirs(local_download_dir)
	for key in bucket.list():
		local_file_path = os.path.join(local_download_dir, key.name)
		print(f"下載檔案: {key.name} 到 {local_file_path}")
		key.get_contents_to_filename(local_file_path)
	print(f"儲存桶內容已下載到: {local_download_dir}")

	# 清空儲存桶內所有物件
	print("清空儲存桶內的所有物件...")
	for key in bucket.list():
		print(f"刪除物件: {key.name}")
		bucket.delete_key(key.name)
	print("儲存桶已清空")

	# 刪除儲存桶
	print("刪除儲存桶...")
	conn.delete_bucket(bucket_name)
	print(f"已刪除儲存桶：{bucket_name}")

# 執行範例
if __name__ == "__main__":
	print("執行儲存桶操作...")
	manage_bucket_operations()
```

## Result
![Pasted image 20241127001610.png](/img/user/images/Pasted%20image%2020241127001610.png)


Ref: https://docs.ceph.com/en/reef/radosgw/s3/python/
