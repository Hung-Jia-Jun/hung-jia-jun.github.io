---
{"tags":["ceph","cephadm"],"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-29","dg-updated_time":"2025-12-29","permalink":"/技術文件/ceph/cephadm/ceph 建立 filesystem/","dgPassFrontmatter":true,"created":"2025-12-29","updated":"2025-12-29"}
---

# 背景
mds service 需要有一個 datapool，才能將這個 datapool 掛載成 nfs

> [!IMPORTANT]  
> 以下操作指令只需要在 master 主機完成


為了可以使用 cephfs，所以需要先建立 data pool
```
$ ceph fs volume create cephfs-data
$ ceph fs volume ls
```
![Pasted image 20251226000547.png](/img/user/images/Pasted%20image%2020251226000547.png)

> [!TIP] Optional
> 在 master 主機下指令開啟 datapool 刪除權限
>> ceph config set mon mon_allow_pool_delete true

Pools 介面上長出兩個 pool
1. cephfs.cephfs-data.data
	- 存放資料的 pool
2. cephfs.cephfs-data.meta
	- 給 mds service 使用的 metadata
![Pasted image 20241210123239.png](/img/user/images/Pasted%20image%2020241210123239.png)


部署 mds 服務
```
ceph orch apply mds cephfs-data
```
![Pasted image 20251226000609.png](/img/user/images/Pasted%20image%2020251226000609.png)


MDS 的服務已經分別部署在 ceph-cluster-01 與 ceph-cluster-02
![Pasted image 20241210123533.png](/img/user/images/Pasted%20image%2020241210123533.png)

之後再根據這篇完成掛載即可
[[技術文件/ceph/Ceph cluster 資料寫入篇 - CephFS\|Ceph cluster 資料寫入篇 - CephFS]]