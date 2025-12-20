---
{"dg-publish":true,"permalink":"/技術文件/Docker/Docker swarm 研究/","tags":["docker-swarm"],"created":"2024-12-02T23:11:42.899+08:00","updated":"2025-12-18T00:25:11.502+08:00"}
---

#docker #docker-swarm

## 啟動 docker swarm cluser
建立 swarm manager
```
$ docker swarm init
Swarm initialized: current node (vft2r90bfk02ybdxwbmomr36w) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-10fb01trc9pmy5ullxpsbroxx2mb340xe9oxea47ggdg7xdzni-7jk0smc4a66r450xlsd7isu9j 10.92.0.153:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

根據顯示的指令，將另一個 node 加入到 swarm cluster
```
$ ssh a9-test-2
$ docker swarm join --token SWMTKN-1-10fb01trc9pmy5ullxpsbroxx2mb340xe9oxea47ggdg7xdzni-7jk0smc4a66r450xlsd7isu9j 10.92.0.153:2377
This node joined a swarm as a worker.
```

查看目前 cluster 的加入指令
```
$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-10fb01trc9pmy5ullxpsbroxx2mb340xe9oxea47ggdg7xdzni-3s6hu2uirlu58ixe6nlakrr49 10.92.0.153:2377
```

管理目前集群的 node
```
$ docker node ls
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
vft2r90bfk02ybdxwbmomr36w *   jason-1    Ready     Active         Leader           27.3.1
x40thj73emfiyvi1371etqir4     jason-2    Ready     Active                          27.3.1
```

---
## 在 docker swarm 上建立一個 service
建立一個 nginx 在 swarm 上成為一個 service
```
$ docker service create --name my_web nginx
```

驗證狀態
```
$ docker service ls
ID             NAME                MODE         REPLICAS   IMAGE          PORTS
y5rdvf9k7tef   vigorous_margulis   replicated   1/1        nginx:latest
```

建立 replicas 2 的一個服務
這兩個 replica 會在這個 cluster 的兩個 node 上運行，並且將  8080 公開到 node port 上
```console
docker service create --name my_web \
                        --replicas 2 \
                        --publish published=8080,target=80 \
                        nginx
```
訪問任一台 node 的 8080 port 都可以得到 response
```
$ curl 10.92.0.154:8080
```

---

## docker swarm 建立一個  DaemonSet
如果將 mode 設定為 global，就會在每個 node 上面建一個 container(類似 k8s [daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/))
```
$ docker service create \
  --mode global \
  --publish mode=host,target=80,published=8080 \
  --name=nginx \
  nginx:latest
```

```
$ docker service ls
ID             NAME      MODE      REPLICAS   IMAGE          PORTS
wlxtyx45w4qx   nginx     global    2/2        nginx:latest
```

---
## 以 stack 形式啟動 docker swarm
支援 docker compose 格式的設定檔

docker-compose.yml
```
 services:
    web:
      image: nginx
      ports:
        - "8000:80"
      deploy:
        mode: global  <--- 變成 daemonset 形式，每個 node 都會有一個 container
    redis:
      image: redis:alpine
```

建立一個基於 docker compose 的 stack 部署
```
$ docker stack deploy --compose-file docker-compose.yml stackdemo
```

檢查是否正在運行
```
$ docker stack services stackdemo
ID             NAME              MODE         REPLICAS   IMAGE          PORTS
qyy4hxvdng72   stackdemo_redis   replicated   1/1        redis:alpine
y1b5uyxewmqn   stackdemo_web     replicated   1/1        nginx:latest   *:8000->80/tcp
```

## Log 系統

```
$ docker service logs stackdemo_web -f
```