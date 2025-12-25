---
{"tags":["ceph","docker"],"linklist":["[[link.tech.ceph]]"],"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-25","dg-updated_time":"2025-12-25","permalink":"/技術文件/ceph/搭建 Ceph cluster/","dgPassFrontmatter":true,"created":"2025-12-25","updated":"2025-12-25"}
---



## 前言
在眾多搭建 ceph 的方法中，最適合 POC 測試的就是用 docker 搭建 ceph 集群，如果是生產環境，可以使用官方推薦的 cephadm 或是 ceph-ansible。

## 目的
用 Docker 建立 ceph cluster，並且支援 aws S3 儲存 API 以及 cephfs

## 環境準備
* os: ubuntu
* 一台有安裝 docker 的電腦
## 節點角色解釋
 1. MON（Monitor 節點）
	- **功能**：
	    - 負責存儲集群的核心數據，例如集群狀態、配置、訪問權限等。
	    - 提供其他節點（OSD、MGR 等）與客戶端的集群狀態信息。
	- **用途**：
	    - 是 Ceph 集群的基礎，沒有 MON 節點，集群無法正常運作。
	- 節點失效情況 :
		- 無法讀寫資料
1. OSD（Object Storage Daemon 節點）
	- **功能**：
	    - 負責實際存儲數據，並管理數據副本的一致性。
	    - 根據 CRUSH 算法決定數據應存放的物理位置。
	    - 提供數據讀取與寫入服務，並定期回報狀態給 MON。
	- **用途**：
	    - Ceph 中的數據存儲骨幹，所有的物件數據都存放在 OSD 中。
	    - 通常一個 OSD 容器或 Process 對應一個物理磁碟。
	- 節點失效情況 :
		- 如果 data pool 內的資料沒有設定 replica，在該 OSD 上的資料會有遺失風險
2. MGR（Manager 節點）
	- **功能**：
	    - 提供集群的內部狀態監控
	- **用途**：
	    - 確保集群監控與健康狀態的即時性。
	- 節點失效情況 :
		- 因為無法正常管理 OSD 節點加入與移除，無法掛載 cephfs，整體集群無法訪問
3. RGW（RADOS Gateway 節點）
	- **功能**：
	    - 為 Ceph 提供對象存儲服務的接口，兼容 S3 和 Swift 協議。
	    - 管理存儲桶（bucket）與對象，用於存儲環境的數據操作。
	- **用途**：
	    - 用於提供對象存儲功能，類似於 AWS S3 的操作方式。
	    - 支援數據訪問的多租戶模式（multi-tenancy）。
	- 節點失效情況 :
		- 應用程式無法使用 S3 API 存取資料，但不會有資料遺失風險
4. MDS (metadata server) 節點
	* **功能**：
	    - 為 Ceph 提供 NFS 接口。
	- **用途**：
	    - 處理 node 掛載請求
	- 節點失效情況 :
		- 應用程式無法掛載 NFS 資料夾，導致應用程式無法存取資料，但資料沒有遺失風險
## Demo
docker-compose.yaml
```
version: '3.9'

services:
  ceph-mon:
    image: ceph/daemon:latest-luminous
    container_name: ceph-mon
    environment:
      - CLUSTER=ceph
      - WEIGHT=1.0
      - MON_IP=192.168.1.55
      - MON_NAME=ceph-mon
      - CEPH_PUBLIC_NETWORK=0.0.0.0/0
    network_mode: "host"
    volumes:
      - /etc/ceph:/etc/ceph
      - /var/lib/ceph:/var/lib/ceph
      - /var/log/ceph:/var/log/ceph
    ports:
      - "6789:6789"
    restart: unless-stopped
    command: mon

  ceph-mgr:
    image: ceph/daemon:latest-luminous
    container_name: ceph-mgr
    privileged: true
    environment:
      - CLUSTER=ceph
    network_mode: "host"
    depends_on:
      - ceph-mon
    pid: "container:ceph-mon"
    volumes:
      - /etc/ceph:/etc/ceph
      - /var/lib/ceph:/var/lib/ceph
    ports:
      - "7000:7000"
    restart: unless-stopped
    command: mgr

  ceph-mds:
    image: ceph/daemon:latest-luminous
    container_name: ceph-mds
    privileged: true
    environment:
      - CLUSTER=ceph
      - CEPHFS_CREATE=1
    network_mode: "host"
    depends_on:
      - ceph-mon
    pid: "container:ceph-mon"
    volumes:
      - /etc/ceph:/etc/ceph
      - /var/lib/ceph:/var/lib/ceph
    restart: unless-stopped
    command: mds

  ceph-osd-1:
    image: ceph/daemon:latest-luminous
    container_name: ceph-osd-1
    privileged: true
    environment:
      - CLUSTER=ceph
      - WEIGHT=1.0
      - MON_NAME=ceph-mon
      - MON_IP=127.0.0.1
      - OSD_TYPE=directory
    network_mode: "host"
    depends_on:
      - ceph-mon
    volumes:
      - /etc/ceph:/etc/ceph
      - /var/lib/ceph:/var/lib/ceph
      - /var/lib/ceph/osd/1:/var/lib/ceph/osd
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    command: osd

  ceph-osd-2:
    image: ceph/daemon:latest-luminous
    container_name: ceph-osd-2
    privileged: true
    environment:
      - CLUSTER=ceph
      - WEIGHT=1.0
      - MON_NAME=ceph-mon
      - MON_IP=127.0.0.1
      - OSD_TYPE=directory
    network_mode: "host"
    depends_on:
      - ceph-mon
    volumes:
      - /etc/ceph:/etc/ceph
      - /var/lib/ceph:/var/lib/ceph
      - /var/lib/ceph/osd/2:/var/lib/ceph/osd
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    command: osd

  ceph-osd-3:
    image: ceph/daemon:latest-luminous
    container_name: ceph-osd-3
    privileged: true
    environment:
      - CLUSTER=ceph
      - WEIGHT=1.0
      - MON_NAME=ceph-mon
      - MON_IP=127.0.0.1
      - OSD_TYPE=directory
    network_mode: "host"
    depends_on:
      - ceph-mon
    volumes:
      - /etc/ceph:/etc/ceph
      - /var/lib/ceph:/var/lib/ceph
      - /var/lib/ceph/osd/3:/var/lib/ceph/osd
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    command: osd
```

建立 osd 節點的 token
```
docker exec ceph-mon ceph auth get client.bootstrap-osd -o /var/lib/ceph/bootstrap-osd/ceph.keyring
```

修改設定檔以支援 etx4 硬碟
```
sudo vi /etc/ceph/ceph.conf
```

在文件末尾添加
```
osd max object name len = 256
osd max object namespace len = 64
```

## 啟動 osd 節點
因為寫入 OSD 需要過半數才能算是寫入成功，如果寫入的 OSD 數量未達「最少寫入共識」就會停止這次寫入，如果持續發生寫入失敗 Ceph 集群就會 crash，而「允許離線數量」表達的是在目前「OSD 數量」下，可以允許最多幾個 OSD 節點下線

| OSD 數量 | 最少寫入共識 | 允許離線數量 |
| ------ | ------ | ------ |
| 1      | 1      | 0      |
| 2      | 2      | 0      |
| 3      | 2      | 1      |
| 4      | 3      | 1      |
| 5      | 3      | 2      |
所以建議最佳 OSD 數量為 3 或 5 個。
## 查看 Ceph 狀態
```
$ sudo ceph status
  cluster:
    id:     f84157cf-dd8e-497b-b8f3-fbc2f597eed2
    health: HEALTH_WARN
            46/69 objects misplaced (66.667%)

  services:
    mon: 1 daemons, quorum ceph-mon
    mgr: jason-loges(active)
    mds: cephfs-1/1/1 up  {0=jason-loges=up:active}
    osd: 3 osds: 3 up, 3 in; 192 remapped pgs

  data:
    pools:   2 pools, 192 pgs
    objects: 23 objects, 139KiB
    usage:   24.2GiB used, 62.5GiB / 86.7GiB avail
    pgs:     46/69 objects misplaced (66.667%)
             192 active+clean+remapped
```
![Pasted image 20241127102210.png](/img/user/images/Pasted%20image%2020241127102210.png)

使用 docker ps 查看 ceph 運行狀態
![Pasted image 20241127102231.png](/img/user/images/Pasted%20image%2020241127102231.png)

ref: https://blog.csdn.net/aslifeih/article/details/135140411

## 環境清理
```
$ rm -r /etc/ceph
$ rm -r /var/lib/ceph/
```
## Next
[[技術文件/ceph/Ceph cluster 資料寫入篇 - ceph S3\|Ceph cluster 資料寫入篇 - ceph S3]]
[[技術文件/ceph/Ceph cluster 資料寫入篇 - RBD\|Ceph cluster 資料寫入篇 - RBD]]
[[技術文件/ceph/Ceph cluster 資料寫入篇 - CephFS\|Ceph cluster 資料寫入篇 - CephFS]]
