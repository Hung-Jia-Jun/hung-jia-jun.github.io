---
{"tags":["network","Cisco"],"dg-publish":true,"dg-draft":false,"dg-created_time":"2026-01-12","dg-updated_time":"2026-01-12","permalink":"/技術文件/Cisco Packet Tracer - 跨網段網路拓樸結構/","dgPassFrontmatter":true,"created":"2026-01-12","updated":"2026-01-12"}
---


# 拓樸圖
---
![Pasted image 20260112141119.png](/img/user/images/Pasted%20image%2020260112141119.png)

## 接線路徑
SwitchA(Fa0/1) -> RouterA(2911) Gig0/0 
SwitchB(Fa0/1) -> RouterA(2911) Gig0/1 
SwitchA(Fa0/2) -> PCA(Fa0) 
SwitchB(Fa0/2) -> PCB(Fa0)

## 開始設定
---
### PC A
{ #9b539b}


Desktop -> IP Configuration
![Pasted image 20260112141222.png](/img/user/images/Pasted%20image%2020260112141222.png)
### PC B
Desktop -> IP Configuration
![Pasted image 20260112140729.png](/img/user/images/Pasted%20image%2020260112140729.png)

### Route A 設定
---
我們現在先針對連接 **Switch A** 的 **Gig0/0** 進行設定。
請在 Router 的 CLI 模式下輸入以下指令：

```
Router> enable
Router# configure terminal
Router(config)# interface gigabitEthernet 0/0
```

現在我們要為 PC A 這端做預設閘道(Default Gateway)
![Pasted image 20260112141240.png](/img/user/images/Pasted%20image%2020260112141240.png)

輸入設定 IP 的指令（格式為 `ip address [IP位址] [子網路遮罩]`）
```
Router(config-if)#ip address 192.168.1.254 255.255.255.0
```

輸入 `no shutdown` 以啟動 Gig 0/0
```
Router(config-if)#no shutdown
```

現在左半邊已設定完畢
![Pasted image 20260112141746.png](/img/user/images/Pasted%20image%2020260112141746.png)

接下來要設定右半邊
![Pasted image 20260112141800.png](/img/user/images/Pasted%20image%2020260112141800.png)

請繼續在 Router 的 CLI 模式下輸入以下指令：
```
Router> enable
Router# configure terminal
Router(config)# interface gigabitEthernet 0/1
```

輸入設定 IP 的指令（格式為 `ip address [IP位址] [子網路遮罩]`）
```
Router(config-if)#ip address 192.168.2.254 255.255.255.0
```

輸入 `no shutdown` 以啟動 Gig 0/1
```
Router(config-if)#no shutdown
```

退出 config 模式
```
exit
```

## ip route table
檢查 Route A 的 ip route table
```
Router#show ip route

Codes: L - local, C - connected
...
Gateway of last resort is not set
192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C 192.168.1.0/24 is directly connected, GigabitEthernet0/0
L 192.168.1.254/32 is directly connected, GigabitEthernet0/0

192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C 192.168.2.0/24 is directly connected, GigabitEthernet0/1
L 192.168.2.254/32 is directly connected, GigabitEthernet0/1
```

### 解釋
Router A 的 GigabitEthernet0/0 已跟 192.168.1.0/24 網段建立連接(C - connected)
```
C 192.168.1.0/24 is directly connected, GigabitEthernet0/0
```

Router A 的 GigabitEthernet0/0 已被配置為 `192.168.1.254/32` 而且為(L - local)
，代表在 192.168.1.0/24 這個網段內，路由器在 192.168.1.254/32。
當 PC A 想要把資料送往其他網段時，它會把封包丟給這個地址。
詳見 [[技術文件/Cisco Packet Tracer - 跨網段網路拓樸結構#^9b539b\|Cisco Packet Tracer - 跨網段網路拓樸結構#^9b539b]] PC A 的 Default Gateway 設定
```
L 192.168.1.254/32 is directly connected, GigabitEthernet0/0
```

## 網路連線測試
---
Desktop -> Command Prompt
![Pasted image 20260112143518.png](/img/user/images/Pasted%20image%2020260112143518.png)
PC A(192.168.1.1) - > PC B(192.168.2.1)
```
C:\>ping 192.168.2.1
Pinging 192.168.2.1 with 32 bytes of data:
Reply from 192.168.2.1: bytes=32 time=8ms TTL=127
Reply from 192.168.2.1: bytes=32 time=8ms TTL=127
Reply from 192.168.2.1: bytes=32 time=8ms TTL=127
Reply from 192.168.2.1: bytes=32 time=8ms TTL=127

Ping statistics for 192.168.2.1:
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
Minimum = 8ms, Maximum = 8ms, Average = 8ms
```


PC B(192.168.2.1) -> PC A(192.168.1.1)
```
C:\>ping 192.168.1.1
Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time=15ms TTL=127
Reply from 192.168.1.1: bytes=32 time=15ms TTL=127
Reply from 192.168.1.1: bytes=32 time=16ms TTL=127
Reply from 192.168.1.1: bytes=32 time=17ms TTL=127

Ping statistics for 192.168.1.1:
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
Minimum = 15ms, Maximum = 17ms, Average = 15ms
```

## 總結
---
透過 ㄇ型拓樸結構
![Pasted image 20260112143727.png](/img/user/images/Pasted%20image%2020260112143727.png)

已經能使用 Router 實現跨網段的通訊