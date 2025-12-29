---
{"tags":["#ceph","cephadm"],"linklist":["[[link.tech.ceph]]"],"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-29","dg-updated_time":"2025-12-29","permalink":"/技術文件/ceph/cephadm/ceph 新增 OSD/","dgPassFrontmatter":true,"created":"2025-12-29","updated":"2025-12-29"}
---

確認 orch 服務正常運行
```
root@ceph-cluster-01:/home/# ceph orch status
Backend: cephadm
Available: True
```

嘗試列出所有可用設備，目前會是空的，因為還沒建立
```
ceph orch device ls

```
![Pasted image 20241207225601.png](/img/user/images/Pasted%20image%2020241207225601.png)

查看 block，目前只有一個 sda2 給系統使用
```
lsblk -f 
```
![Pasted image 20251226000701.png](/img/user/images/Pasted%20image%2020251226000701.png)

## 部署要求
- 設備必須沒有分區。
- 設備不得具有任何 LVM 狀態。
- 不得安裝設備。
- 設備不能包含檔案系統。
- 設備不得包含 Ceph BlueStore OSD。
- 設備必須大於 5 GB。

## ESXi 新增磁碟空間
![Pasted image 20241207230318.png](/img/user/images/Pasted%20image%2020241207230318.png)

儲存
![Pasted image 20251226000720.png](/img/user/images/Pasted%20image%2020251226000720.png)

重新開機
![Pasted image 20241207230456.png](/img/user/images/Pasted%20image%2020241207230456.png)

重新開機後，/dev/sdb 磁碟區出現
![Pasted image 20251226000748.png](/img/user/images/Pasted%20image%2020251226000748.png)

## 建立 LVM 分區
![Pasted image 20251226000812.png](/img/user/images/Pasted%20image%2020251226000812.png)
## Ceph 新增 device
> ceph orch daemon add osd {HOST_NAME}:{DEVICE}

只要在 ceph-cluster-01 這台機器輸入 ceph osd daemon 部署指令就好
```
root@ceph-cluster-01:/home# sudo ceph orch daemon add osd ceph-cluster-02:/dev/ceph-vg/ceph-lv-1
Created osd(s) 0 on host 'ceph-cluster-02'

root@ceph-cluster-01:/home# sudo ceph orch daemon add osd ceph-cluster-02:/dev/ceph-vg/ceph-lv-2
Created osd(s) 1 on host 'ceph-cluster-02'

root@ceph-cluster-01:/home# sudo ceph orch daemon add osd ceph-cluster-03:/dev/ceph-vg/ceph-lv-3
Created osd(s) 2 on host 'ceph-cluster-03'
```

![Pasted image 20251226000904.png](/img/user/images/Pasted%20image%2020251226000904.png)

![Pasted image 20241210120104.png](/img/user/images/Pasted%20image%2020241210120104.png)