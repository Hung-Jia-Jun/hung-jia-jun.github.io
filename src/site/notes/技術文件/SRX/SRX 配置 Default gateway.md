---
{"tags":null,"dg-publish":true,"dg-draft":false,"dg-created_time":"2026-01-02","dg-updated_time":"2026-01-02","permalink":"/技術文件/SRX/SRX 配置 Default gateway/","dgPassFrontmatter":true,"created":"2026-01-02","updated":"2026-01-02"}
---


目前環境前面有一個 Fortigate 

所以先檢查 routing-table 的狀態
```
get router info routing-table all
...
Routing table for VRF=0
S*      0.0.0.0/0 [10/0] via 10.6x.1.1, wanlink, [1/0]
...
C       10.7x.1x.0/24 is directly connected, my_esxi
...
FGT60F # 
```


檢查 SRX 上的自動收到 arp 封包產生的 arp table
```
root> show arp no-resolve
MAC Address       Address         Interface     Flags
0x:x5:x0:ax:ex:xx 10.7x.1x.254    fe-0/0/7.0           none

root>
```

要配置在 SRX 上的 default gateway 就會是 10.7x.1x.254，靜態 IP 就要在 10.7x.1x.25/24 裡面挑一個 IP

## 下一步
---
配置一個 route 規則
詳見 [[技術文件/SRX/SRX 上網設定\|SRX 上網設定]]

