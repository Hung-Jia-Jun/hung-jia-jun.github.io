---
{"tags":["vsrx","ESXi"],"dg-publish":true,"dg-draft":false,"dg-created_time":{"{ date }":null},"dg-updated_time":{"{ date }":null},"permalink":"/技術文件/ESXi/在 ESXi 以 vSRX 實作單臂路由/","dgPassFrontmatter":true,"created":{"{ date }":null},"updated":{"{ date }":null}}
---

## 建立 vsrx 機器
---
下載與試用 vsrx
https://www.juniper.net/us/en/dm/download-next-gen-vsrx-firewall-trial.html


1. 啟動並登入 vsrx![Pasted image 20251226154318.png](/img/user/images/Pasted%20image%2020251226154318.png)
2. 新增連接埠群組 test-1![Pasted image 20251226154910.png](/img/user/images/Pasted%20image%2020251226154910.png)
3. 新增連接埠群組 test-2![Pasted image 20251226155140.png](/img/user/images/Pasted%20image%2020251226155140.png)
4. 設定 trunk 讓所有封包都走來這個 port-group![Pasted image 20251226160743.png](/img/user/images/Pasted%20image%2020251226160743.png)
5. 將 trunk port-group 綁到 vSRX![Pasted image 20251226160908.png](/img/user/images/Pasted%20image%2020251226160908.png)

## 註冊並設定 vlan 到 vSRX
---
### 新增一個 VLAN
---
新增 
- ESXi port-group name: test-1
- VLAN ID: 888
- Subnet：`10.10.106.0/24`(自行指定)
- Gateway：`10.10.106.254`

新增 
- ESXi port-group name: test-2
- VLAN ID: 123
- Subnet：`10.10.107.0/24`  (自行指定)
- Gateway：`10.10.107.254`

## 網路拓樸圖
---
pg-trunk-vsrx VLAN ID 4095
test-1 VLAN ID 888
test-2 VLAN ID 123
![Pasted image 20251227095327.png](/img/user/images/Pasted%20image%2020251227095327.png)
## 開始設定 vSRX
---
輸入 root 登入
並輸入 cli 進入 cli 模式
![Pasted image 20251226161349.png](/img/user/images/Pasted%20image%2020251226161349.png)

檢查目前 interface 狀態![Pasted image 20251226161557.png](/img/user/images/Pasted%20image%2020251226161557.png)

## 指令解釋
---
```
set interfaces ge-0/0/0 unit 888 vlan-id 888 family inet address 10.10.106.254/24
```
- set interfaces
	功能：設定網路介面（interface）。
	簡單說，就是告訴 Juniper「我要修改或新增一個網路介面設定」。
- ge-0/0/0
	功能：指定要設定的實體介面。
	ge = Gigabit Ethernet（千兆以太網）
	0/0/0 = Slot / PIC / Port（Juniper 介面的命名方式）
	簡單說，就是選定一塊實體網卡。

- unit 888
	功能：指定子介面（sub-interface）的編號。
	Juniper 允許一個實體介面拆成多個子介面，用於不同 VLAN。
	這裡 unit 888 就是給這個子介面編號為 888。
- vlan-id 888
	功能：指定這個子介面對應的 VLAN ID。
	VLAN 888 的流量會走到這個子介面上。
	換句話說，這個子介面屬於 VLAN 888，這個值對應到的是 ESXi 裡的 port-group 定義的 VLAN ID。

- family inet
	功能：指定 IP 層類型，這裡是 IPv4。
	Juniper 的介面可以同時支援多種協議（例如 IPv4, IPv6, MPLS），用 family 來區分。
	這裡的 family inet 就是 IPv4。
	如果你要用 IPv6，就會寫成 family inet6。
		family = 協議類型
		inet = IPv4
		inet6 = IPv6

- address 10.10.106.254/24
	功能：設定這個子介面的 IP 位址及子網路遮罩。
	10.10.106.254 = IP
	/24 = 子網路遮罩 255.255.255.0
	這個 IP 就是 VLAN 888 的 Layer 3 閘道（gateway）或 VM 要配發的 IP 網段。

## 設定 root 帳戶權限
---
```bash
set system root-authentication plain-text-password
```

## 開始設定 VLAN
---
進入設定模式：

```bash fold title=configure
# 1. 啟動 Tagging
set interfaces ge-0/0/0 vlan-tagging

# 2. 定義 Subinterfaces (L3)
set interfaces ge-0/0/0 unit 888 vlan-id 888 family inet address 10.10.106.254/24
set interfaces ge-0/0/0 unit 123 vlan-id 123 family inet address 10.10.107.254/24

# 3. 安全區域設定 (允許介面通訊與 Ping 自機)
set security zones security-zone trust interfaces ge-0/0/0.888 host-inbound-traffic system-services ping
set security zones security-zone trust interfaces ge-0/0/0.123 host-inbound-traffic system-services ping

# 4. 預設全開 Policy (Trust to Trust)
set security policies from-zone trust to-zone trust policy allow-all match source-address any destination-address any application any
set security policies from-zone trust to-zone trust policy allow-all then permit

commit
```

輸入
```
show interfaces ge-0/0/0
```

vlan 888 與 vlan 123 已設定完成
![Pasted image 20251227100135.png](/img/user/images/Pasted%20image%2020251227100135.png)

## 下一步
---
依以下步驟建立兩台 VM，
[[技術文件/ESXi/建立虛擬機器並綁定 VLAN\|建立虛擬機器並綁定 VLAN]]
VM1 :
- VM name: test-1
- VLAN ID: 888
- Address: `10.10.106.1/32`
- Subnet：`10.10.106.0/24`
- Gateway：`10.10.106.254`
- 
VM2 :
- VM name: test-2
- VLAN ID: 123
- Address: `10.10.107.1/32`
- Subnet：`10.10.107.0/24` 
- Gateway：`10.10.107.254`
- 依序填入下列資訊![Pasted image 20251227103559.png](/img/user/images/Pasted%20image%2020251227103559.png)

## vSRX 設定檢查
---
### 檢查機器是否有廣播 ARP
輸入
```
show arp
```

![Pasted image 20251227104844.png](/img/user/images/Pasted%20image%2020251227104844.png)

兩台新建立的機器已經開始廣播 ARP 了

### 查看 vSRX interface 設定
輸入
```
show configuration interfaces
```
![Pasted image 20251227110207.png](/img/user/images/Pasted%20image%2020251227110207.png)

### 檢查 security zone
輸入
```
show security zones
```
![Pasted image 20251227110533.png](/img/user/images/Pasted%20image%2020251227110533.png)

## 測試路由狀況
---
從 test-1 ping 到 test-2 (ping OK)
![Pasted image 20251227105248.png](/img/user/images/Pasted%20image%2020251227105248.png)

從 test-2 ping 到 test-1 (ping OK)
![Pasted image 20251227105322.png](/img/user/images/Pasted%20image%2020251227105322.png)

暫停 vSRX 後測試 (ping 不通)
![Pasted image 20251227105439.png](/img/user/images/Pasted%20image%2020251227105439.png)

兩台 VM 無法互通![Pasted image 20251227105537.png](/img/user/images/Pasted%20image%2020251227105537.png)

恢復 vSRX 後測試 (ping OK)
![Pasted image 20251227105637.png](/img/user/images/Pasted%20image%2020251227105637.png)

### 測試 80 port
test-2 -> test-1(port 80 OK)
![Pasted image 20251227111916.png](/img/user/images/Pasted%20image%2020251227111916.png)
test-1 -> test-2(port 80 OK)
![Pasted image 20251227112035.png](/img/user/images/Pasted%20image%2020251227112035.png)
# 總結
---
透過在 ESXi 上建立 VLAN 連接埠群組，並在 vSRX 上設定對應的子介面與 VLAN ID，可以成功實現單臂路由，讓不同 VLAN 的虛擬機器能夠互相通訊。
