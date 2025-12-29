---
{"tags":null,"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-29","dg-updated_time":"2025-12-29","permalink":"/技術文件/ceph/Cephfs ACL mount path/","dgPassFrontmatter":true,"created":"2025-12-29","updated":"2025-12-29"}
---

# 背景
現在已經可以 mount 到指定的資料夾，但還是有機會被 mount 到 "/"
期望可以依照 user keyring 來指定能掛載的 path
官方文件[^1] 已經有實現這部分的 ACL 控制了



# 思路
1. 建立用戶並設定 Auth 規則，只能掛載特定的 Path
2. copy client keyring 到 client 機器
# Demo

## Ceph-cluster master
輸出所有的 ceph filesystem name
```
root@ceph-cluster-01:/mnt/ceph-cluster# ceph fs ls
name: cephfs-data, metadata pool: cephfs.cephfs-data.meta, data pools: [cephfs.cephfs-data.data ]
```

建立 「api_user」，並指定該用戶能掛載目錄位置
```sh
root@ceph-cluster-01:/etc/ceph# ceph fs authorize cephfs-data client.api_user /lab/latest rw
[client.api_user]
	key = YOUR_KEY==
```

建立的 client 會指名只能掛載指定的 path
```bash
root@ceph-cluster-01:/etc/ceph# ceph auth get client.api_user
exported keyring for client.api_user
[client.api_user]
	key = YOUR_KEY==
	caps mds = "allow rw path=/test/api/lab/latest"
	caps mon = "allow r"
	caps osd = "allow rw tag cephfs data=cephfs-data"
```
## client server
將 server 端產製的 auth key 放到 ceph config目錄
```bash
root@lab-api:/etc/ceph# cat << EOF > ceph.client.api_user.keyring
> [client.api_user]
>     key = YOUR_KEY==
> EOF

root@lab-api:/etc/ceph# cat ceph.client.api_user.keyring 
[client.api_user]
    key = YOUR_KEY==
```

嘗試 mount 一個不允許的資料夾位置 `/test/api/lab`
出現 「Operation not permitted」
```bash
root@test:/etc/ceph# ceph-fuse -n client.api_user  -r /test/api/lab /mnt/fuse_cephfs2
ceph-fuse[566845]: starting ceph client
ceph-fuse[5668452024-12-17T17:36:51.358+0800 7f18a4ff9700 -1 client.35409 mds.0 rejected us (non-allowable root '/test/api/lab')
]: ceph mount failed with (1) Operation not permitted
```

如果 mount 到合法的位置，就掛載成功
```bash
root@lab-api:/etc/ceph# ceph-fuse -n client.api_user  -r /test/api/lab/latest /mnt/fuse_cephfs2
ceph-fuse[567096]: starting ceph client
2024-12-17T17:38:16.602+0800 7fd850d0a080 -1 init, newargv = 0x5636070a9900 newargc=9
ceph-fuse[567096]: starting fuse
```

同樣的，也可以使用 linux 系統的 Mount 指令
```
root@lab-api:/etc/ceph# cat /etc/hosts
10.94.1.20 ceph-cluster-01
10.94.1.21 ceph-cluster-02
10.94.1.22 ceph-cluster-03
```

```bash
$ mount -t ceph ceph-cluster-01,ceph-cluster-02,ceph-cluster-03:/test/api/lab/latest /mnt/fuse_cephfs2 -o name=api_user
```
## 清理環境
```bash
$ sudo umount -lf /mnt/fuse_cephfs2/
```
# Reference

[^1]: https://docs.ceph.com/en/reef/cephfs/client-auth/#path-restriction