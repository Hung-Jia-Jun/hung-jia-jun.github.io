---
{"tags":["ceph","docker"],"linklist":["[[link.tech.ceph]]"],"dg-publish":true,"dg-draft":false,"dg-created_time":{"{ date }":null},"dg-updated_time":{"{ date }":null},"permalink":"/技術文件/ceph/Ceph cluster 資料寫入篇 - CephFS/","dgPassFrontmatter":true,"created":{"{ date }":null},"updated":{"{ date }":null}}
---

## 環境準備

## 在準備掛載的主機
---
安裝 ceph-fuse
```
apt install ceph-fuse
```


### 在 ceph 主機
---
```
ssh ubuntu@ceho-mon <--- cephFS 主機
```

安裝 ceph-fuse
```
apt install ceph-fuse
```

建立使用者
設定client.mds_user 為使用者名稱，後面則是分別設定mon,mds,osd的存取權限
```
ceph auth get-or-create client.mds_user mon 'allow r' mds 'allow *' osd 'allow rw pool=cephfs_metadata,allow rwx pool=cephfs_data'
```

查看使用者及其存取權限
```shell
ceph auth list
```

同步主機憑證
ubuntu@192.168.1.62 為待會要掛載 NFS 資料夾的主機位置
```
sudo scp -i ~/.ssh/id_rsa -r /etc/ceph ubuntu@192.168.1.62:/etc/ceph
```

## 在準備掛載的主機
---

```
ssh ubuntu@192.168.1.62 <--- 欲掛載的主機
```

建立掛載資料夾
```
mkdir -p /mnt/fuse_cephfs
```
掛載 cephfs 到 /mnt/fuse_cephfs
```
# 指定 172.20.0.10 為 ceph-mon 的 IP 位置
sudo ceph-fuse -m 172.20.0.10:6789 /mnt/fuse_cephfs

# 如果 /etc/ceph/ceph.confg 設定檔有設定 mon_ip 就不用帶『-m 172.20.0.10:6789』 參數
sudo ceph-fuse /mnt/fuse_cephfs

# Debug mode
sudo ceph-fuse /mnt/fuse_cephfs -d

# 指定 mount cephfs 的位置
ceph-fuse -r /lab/api /mnt/fuse_cephfs
```
![Pasted image 20241127141833.png](/img/user/images/Pasted%20image%2020241127141833.png)

嘗試讀取同一個檔案內容
![Pasted image 20251225235515.png](/img/user/images/Pasted%20image%2020251225235515.png)

修改檔案後，也同步到另一台機器了
![Pasted image 20251225235413.png](/img/user/images/Pasted%20image%2020251225235413.png)

## 指定 mount cephfs 的位置
```
ceph-fuse -r /lab/api /mnt/fuse_cephfs
```
[[技術文件/ceph/cephfs 指定 mount cephfs 的位置\|cephfs 指定 mount cephfs 的位置]]

# 清理環境
```
$ df -h
```
![Pasted image 20251225235639.png](/img/user/images/Pasted%20image%2020251225235639.png)

清除正在這個 mount 資料夾的user
```
$ fuser -km /mnt/fuse_cephfs/
```
![Pasted image 20251225235620.png](/img/user/images/Pasted%20image%2020251225235620.png)

umount
```
$ sudo umount -lf /mnt/fuse_cephfs/
```

![Pasted image 20251225235707.png](/img/user/images/Pasted%20image%2020251225235707.png)
