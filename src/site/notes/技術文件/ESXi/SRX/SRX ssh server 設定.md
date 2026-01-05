---
{"tags":null,"dg-publish":true,"dg-draft":false,"dg-created_time":"2026-01-05","dg-updated_time":"2026-01-05","permalink":"/技術文件/ESXi/SRX/SRX ssh server 設定/","dgPassFrontmatter":true,"created":"2026-01-05","updated":"2026-01-05"}
---


## 設定 SSH
---
```
configure
set system services ssh
set system services ssh root-login allow
set system root-authentication plain-text-password
commit
```

## 在安全區域上允許 SSH 流量   
---
需要確認 10.77.18.12 這個 IP 所在的介面是屬於哪個安全區域 (security zone)。
```
root> show security zones
...
Security zone: untrust
  Send reset for non-SYN session TCP packets: Off
  Policy configurable: Yes
  Interfaces bound: 1
  Interfaces:
    fe-0/0/7.0
```
目前配置 fe-0/0/7.0 屬於 untrust zone

為了允許 ssh 流量來到 "untrust" 區域，請執行以下指令：
```
configure
set security zones security-zone untrust interfaces fe-0/0/7.0 host-inbound-traffic system-services ssh
commit
```

設定連線資訊在 ~/.ssh/config
```
Host srx-firewall
	HostName 10.7x.1x.1x
	User root
	KexAlgorithms +diffie-hellman-group1-sha1
	HostKeyAlgorithms +ssh-rsa
```

ssh 連線
```
ssh srx-firewall
```


