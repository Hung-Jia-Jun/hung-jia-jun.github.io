---
{"tags":["#ESXi","#linux","#ubuntu","#PV","#LV","VG"],"linklist":["[[link.tech.linux]]"],"related":["[[aws vm 擴容實驗]]","[[ESXi VM 擴容實作 - Partition 擴增]]"],"dg-publish":true,"dg-draft":false,"dg-created_time":"2025-12-29","dg-updated_time":"2025-12-29","permalink":"/技術文件/ESXi/ESXi VM 擴容實作 - LVM2/","dgPassFrontmatter":true,"created":"2025-12-29","updated":"2025-12-29"}
---


# 背景
在 ESXi 建立的 VM 想增加磁碟空間，所以紀錄下整個過程。

目標主機使用 LVM2 的方式掛載系統根目錄，即實際掛載對象為 LV(Logic volume)。
## 系統版本
ubuntu 20.04.5

![Pasted image 20241207173634.png](/img/user/images/Pasted%20image%2020241207173634.png)

# 開始
進入 edit settings
![Pasted image 20251229211619.png](/img/user/images/Pasted%20image%2020251229211619.png)

調整硬碟大小
![Pasted image 20241207173040.png](/img/user/images/Pasted%20image%2020241207173040.png)

![Pasted image 20241207173153.png](/img/user/images/Pasted%20image%2020241207173153.png)

調整完成，硬碟空間變大了
![Pasted image 20251229211639.png](/img/user/images/Pasted%20image%2020251229211639.png)


但目前 linux 主機內空間還是沒改變
![Pasted image 20241207173444.png](/img/user/images/Pasted%20image%2020241207173444.png)

重開機後
/dev/sda 就擴容到 35G 了
![Pasted image 20241207174314.png](/img/user/images/Pasted%20image%2020241207174314.png)

但實際掛載在系統目錄 lv 的大小還是沒改變，目標是「擴增系統位置 "/"」
```
$ lsblk
```
![Pasted image 20241207174728.png](/img/user/images/Pasted%20image%2020241207174728.png)

查看 pv 與 vg 狀態，目前都是 28.25(原始大小)
```
$ pvdisplay
$ vgdisplay
```
![Pasted image 20241207175440.png](/img/user/images/Pasted%20image%2020241207175440.png)

LV 的空間 28G
```
$ lvdisplay
```
- LV 是實際上被掛載到系統根目錄 "/" 的 volume
![Pasted image 20241207175659.png](/img/user/images/Pasted%20image%2020241207175659.png)

---
## 擴充原有 partition，使用剩餘空間
使用 parted 工具進行 resize
{ #dc4611}

```
$ parted
```

輸入「`p`」顯示所有 partition
系統告知不是所有空間都能被使用
輸入「Fix」修復 GPT(GUID Partition Table) 分區表
![Pasted image 20241207180056.png](/img/user/images/Pasted%20image%2020241207180056.png)

執行 partition resize 操作                                                         
```
(parted) resizepart
```

將 /dev/sda 上的 /dev/sda3(partition 3) 從 32G 擴增到 35G，此時 number 3 partition 已經使用了所有空間
![Pasted image 20241207180824.png](/img/user/images/Pasted%20image%2020241207180824.png)

輸入 「`q`」離開
![Pasted image 20241207181023.png](/img/user/images/Pasted%20image%2020241207181023.png)

## 擴充 PV 空間
原本上面的 number 3 partition 已經是現存的 PV，既然 partition 已經變大，接著就可以直接進行 resize 的操作
尚未進行 resize 前 PV 大小 28.25G
```
$ pvdisplay
```
![Pasted image 20241207181459.png](/img/user/images/Pasted%20image%2020241207181459.png)

執行 pvresize  
```
$ pvresize /dev/sda3
```

變成 30.84G
![Pasted image 20241207181722.png](/img/user/images/Pasted%20image%2020241207181722.png)

查看 vg size
vg size 也變大了
```
$ vgdisplay
```
![Pasted image 20241207181909.png](/img/user/images/Pasted%20image%2020241207181909.png)


# 擴充 LV 空間
接著擴充 LV 空間

```
# 原本的 LV 空間狀況  
$ lvdisplay
```
![Pasted image 20241207182040.png](/img/user/images/Pasted%20image%2020241207182040.png)

 # 將 LV 空間擴充成 35GB  
```
$ lvextend -L 35G /dev/ubuntu-vg/ubuntu-lv
```
![Pasted image 20241207182818.png](/img/user/images/Pasted%20image%2020241207182818.png)

出現錯誤，看起來是目前輸入的值超過可擴展的 pv 空間

進一步排查發現，扣掉 1.8G 的 sda2 分區，還有2.3GB 的 disk 空間未分配給 /dev/sda3
![Pasted image 20241207183136.png](/img/user/images/Pasted%20image%2020241207183136.png)

輸入
```
$ parted
```
回到 [[技術文件/ESXi/ESXi VM 擴容實作 - LVM2#擴充原有 partition，使用剩餘空間\|ESXi VM 擴容實作 - LVM2#擴充原有 partition，使用剩餘空間]]
但這次請注意這個 Disk 值，這是可用的最大空間
![Pasted image 20241207184423.png](/img/user/images/Pasted%20image%2020241207184423.png)

partition 3 確實沒有佔滿所有的 37.6GB 的空間
![Pasted image 20241207184930.png](/img/user/images/Pasted%20image%2020241207184930.png)

將 end 設為 37.6GB
![Pasted image 20241207184647.png](/img/user/images/Pasted%20image%2020241207184647.png)

再執行一次 pvresize
```
pvresize /dev/sda3
```

已經從原本的 30.84GB 增加到 33.25GB 了
![Pasted image 20241207185342.png](/img/user/images/Pasted%20image%2020241207185342.png)

執行 lvextend 增加 lv 空間
```
lvextend -L 33G /dev/ubuntu-vg/ubuntu-lv
```
![Pasted image 20241207185752.png](/img/user/images/Pasted%20image%2020241207185752.png)

最後再 resize2fs 增加系統根目錄 `/` 的可用空間
```
resize2fs /dev/ubuntu-vg/ubuntu-lv
```
![Pasted image 20241207190011.png](/img/user/images/Pasted%20image%2020241207190011.png)
# 知識點
1. 磁碟掛載命名順序與分區命名規則如下:
	1. disk:
		1. 第一顆 disk -> /dev/sda
			1. 第一個分區 -> /dev/sda1
				1. partition id: 1
			2. 第二個分區 -> /dev/sda2
				1. partition id: 2
			3. ... 以此類推
		2. 第二顆 disk -> /dev/sdb
		3. 第三顆 disk -> /dev/sdc
		4. ... 以此類推
2. 理解 Linux PV、LV、VG 之間的關係
	1. PV(Physical volume) = 磁碟分割區 (/dev/sda1、/dev/sda2) 
		1. 可以從 `pvdisplay` 看到，PV Name = 系統磁碟分區![Pasted image 20241207190420.png](/img/user/images/Pasted%20image%2020241207190420.png)
	2. VG(Volume group)
		1. 多個 PV 可以建成一個 VG，也可以將 PV 加入現有 VG
	3. LV(Logical volume)
		1. 用 `lsblk -f` 可以看出，sda3 下面有 一個 vg，vg下面有一個 lv![Pasted image 20241207190231.png](/img/user/images/Pasted%20image%2020241207190231.png)
		2. VG 下面會有多個 LV
		3. LV 就是實際掛載給系統根目錄使用的，屬於虛擬邏輯分區
# 指令
```
lsblk -f
```

```
$ pvdisplay
```

```
$ lvdisplay
```

```
$ vgdisplay
```


## parted
```
$ parted
```
## fdisk
輸入 fdisk 指令後，會進入磁碟管理工具裡面
```
$ fdisk /dev/sda
```
指令:
- p 列出該 disk 所有的分區
- m 輸出 menu 列表
- n 建立新的磁碟分區


# 參考資料
https://godleon.github.io/blog/Linux/Linux-extend-lvm-from-unused-space/
https://sc8log.blogspot.com/2017/03/linux-lvm-lvm.html