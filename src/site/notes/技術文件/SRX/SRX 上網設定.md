---
{"tags":null,"dg-publish":true,"dg-draft":false,"dg-created_time":"2026-01-01","dg-updated_time":"2026-01-01","permalink":"/技術文件/SRX/SRX 上網設定/","dgPassFrontmatter":true,"created":"2026-01-01","updated":"2026-01-01"}
---


::: hidden
帳號 : root
密碼 : 1qaz2wsx
:::
## 網路拓樸圖
---
![Pasted image 20260101201716.png](/img/user/images/Pasted%20image%2020260101201716.png)
若需要重置 SRX 可以參考 [[技術文件/SRX/重置 SRX\|重置 SRX]]

## 初始狀態(無法上網)
---
初始狀態
```
root> show interfaces terse
Interface               Admin Link Proto    Local                 Remote
fe-0/0/0                up    up
...
fe-0/0/1                up    up
fe-0/0/2                up    down
fe-0/0/3                up    down
fe-0/0/4                up    down
fe-0/0/5                up    down
fe-0/0/6                up    down
fe-0/0/7                up    down
...
---(more)---
```

fe-0/0/0  有連接數據機，但沒有被數據機配置 IP，現在要來設定 Static IP

無法 `ping 8.8.8.8`
```
root> ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
ping: sendto: Can't assign requested address
ping: sendto: Can't assign requested address
ping: sendto: Can't assign requested address
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss

root>
```

## 檢查 default gateway
---
輸入 `netstat -nr | grep default`
```
netstat -nr | grep default
default 192.168.1.1 UGScg                 en0
```

default gateway 為 `192.168.1.1`
## 檢查 arp table
---
輸入 `arp -an` 檢查未被使用的 IP
```
$ arp -an
? (192.168.1.1) at a8:c2:46:c:5a:71 on en0 ifscope [ethernet]
? (192.168.1.110) at b6:d5:a6:81:7d:64 on en0 ifscope [ethernet]
? (192.168.1.111) at ba:1d:e9:3:da:b1 on en0 ifscope [ethernet]
? (192.168.1.255) at ff:ff:ff:ff:ff:ff on en0 ifscope [ethernet]
```

以本次範例來說，可將 SRX 設定一個靜態 IP 在  `192.168.1.2/32` 因為尚未被使用

## 設定流程
---
進入 `cli`
```
root@% cli
root>
```

進入 configuration mode
```
root> configure
Entering configuration mode
The configuration has been changed but not committed

[edit]
root#
```


設定靜態 IP
Default gateway: `192.168.1.1`
```
root# set interfaces fe-0/0/0 unit 0 family inet
root# set interfaces fe-0/0/0 unit 0 family inet address 192.168.1.2/24
root# set routing-options static route 0.0.0.0/0 next-hop 192.168.1.1
```

將 fe-0/0/0.0 介面放入 untrust 區域 
```
root# set security zones security-zone untrust interfaces fe-0/0/0.0 host-inbound-traffic system-services ping
```

`commit` 保存
```
root# commit
```

輸入 `exit` 離開 configuration mode
```
root# exit
Exiting configuration mode

root>
```

## 確認能否上網
---

輸入 `show interfaces terse` 靜態 IP 已配置為 `192.168.1.2/24`
```
root> show interfaces terse
Interface               Admin Link Proto    Local                 Remote
fe-0/0/0                up    up
fe-0/0/0.0              up    up   inet     192.168.1.2/24
...
```

從 arp table 已能發現靜態 IP (`192.168.1.2`)已配置完成
```
jason@my-mac ~ % arp -an
? (192.168.1.1) at a8:c2:46:c:5a:71 on en0 ifscope [ethernet]
? (192.168.1.2) at 84:18:88:b8:82:0 on en0 ifscope [ethernet]
```

可以從 SRX ping 到 8.8.8.8
```
root> ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=116 time=4.052 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=5.899 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 4.052/4.976/5.899/0.923 ms

root>
```

## 總結
---
現在 SRX 已能透過 Default gateway 上網，下一步是要讓連接 SRX 的設備能上網

