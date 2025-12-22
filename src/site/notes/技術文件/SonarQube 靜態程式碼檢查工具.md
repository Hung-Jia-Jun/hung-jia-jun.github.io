---
{"tags":["#DevSecOps"],"dg-publish":true,"dg-created_time":"2025-12-22","dg-updated_time":"2025-12-22","permalink":"/技術文件/SonarQube 靜態程式碼檢查工具/","dgPassFrontmatter":true,"created":"2025-12-22","updated":"2025-12-22"}
---

# 簡介
### SonarQube
屬於 SAST(Static Application Security Testing) 工具，會做為程式品質的守門員，內含 js、python、lua 等代碼風格規則集。
### deepfence
使用 Deepfence 做 DAST(Dynamic Application Security Testing)  監控，目的是讓線上服務的進出流量都受到 OWASP 安全掃描，最終掃描結果會放到主控台


## SDLC
![Pasted image 20241211135117.png](/img/user/images/Pasted%20image%2020241211135117.png)

DAST
https://community.deepfence.io/threatmapper/docs/installation/
![Pasted image 20241211135659.png](/img/user/images/Pasted%20image%2020241211135659.png)
## Firewall

| Entity           | Port | direction |
| ---------------- | ---- | --------- |
| SonarQube Server | 9000 | inbound   |


# 使用平台
SonarQube

這次使用的平台定位在 test 之後要 release 之前
https://docs.sonarsource.com/sonarqube-community-build/
![Pasted image 20241211133427.png](/img/user/images/Pasted%20image%2020241211133427.png)


後台管理頁面
```
docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube
```

建立 project key
https://docs.sonarsource.com/sonarqube-server/latest/analyzing-source-code/scanners/sonarscanner/#configuring-your-project
在欲掃描的資料夾底下建立 sonar-project.properties
```
$ touch sonar-project.properties
```

填入 project key
```
# must be unique in a given SonarQube Server instance
sonar.projectKey=my:project
```

產出 token
![Pasted image 20241211132541.png](/img/user/images/Pasted%20image%2020241211132541.png)

取得 project key、token
![Pasted image 20241211132632.png](/img/user/images/Pasted%20image%2020241211132632.png)


帶入環境變數進行程式碼掃描
```
docker run \
--rm \
-e SONAR_HOST_URL="http://10.66.16.108:9000"  \
-e SONAR_TOKEN=sqp_4e22abc134b4d52b9fe654b7d549427abb71db90 \
-v "$(pwd)/:/usr/src" \
sonarsource/sonar-scanner-cli
```

![Pasted image 20241211134142.png](/img/user/images/Pasted%20image%2020241211134142.png)

掃描結果
![Pasted image 20241211134155.png](/img/user/images/Pasted%20image%2020241211134155.png)