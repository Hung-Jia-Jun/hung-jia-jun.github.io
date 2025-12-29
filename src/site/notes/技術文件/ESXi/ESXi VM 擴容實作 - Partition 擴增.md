---
{"tags":["#linux","#ubuntu","#PV","#LV","VG"],"linklist":["[[link.tech.linux]]"],"related":["[[ESXi VM 擴容實作 - LVM2]]","[[aws vm 擴容實驗]]"],"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-29","dg-updated_time":"2025-12-29","permalink":"/技術文件/ESXi/ESXi VM 擴容實作 - Partition 擴增/","dgPassFrontmatter":true,"created":"2025-12-29","updated":"2025-12-29"}
---



# 背景
在 ESXi 調整硬碟大小
![Pasted image 20241207193016.png](/img/user/images/Pasted%20image%2020241207193016.png)

但這次是直接將 partition 掛載到 `/`
沒有再經過 PV -> VG -> LV 掛載流程
![Pasted image 20251229211818.png](/img/user/images/Pasted%20image%2020251229211818.png)

# 開始
輸入
```
parted
```
![Pasted image 20251229211932.png](/img/user/images/Pasted%20image%2020251229211932.png)
確認新擴增的空間尚未被格式化

> [!CAUTION] 注意 !!!
> 要被 growpart 的空間絕對不能被格式化，不然會出現「NOCHANGE: partition 2 is size 62908416. it cannot be grown」錯誤
>> 如果發現已經被格式化過了，就刪除該分區，讓他變成未被格式化的狀態，如下圖
>
![Pasted image 20241207193928.png](/img/user/images/Pasted%20image%2020241207193928.png)

增加 partition 空間
```
growpart /dev/sda 2
```
![Pasted image 20251229212025.png](/img/user/images/Pasted%20image%2020251229212025.png)


用 `resize2fs` 讓實際掛載在系統根目錄的 `/dev/sda2` 擴容
```
resize2fs /dev/sda2
```
![Pasted image 20251229212105.png](/img/user/images/Pasted%20image%2020251229212105.png)


# 總結
如果掛載到系統根目錄的方式沒有使用 LVM 的方式，即透過 PV(Physical Volume) 組成 VG(Volume group)，然後在 VG(Volume group) 裡面建立 LV(Logic volume) 的方法，用上面的方式就可以對特定的 partition 做擴容了，方法相對簡單，但沒有 LVM2(Logic volume manage, version 2) 的加持，未來調整空間就比較受限