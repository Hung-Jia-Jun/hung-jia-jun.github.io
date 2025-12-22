---
{"tags":["cloudflare","tunnel"],"dg-publish":true,"dg-created_time":"2025-12-22","dg-updated_time":"2025-12-22","permalink":"/技術文件/other/Cloudflare tunnel 研究/","dgPassFrontmatter":true,"created":"2025-12-22","updated":"2025-12-22"}
---


# 目的
手上有一台 respberry pi 3 b+
想在外網使用 ssh 連入進行個人機開發 or 實驗

參考網站
https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/use-cases/ssh/ssh-cloudflared-authentication/?utm_source=chatgpt.com

## 流程解釋(來自官網)
使用者可以透過在原生終端機中驗證 `cloudflared` ，連接到 SSH 伺服器，而無需 WARP 客户端。 此方法需要伺服器和客户端兩者都已安裝 `cloudflared` ，以及 Cloudflare 上的活躍 Zone。 流量會透過此連線被 Proxy，使用者會使用 Cloudflare Access 憑證登入伺服器。

來到 cloudflare `Overview/Tunnels` 按下「新增通道」
![Pasted image 20251218100540.png](/img/user/images/Pasted%20image%2020251218100540.png)


使用 Cloudflared 的方案
![Pasted image 20251218100617.png](/img/user/images/Pasted%20image%2020251218100617.png)

設定通道名稱
![Pasted image 20251218100643.png](/img/user/images/Pasted%20image%2020251218100643.png)

選擇 Respberrry pi 的 Debian 作業系統的選項
![Pasted image 20251218102645.png](/img/user/images/Pasted%20image%2020251218102645.png)

複製安裝指令
![Pasted image 20251218102723.png](/img/user/images/Pasted%20image%2020251218102723.png)

ssh 登入到本地的 Respberry pi 機器
並貼上安裝指令碼
![Pasted image 20250717152029.png](/img/user/images/Pasted%20image%2020250717152029.png)

安裝後啟動 cloudflared 服務
![Pasted image 20251218104151.png](/img/user/images/Pasted%20image%2020251218104151.png)

註冊完成
![Pasted image 20251218104238.png](/img/user/images/Pasted%20image%2020251218104238.png)

cloudflare 儀表板已出現註冊的機器
![Pasted image 20251218111615.png](/img/user/images/Pasted%20image%2020251218111615.png)

按「下一步」
![Pasted image 20251218111632.png](/img/user/images/Pasted%20image%2020251218111632.png)

輸入下列資訊，並按下「完成設定」
![Pasted image 20251218111700.png](/img/user/images/Pasted%20image%2020251218111700.png)

現在已建立完通道，主機端連線設定已完成
![Pasted image 20251218111725.png](/img/user/images/Pasted%20image%2020251218111725.png)

---

# client 連線端設定
參考資料: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/#macos


mac 上面安裝
```
brew install cloudflared
```


在 SSH 設定檔中變更：
```
vim ~/.ssh/config
```


輸入以下值
my-ssh.xxx.ltd 為此次 tunnel 的位置
```
Host my-ssh.xxx.ltd
ProxyCommand /usr/local/bin/cloudflared access ssh --hostname %h
```

連線成功
![Pasted image 20251218111840.png](/img/user/images/Pasted%20image%2020251218111840.png)

切換網路到外網環境
![Pasted image 20251218111902.png](/img/user/images/Pasted%20image%2020251218111902.png)

一樣是可以連線到 `my-ssh.xxx.ltd` 的 ssh 機器
![Pasted image 20251218111922.png](/img/user/images/Pasted%20image%2020251218111922.png)
