---
{"tags":["ceph","cephadm"],"linklist":["[[link.tech.ceph]]"],"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-29","dg-updated_time":"2025-12-29","permalink":"/技術文件/ceph/cephadm/cephadm 搭建 ceph 集群/","dgPassFrontmatter":true,"created":"2025-12-29","updated":"2025-12-29"}
---


## Firewall

| Entity                                          | Port      | direction |
| ----------------------------------------------- | --------- | --------- |
| ceph-cluster-01、ceph-cluster-02、ceph-cluster-03 | 8443      | inbound   |
|                                                 | 3300      | inbound   |
|                                                 | 6789      | inbound   |
|                                                 | 6800~7568 | inbound   |

ref: https://docs.ceph.com/en/reef/cephadm/install/#cephadm-deploying-new-cluster

## 前置條件
1. ceph cluster 內，master 機器都可以用 ssh key 登入每台 VM 的 root 用戶
2. ceph master 主機需要允許 port  `8443`(ceph web ui)
3. 每台機器都要安裝 docker

```
apt install -y cephadm
```

```
cephadm add-repo --release reef
cephadm install
```

透過執行`which`確認`cephadm`現在位於您的 PATH 中
```
which cephadm
```

成功`which cephadm`指令將傳回以下內容：
```
/usr/sbin/cephadm
```

引導 ceph 集群建立
```
cephadm bootstrap --mon-ip 10.0.0.2
```

登入資訊
```
Ceph Dashboard is now available at:

	     URL: https://10.0.0.2:8443/
	    User: admin
	Password: test

You can access the Ceph CLI with:

	sudo /usr/sbin/cephadm shell --fsid a90fc58e-b38f-11ef-b557-732197d2d654 -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring
```

打開 dashboard
https://10.0.0.2:8443/
![Pasted image 20241206183226.png](/img/user/images/Pasted%20image%2020241206183226.png)

進入 cephadm shell
```
$ cephadm shell

Inferring fsid a90fc58e-b38f-11ef-b557-732197d2d654
Inferring config /var/lib/ceph/a90fc58e-b38f-11ef-b557-732197d2d654/mon.ceph-cluster-01/config
Using recent ceph image quay.io/ceph/ceph@sha256:c08064dde4bba4e72a1f55d90ca32df9ef5aafab82efe2e0a0722444a5aaacca
root@ceph-cluster-01:/# 
```

## 加入 host 到 ceph cluster
確認完可以用 ssh key 登入目標主機成為 root 後，就可以新增 ceph node 了
這邊附上卡住的問題記錄 [[技術文件/踩坑日記/踩坑日記 - 遠端用 SSH key 登入 root 權限問題\|踩坑日記 - 遠端用 SSH key 登入 root 權限問題]]

設定 `/etc/hosts`
每台電腦都要
```
# ceph-cluster-01
127.0.1.1 ceph-cluster-01
10.94.1.21 ceph-cluster-02
10.94.1.22 ceph-cluster-03

# ceph-cluster-02
0.94.1.20 ceph-cluster-01
127.0.1.1 ceph-cluster-02
10.94.1.22 ceph-cluster-03

# ceph-cluster-03
10.0.0.2 ceph-cluster-01
10.94.1.21 ceph-cluster-02
127.0.1.1 ceph-cluster-03
```

使用 `ceph orch host add`  指令來新增 host
```
test@ceph-cluster-01:~$ sudo cephadm shell ceph orch host add ceph-cluster-02 10.94.1.21

Inferring fsid a90fc58e-b38f-11ef-b557-732197d2d654
Inferring config /var/lib/ceph/a90fc58e-b38f-11ef-b557-732197d2d654/mon.ceph-cluster-01/config
Using recent ceph image quay.io/ceph/ceph@sha256:c08064dde4bba4e72a1f55d90ca32df9ef5aafab82efe2e0a0722444a5aaacca
Added host 'ceph-cluster-02'
```

ceph-cluster-02 身上的 docker container
![Pasted image 20251226001117.png](/img/user/images/Pasted%20image%2020251226001117.png)



其餘的機器也照同樣的 [[#加入 host 到 ceph cluster]] 流程走

加上 label 到 master ceph node (介面上就會顯示哪台機器是 master 了)
```
test@ceph-cluster-01:~$ sudo ceph orch host label add ceph-cluster-01 master
Added label master to host ceph-cluster-01
```
### 完成
![Pasted image 20241206184354.png](/img/user/images/Pasted%20image%2020241206184354.png)

# Next
[[技術文件/ceph/cephadm/ceph 新增 OSD\|ceph 新增 OSD]]
