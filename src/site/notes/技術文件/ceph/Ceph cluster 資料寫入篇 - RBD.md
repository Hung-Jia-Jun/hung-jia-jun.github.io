---
{"tags":["ceph","docker"],"linklist":["[[link.tech.ceph]]"],"dg-publish":true,"dg-draft":false,"dg-created_time":{"{ date }":null},"dg-updated_time":{"{ date }":null},"permalink":"/技術文件/ceph/Ceph cluster 資料寫入篇 - RBD/","dgPassFrontmatter":true,"created":{"{ date }":null},"updated":{"{ date }":null}}
---


進入 ceph-mon
```
docker exec -it ceph-mon bash
```
建立 osd pool
```
osd pool create test-rbd 128 128
```
  
顯示所有 pool
```
ceph osd pool ls
```


```
ceph osd pool application enable test-rbd rbd
```

  

在已建立的 osd pool 內，建立一個 test-data-image
```
rbd create test-data-image --size 20G --pool test-rbd --image-format 2 --image-feature layering
```
  
```
rbd create --pool test-rbd --image rbd-demo1.img --size 10G
```

```
rbd ls --pool test-rbd
```

  

```
rbd --image test-data-image --pool test-rbd info
```
  
  

```
$ ceph auth get-or-create client.testuser mon 'allow r' osd 'allow * pool=test-rbd' -o /var/lib/ceph/client-auth/ceph.keyring
```

回到宿主機上
```
mkdir -p /var/lib/ceph/client-auth/
cp /var/lib/ceph/client-auth/ceph.keyring /etc/ceph/client-auth/ceph.keyring
```

安裝 docker rbd plugin (每個 node 都要)
```
docker plugin install wetopi/rbd \
  --alias=wetopi/rbd \
  LOG_LEVEL=1 \
  RBD_CONF_POOL="test-rbd" \
  RBD_CONF_CLUSTER=ceph \
  RBD_CONF_KEYRING_USER=client.testuser
```

```
docker plugin ls
ID             NAME                DESCRIPTION             ENABLED
029dbbd2e032   wetopi/rbd:latest   RBD plugin for Docker   true
```

```
docker volume create -d wetopi/rbd my_rbd_volume
```

https://github.com/wetopi/docker-volume-rbd

https://rdwaykos.medium.com/ceph-rbd-for-docker-volume-fa2884cedaf5

測試掛載剛剛建立的 volume
```
docker run -it --rm \
  --volume my_rbd_volume:/data:ro \
  --network ceph-network \
  --volume-driver=wetopi/rbd \
  ubuntu:latest bash
```

連線到 ceph mon 可以看到剛剛建立的 volume 已經被 ceph 託管了
```
$ rbd ls test-rbd
my_rbd_volume
```


將建立好的 ceph.keyring 同步到其他台主機的 /etc/ceph
```
sudo scp -i /home/ubuntu/.ssh/id_rsa -r /etc/ceph ubuntu@192.168.1.51:~/ceph
```

```
$ sudo mv ./ceph.keyring /etc/ceph
$ ls /etc/ceph
ceph.keyring
```

## 重要事項
RBD 不支援一個 volume 被多個 container 掛載(readwrite many)，只有 ReadWriteOnce 模式