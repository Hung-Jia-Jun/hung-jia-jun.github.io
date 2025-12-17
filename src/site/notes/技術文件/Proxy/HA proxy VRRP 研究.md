---
{"tags":["vrrp","haproxy"],"dg-publish":true,"permalink":"/技術文件/Proxy/HA proxy VRRP 研究/","dgPassFrontmatter":true}
---


#haproxy #vrrp #linux #keepalived #proxy

# VRRP(Virtual Router Redundancy Protocol)
 主要實現工具是 keepalived，他會透過 vrrp 廣播 master 的心跳，一但 master 死掉，Backup 就會代替 Master 回覆指定 IP 的 arp request。
 兩台主機上都有 HA Proxy，接收到請求後會再傳到後端 server

## 架構圖
![Pasted image 20241122113609.png](/img/user/images/Pasted%20image%2020241122113609.png)

參考教學 : https://medium.com/@abhilashkulkarni340/vrrp-and-4-simple-steps-to-set-it-up-on-ubuntu-454c46abb3b4

## 安裝
### haproxy
```
$ sudo apt install haproxy
$ haproxy -v
HA-Proxy version 2.0.33-0ubuntu0.1 2023/10/31 - https://haproxy.org/
```

### keepalived
```
$ apt-get install keepalived
$ systemctl status keepalived
● keepalived.service - Keepalive Daemon (LVS and VRRP)
     Loaded: loaded (/lib/systemd/system/keepalived.service; enabled; vendor preset: enabled)
     Active: inactive (dead)
  Condition: start condition failed at Thu 2024-11-21 13:47:53 CST; 2h 11min ago

Nov 21 13:47:53 jason-2 systemd[1]: Condition check resulted in Keepalive Daemon (LVS and VRRP) being skipped.
```
出現 inactive 是正常的，因為沒有 keepalived 的設定檔

所以要從 keepalived config sample 裡面拿到範例設定檔
```
$ cp /usr/share/doc/keepalived/samples/keepalived.conf.sample /etc/keepalived/keepalived.conf
```

## VRRP 主備切換環境
![Pasted image 20241122103718.png](/img/user/images/Pasted%20image%2020241122103718.png)

## 設定檔
jason-1(Master)、jason-2(Backup) 兩台測試機的設定檔都要放
### jason-1(Master)
/etc/haproxyhaproxy.cfg
```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

frontend firstbalance
    bind *:80
    option forwardfor
    default_backend webservers

backend webservers
    balance roundrobin
    server test-2 10.xx.x.154:8080 check <-- 這個要指定後端 server 位置
```

/etc/keepalived/keepalived.conf
```
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state MASTER # 路由器的首選狀態 - MASTER 或 BACKUP
    interface ens160 # IP 位址綁定的介面。它還必須添加到 virtual_ipaddress 部分
    unicast_src_ip 10.xx.x.153 # 目前路由器的IP位址
    unicast_peer{ 
        10.xx.x.154 # VRRP中其他路由器的IP位址
    } 
    virtual_router_id 50
    # nopreempt # nopreempt允許一個priority比較低的節點作為master，即使有priority更高的節點啟動。
    priority 101 # 優先權：該路由器在其他路由器中的優先權
    advert_int 1
    virtual_ipaddress {
	    10.xx.x.160 # 此部分用於新增虛擬 IP 位址 (VIP)。請注意它如何與 src_ip 和對等點位於同一網路中。
    }
}
```

### jason-2(Backup)
/etc/haproxyhaproxy.cfg
```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http
frontend firstbalance
    bind *:80
    option forwardfor
    default_backend webservers

backend webservers
    server test-2 172.xx.x.1:8080 check
    # option httpchk
```
/etc/keepalived/keepalived.conf
```
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}
vrrp_instance VI_1 {
    interface ens160
    state BACKUP
    unicast_src_ip 10.xx.x.154
    unicast_peer{ 
        10.xx.x.153
    } 
    virtual_router_id 50
    # nopreempt
    priority 101
    advert_int 1
    virtual_ipaddress {
	    10.xx.x.160
    }
}
```
> [!TIP] Tip: nopreempt
> 允許一個priority比較低的節點作為master，即使有priority更高的節點啟動。
> 發生情境:
>     其中一台設置為master，一台設置為backup。 當master出現異常后，backup自動切換為master。 當backup成為master后，master恢復正常后會再次搶佔成為master，導致不必要的主備切換。 因此可以將兩台keepalived初始狀態均配置為backup，設置不同的優先順序，優先順序高的設置nopreempt解決異常恢復后再次搶佔的問題。

---

## 執行
jason-1(Master)
```
$ sudo systemctl restart keepalived
```
jason-2(Backup)
```
$ sudo systemctl restart keepalived
```

jason-2 這台目前是 BACKUP mode
![Pasted image 20241121162744.png](/img/user/images/Pasted%20image%2020241121162744.png)

## 實驗
目前狀態

jason-1 - Master
jason-2 - Backup
![Pasted image 20241121163855.png](/img/user/images/Pasted%20image%2020241121163855.png)




#### jason-3 觀測機
arp table 測試
```
$ sudo arping 10.xx.x.160
60 bytes from 00:xx:xx:xx:xx:88 (10.xx.x.160): index=1 time=517.786 usec
```

現在 `10.xx.x.160` 是 jason-1(00:xx:xx:xx:88) 所負責的
![Pasted image 20241122105210.png](/img/user/images/Pasted%20image%2020241122105210.png)
查看 arp table
```
$ arp -a
...
? (10.xx.x.160) at 00:xx:xx:xx:xx:88 [ether] on ens160
...
```


關閉 jason-1(Master)
sudo systemctl stop keepalived
![Pasted image 20241122104314.png](/img/user/images/Pasted%20image%2020241122104314.png)

發現 jason-2(Backup) 主機接手 Master 位置
![Pasted image 20251217231432.png](/img/user/images/Pasted%20image%2020251217231432.png)
到 jason-3 探測機，發現已經迅速切換主備位置了
![Pasted image 20241122104433.png](/img/user/images/Pasted%20image%2020241122104433.png)

arp table 也同步更新
![Pasted image 20241122104539.png](/img/user/images/Pasted%20image%2020241122104539.png)

服務也未中斷
![Pasted image 20241122104608.png](/img/user/images/Pasted%20image%2020241122104608.png)

## VRRP工作原理
ref: https://info.support.huawei.com/info-finder/encyclopedia/zh/VRRP.html

當Master設備出現故障時，路由器B和路由器C會選舉出新的Master設備。 新的Master設備開始回應對虛擬IP位址的[ARP](https://info.support.huawei.com/info-finder/encyclopedia/zh/ARP.html "ARP")回應，並定期發送VRRP通告報文。

VRRP的詳細工作過程如下：

1. VRRP備份組中的設備根據優先順序選舉出Master。 Master設備通過發送[免費ARP](https://info.support.huawei.com/info-finder/encyclopedia/zh/ARP.html "ARP")報文，將虛擬MAC位址通知給與它連接的設備或者主機，從而承擔報文轉發任務。
2. Master設備週期性向備份組內所有Backup設備發送VRRP通告報文，通告其配置資訊（優先順序等）和工作狀況。
3. 如果Master設備出現故障，VRRP備份組中的Backup設備將根據優先順序重新選舉新的Master。
4. VRRP備份組狀態切換時，Master設備由一台設備切換為另外一台設備，新的Master設備會立即發送攜帶虛擬路由器的虛擬MAC位址和虛擬IP位址資訊的免費ARP報文，刷新與它連接的設備或者主機的MAC表項，從而把使用者流量引到新的Master設備上來，整個過程對使用者完全透明。
5. 原Master設備故障恢復時，若該設備為IP位址擁有者（優先順序為255），將直接切換至Master狀態。 若該設備優先順序小於255，將首先切換至Backup狀態，且其優先順序恢復為故障前配置的優先順序。
6. Backup設備的優先順序高於Master設備時，由Backup設備的工作方式（搶佔方式和非搶佔方式）決定是否重新選舉Master。