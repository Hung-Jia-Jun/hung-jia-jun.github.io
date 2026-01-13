---
{"tags":["network"],"dg-publish":true,"dg-draft":false,"dg-created_time":"2026-01-12","dg-updated_time":"2026-01-12","permalink":"/技術文件/Network/多 Router 間的跨網段拓樸/","dgPassFrontmatter":true,"created":"2026-01-12","updated":"2026-01-12"}
---


# 環境
---
採用 Cisco Packet Tracer 模擬環境搭建

# 網路拓樸圖
---
![Pasted image 20260112193725.png](/img/user/images/Pasted%20image%2020260112193725.png)

## 實作範圍
---
紅框處請見 [[技術文件/Network/跨網段網路拓樸結構\|跨網段網路拓樸結構]]

本次實作「藍框處」+ 與「紅框處」的介接
![Pasted image 20260112193712.png](/img/user/images/Pasted%20image%2020260112193712.png)

## 接線路徑
紅框處
Switch A(Fa0/1) -> Router A(2911) Gig0/0 
Switch B(Fa0/1) -> Router A(2911) Gig0/1 
Switch A(Fa0/2) -> PC A(Fa0) 
Switch B(Fa0/2) -> PC B(Fa0)

藍框處
Switch C(Fa0/1) -> Router B(2911) Gig0/1 
Switch D(Fa0/1) -> Router B(2911) Gig0/2 
Switch C(Fa0/2) -> PC C(Fa0) 
Switch D(Fa0/2) -> PC D(Fa0)

橋接處
Router B(2911) Gig0/0 -> Router A(2911) Gig0/2

# 開始設定
---
### PC C
Desktop -> IP Configuration
![Pasted image 20260112193218.png](/img/user/images/Pasted%20image%2020260112193218.png)

### PC D
Desktop -> IP Configuration
![Pasted image 20260112193352.png](/img/user/images/Pasted%20image%2020260112193352.png)
## Router 間的通訊
現在要設定 Router 間的通訊
![Pasted image 20260112194059.png](/img/user/images/Pasted%20image%2020260112194059.png)
### Router A
```
Router> enable
Router# configure terminal
Router(config)# interface gigabitEthernet 0/2
```

輸入設定 IP 的指令（格式為 `ip address [IP位址] [子網路遮罩]`）
```
Router(config-if)#ip address 10.0.0.1 255.255.255.0
```

輸入 `no shutdown` 以啟動 Gig 0/2
```
Router(config-if)#no shutdown
```


### Router B
```
Router> enable
Router# configure terminal
Router(config)# interface gigabitEthernet 0/2
```

輸入設定 IP 的指令（格式為 `ip address [IP位址] [子網路遮罩]`）
```
Router(config-if)#ip address 10.0.0.2 255.255.255.0
```

輸入 `no shutdown` 以啟動 Gig 0/2
```
Router(config-if)#no shutdown
```

## 設定與 Switch 連接的 Router B
---
![Pasted image 20260112194630.png](/img/user/images/Pasted%20image%2020260112194630.png)

### Router B(左半邊)
![Pasted image 20260112195135.png](/img/user/images/Pasted%20image%2020260112195135.png)
```
Router> enable
Router# configure terminal
Router(config)# interface gigabitEthernet 0/1
```

輸入設定 IP 的指令（格式為 `ip address [IP位址] [子網路遮罩]`）
```
Router(config-if)#ip address 192.168.3.254 255.255.255.0
```

輸入 `no shutdown` 啟動 Gig 0/1
```
Router(config-if)#no shutdown
```
離開目前 interface Gig 0/1 的設定
```
Router(config-if)# exit
Router(config)#
```
### Router B(右半邊)
設定 Gig 0/2
```
Router> enable
Router# configure terminal
Router(config)# interface gigabitEthernet 0/2
```
![Pasted image 20260112195153.png](/img/user/images/Pasted%20image%2020260112195153.png)

輸入設定 IP 的指令（格式為 `ip address [IP位址] [子網路遮罩]`）
```
Router(config-if)#ip address 192.168.4.254 255.255.255.0
```

輸入 `no shutdown` 啟動 Gig 0/2
```
Router(config-if)#no shutdown
```

# 設定靜態路由 (Static Route)
我們現在要用 ip route 指令來幫兩台路由器增加 router 資訊。
這個指令的語法是： 
```
> ip route [目標網段] [遮罩] [下一跳位址]
```

目標網段：你想去的地方。
下一跳 (Next Hop)：為了去那裡，你要把封包交給誰（鄰居路由器(Router B)的 IP）。

## Router A
設定封包去程路徑
![Pasted image 20260112201012.png](/img/user/images/Pasted%20image%2020260112201012.png)
進入 Router A 的 CLI
```
Router> enable
Router# configure terminal
Router(config)#
```

在 Router A 設定 192.168.3.0/24 與 192.168.4.0/24 的網段轉發
![Pasted image 20260112200300.png](/img/user/images/Pasted%20image%2020260112200300.png)
告訴 Router A，若目的地是 192.168.3.0/24 那就轉交給 Router B(10.0.0.2)
```
Router(config)#ip route 192.168.3.0 255.255.255.0 10.0.0.2
```

同樣的，Router B `192.168.4.0` 網段，也需要在 Router A 這邊設定
```
#Router(config)#ip route 192.168.4.0 255.255.255.0 10.0.0.2
```

## Router B
現在要設定封包回程路徑
![Pasted image 20260112201100.png](/img/user/images/Pasted%20image%2020260112201100.png)

進入 Router B 的 CLI
```
Router> enable
Router# configure terminal
Router(config)#
```

在 Router B 設定 192.168.1.0/24 與 192.168.2.0/24 的封包轉發
![Pasted image 20260112201220.png](/img/user/images/Pasted%20image%2020260112201220.png)

告訴 Router B，若封包的 Destination IP 為 192.168.1.0/24 或是 192.168.2.0/24 的話，就轉交給鄰居 Router A(10.0.0.1)
```
Router(config)#ip route 192.168.1.0 255.255.255.0 10.0.0.1

Router(config)#ip route 192.168.2.0 255.255.255.0 10.0.0.1

Router(config)# exit
```


# 網路測試
---
## PC A -> PC C
```
C:\>ping 192.168.3.1 

Pinging 192.168.3.1 with 32 bytes of data:

Reply from 192.168.3.1: bytes=32 time=10ms TTL=126
Reply from 192.168.3.1: bytes=32 time=10ms TTL=126
Reply from 192.168.3.1: bytes=32 time=10ms TTL=126
Reply from 192.168.3.1: bytes=32 time=10ms TTL=126

Ping statistics for 192.168.3.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 10ms, Maximum = 10ms, Average = 10ms

C:\>tracert 192.168.3.1

Tracing route to 192.168.3.1 over a maximum of 30 hops: 

  1   4 ms      4 ms      4 ms      192.168.1.254
  2   6 ms      6 ms      6 ms      10.0.0.2
  3   10 ms     10 ms     10 ms     192.168.3.1

Trace complete.
```
![Pasted image 20260112201900.png](/img/user/images/Pasted%20image%2020260112201900.png)
## PC C -> PC A
```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time=10ms TTL=126
Reply from 192.168.1.1: bytes=32 time=10ms TTL=126
Reply from 192.168.1.1: bytes=32 time=10ms TTL=126
Reply from 192.168.1.1: bytes=32 time=10ms TTL=126

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 10ms, Maximum = 10ms, Average = 10ms

C:\>
```
![Pasted image 20260112201920.png](/img/user/images/Pasted%20image%2020260112201920.png)
## PC A -> PC D
```
C:\>tracert 192.168.4.1

Tracing route to 192.168.4.1 over a maximum of 30 hops: 

  1   4 ms      4 ms      4 ms      192.168.1.254
  2   6 ms      6 ms      6 ms      10.0.0.2
  3   *         10 ms     10 ms     192.168.4.1

Trace complete.

C:\>ping 192.168.4.1

Pinging 192.168.4.1 with 32 bytes of data:

Reply from 192.168.4.1: bytes=32 time=10ms TTL=126
Reply from 192.168.4.1: bytes=32 time=10ms TTL=126
Reply from 192.168.4.1: bytes=32 time=10ms TTL=126
Reply from 192.168.4.1: bytes=32 time=10ms TTL=126

Ping statistics for 192.168.4.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 10ms, Maximum = 10ms, Average = 10ms

C:\>
```
![Pasted image 20260112202032.png](/img/user/images/Pasted%20image%2020260112202032.png)
## PC D -> PC A
```
Cisco Packet Tracer PC Command Line 1.0
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time=10ms TTL=126
Reply from 192.168.1.1: bytes=32 time=10ms TTL=126
Reply from 192.168.1.1: bytes=32 time=10ms TTL=126
Reply from 192.168.1.1: bytes=32 time=10ms TTL=126

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 10ms, Maximum = 10ms, Average = 10ms

```
![Pasted image 20260112202100.png](/img/user/images/Pasted%20image%2020260112202100.png)


# 總結
---
這次的拓樸結構與 [[技術文件/Network/跨網段網路拓樸結構\|跨網段網路拓樸結構]] 相似，但多了一組 Router 的設計，難點在 Router A 與 Router B 之間的網段躍點(Next-hop) 的設定，只要概念通了，其實設定就比較容易上手