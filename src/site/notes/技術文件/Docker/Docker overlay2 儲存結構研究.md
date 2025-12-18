---
{"tags":["docker","overlay2"],"dg-publish":true,"permalink":"/技術文件/Docker/Docker overlay2 儲存結構研究/","dgPassFrontmatter":true}
---

`docker info` 查看storage driver

首先拉取一個Nginx鏡像，然後通過命令`docker image inspect nginx`查看Nginx鏡像的詳情，每個鏡像都會有一個Data資訊，這個資訊指示了鏡像是怎麼存的。
查看 image 的詳情

![Pasted image 20250219093919.png](/img/user/images/Pasted%20image%2020250219093919.png)

![Pasted image 20250219093927.png](/img/user/images/Pasted%20image%2020250219093927.png)

**其中 MergedDir 代表當前鏡像層在 overlay2 儲存下的目錄，LowerDir 代表當前鏡像的父層關係，使用冒號分隔，冒號最後代表該鏡像的最底層。**

接著我們進入到任意一個鏡像層看看裡面的結構
![Pasted image 20250219094120.png](/img/user/images/Pasted%20image%2020250219094120.png)

![Pasted image 20250219094158.png](/img/user/images/Pasted%20image%2020250219094158.png)

**鏡像層的`link`文件內容為該鏡像層的`短ID
```
$ cat link | xargs echo
DWWRLQXD5ZZYZFLAECXOWA7ODL
```

`lower`檔內容為該層的所有父層鏡像的`短ID`。會用冒號 : 分隔
```
$ cat lower | xargs echo
l/XDQAR5Z4MSK7WNLYDBINB26DK4:l/VPWWG52KMQWNK7K3IMSTON3SRJ:l/UQPY6GNRH4CX3GA66Q75NCAXWN:l/PFGRT5RL3UYLRSI7GXQEOXVLHL:l/IWUVFE4FQR2BE5W5YZIKOYTJWG:l/KEISEPOCFYKYXTNIQQRWTWXGZF:l/JHJ27BDVU264GX5GY2ZHZXINEG:l/HVZVYP2P3H35EVYVSPQRHK6LFQ:l/OGZYCU6IXTZROMPIVINKLUWFJS:l/4FM6QTF6W57G4JUF4S4TSC663T:l/2TRWFLEP26VVMIB2XESNCDKT5R:l/JZ6W4BFQA5LVJOYM5YYX5OHQDU:l/Z4BVWLONYV352YARKDBI4NRZGE:l/N3RDTIUUIJZRVSOAJ6XNSF6YTB:l/G3KDMCGSVUKLBPXEWVYZMPAF4L:l/3TOZSAK6KLMVF3QLT4UIMSBRPB:l/STV7FCMXVMMGPSKOFCZIA7EBOV:l/Y5FZNDARXRLXRZCDNQVL3FFEGB:l/FOI4PWVIXLO6WN7QAUJJMTFJXL
```

這時候我們再回過頭來看看上面中冒號最後的鏡像層，也就是我們Nginx鏡像父層的最底層是什麼樣的`LowerDir`
```json
  "GraphDriver": {                                                                                                                       
            "Data": {                                                                                                                          
                "LowerDir": "/var/lib/docker/overlay2/8dde6c69291ee08772fae49713a1ad2a715f5f53195e02ad921d595735316407/diff:/var/lib/docker/overlay2/f707d5dc454841a838102dbd1d702e9c7db107d8d13c25a6c4228daecb42407d/diff:/var/lib/docker/overlay2/0b39f39a877587cad676d3d2c0d078d26a058f51488a34b7918e7d3fe15b9fa6/diff:/var/lib/docker/overlay2/c25343bca009cf414ef51ec190ce5617b663cb601438eff5b03e818b648be94b/diff:/var/lib/docker/overlay2/5d925f8d2a4bc99e52980e08f3aeb08908a565e54f74023544d5a6952562e1b0/diff:/var/lib/docker/overlay2/47b9dc8bbd4059cb7689a3844f1316dc9f258c4f2d54fd3c2263411fac17a6e5/diff",                                  
                "MergedDir": "/var/lib/docker/overlay2/da9400108f56bc797caeb9ddfe6b33e71de9cd09355b8d141596ae7606f7a406/merged",               
                "UpperDir": "/var/lib/docker/overlay2/da9400108f56bc797caeb9ddfe6b33e71de9cd09355b8d141596ae7606f7a406/diff",                  
                "WorkDir": "/var/lib/docker/overlay2/da9400108f56bc797caeb9ddfe6b33e71de9cd09355b8d141596ae7606f7a406/work"                    
            },                                                                                                                                 
            "Name": "overlay2"                                                                                                                 
        },
```


可以看到它裡面沒有了檔，這是因為他就是最底層了，等於是根鏡像了。 同時，資料夾下的檔，就是我們熟悉的Linux文件目錄結構
```
root@jason-1:/var/lib/docker/overlay2/47b9dc8bbd4059cb7689a3844f1316dc9f258c4f2d54fd3c2263411fac17a6e5/diff# ll
total 68
drwxr-xr-x 17 root root 4096 Nov 25 14:44 ./
drwx--x---  3 root root 4096 Nov 25 14:44 ../
lrwxrwxrwx  1 root root    7 Nov 11 08:00 bin -> usr/bin/
drwxr-xr-x  2 root root 4096 Oct 31 19:04 boot/
drwxr-xr-x  2 root root 4096 Nov 11 08:00 dev/
drwxr-xr-x 29 root root 4096 Nov 11 08:00 etc/
drwxr-xr-x  2 root root 4096 Oct 31 19:04 home/
lrwxrwxrwx  1 root root    7 Nov 11 08:00 lib -> usr/lib/
lrwxrwxrwx  1 root root    9 Nov 11 08:00 lib64 -> usr/lib64/
drwxr-xr-x  2 root root 4096 Nov 11 08:00 media/
drwxr-xr-x  2 root root 4096 Nov 11 08:00 mnt/
drwxr-xr-x  2 root root 4096 Nov 11 08:00 opt/
drwxr-xr-x  2 root root 4096 Oct 31 19:04 proc/
drwx------  2 root root 4096 Nov 11 08:00 root/
drwxr-xr-x  3 root root 4096 Nov 11 08:00 run/
lrwxrwxrwx  1 root root    8 Nov 11 08:00 sbin -> usr/sbin/
drwxr-xr-x  2 root root 4096 Nov 11 08:00 srv/
drwxr-xr-x  2 root root 4096 Oct 31 19:04 sys/
drwxrwxrwt  2 root root 4096 Nov 11 08:00 tmp/
drwxr-xr-x 12 root root 4096 Nov 11 08:00 usr/
drwxr-xr-x 11 root root 4096 Nov 11 08:00 var/
```

## 容器怎麼存儲
啟動一個Nginx容器，以便觀察Docker創建的容器可寫層，並查看它的配置資訊
```
$ docker run --name=nginx -d -v /tmp/test.txt:/tmp/test.txt nginx
$ docker inspect nginx
```

進入容器層目錄，看看裡面的結構
![Pasted image 20250219100023.png](/img/user/images/Pasted%20image%2020250219100023.png)

![Pasted image 20250219100102.png](/img/user/images/Pasted%20image%2020250219100102.png)

進入容器內部
```
$ docker exec -it nginx bash
```

在容器內部建一個檔案，輸入`ls -i`，會輸出 inode id 為 677855
![Pasted image 20250219102651.png](/img/user/images/Pasted%20image%2020250219102651.png)

在宿主機的 diff 層也可以看到該檔案的 inode 為 677855
![Pasted image 20250219102626.png](/img/user/images/Pasted%20image%2020250219102626.png)

#### 容器與鏡像的寫時複製技術
前面我們說到了，一個鏡像的多個容器用到的文件系統就是鏡像的文件系統
![Pasted image 20250219103549.png](/img/user/images/Pasted%20image%2020250219103549.png)

為了保證容器的修改不會互相影響，Docker採用了寫時複製技術。

>Copy-on-Write特性：
	當容器啟動時，一個新的可寫層被載入到鏡像的頂部。 這一層被稱之為「容器層」，容器層之下的都叫做「鏡像層」。
	所有對容器的改動，無論添加、刪除，還是修改檔都只會發生在容器層中。 只有容器層是可以寫的，容器層下面的所有鏡像層都是只讀的。
	我們在容器中進行操作時：
		- 添加檔: 在容器中創建檔時，新檔被添加到容器層中。
		- 讀取檔: 在容器中讀取某個檔時，Docker會從上往下依次在各鏡像層中查找此檔。 一旦找到，打開並讀入記憶體。
		- 修改檔: 在容器中修改已存在的檔時，Docker會從上往下依次在各鏡像層中查找此檔。 一旦找到，立即將其複製到容器層，然後修改。
		- 刪除檔案: 在容器中刪除檔時，Docker也是從上往下依次在各鏡像層中查找此檔。 找到后，會在容器層中記錄下此刪除操作。
	只有當需要修改時才複製一份數據，這種特性被稱作Copy-on-Write。 可見，容器層保存的鏡像變化的部分，不會對鏡像本身進行任何修改。
	寫時複製不僅節省空間，而且還減少了容器啟動時間。 當你創建一個容器（或者來自同一個鏡像的多個容器）時，Docker 只需要創建可寫容器層即可。

### **驗證寫時複製技術的過程**
---
```shell
$ docker inspect nginx | jq '.[0].GraphDriver.Data.LowerDir' | awk -F: '{for (i=1; i<=NF; i++) print "Layer " i ": " $i}'| sed 's/"//g'
Layer 1: /var/lib/docker/overlay2/04ad8e4...3b037397e8bdbc5-init/diff
Layer 2: /var/lib/docker/overlay2/da9400108f56...406/diff
Layer 3: /var/lib/docker/overlay2/8dde6c6929...6407/diff
Layer 4: /var/lib/docker/overlay2/f707d5dc454841a8381...407d/diff
Layer 5: /var/lib/docker/overlay2/0b39f39a877587c...9fa6/diff
Layer 6: /var/lib/docker/overlay2/c25343bca0...94b/diff
Layer 7: /var/lib/docker/overlay2/5d9...544d5a6952562e1b0/diff
Layer 8: /var/lib/docker/overlay2/47b9dc1f...ac17a6e5/diff <-- 最後一層為容器層
```

![Pasted image 20250219105615.png](/img/user/images/Pasted%20image%2020250219105615.png)

再將合併結果寫成跟 Layer 1 去掉 `-init` 的同樣的目錄名稱作為工作目錄
![Pasted image 20250219105638.png](/img/user/images/Pasted%20image%2020250219105638.png)

![Pasted image 20250219105831.png](/img/user/images/Pasted%20image%2020250219105831.png)

### 測試在容器內部修改鏡像層已存在的檔案
---
### Before

鏡像層
```
$ find $(docker inspect nginx | jq -r '.[0].GraphDriver.Data.LowerDir' | tr ':' ' ') -type f -name "nginx.conf" | xargs ls -i

1882130 /var/snap/docker/common/var-lib-docker/overlay2/227a9d016be667d656c65c1c5440071d45faeb83db145c5482ef812b6c47979f/diff/etc/nginx/nginx.conf
```

容器內
```
root@592347294aa1:/etc/nginx# ls -i | grep nginx

1882130 nginx.conf
```

```
$ UPPER_DIR=$(docker inspect nginx | jq -r '.[0].GraphDriver.Data.UpperDir')
$ find "$UPPER_DIR" -type f -name "nginx.conf"
<在 Upper dir 內找不到 nginx.conf檔案，因為現在該檔案在鏡像層只讀> 
```

## After
在 container 內修改檔案
```
root@eae0f9d12762:/etc/nginx# echo 123 >> nginx.conf
```

原始 inode 未改變
```
root@592347294aa1:/etc/nginx# ls -i | grep nginx

1882130 nginx.conf
```


從宿主機上看在 upper dir 內的 inode 已改變，代表 docker「寫時複製功能」生效，當檔案在鏡像層被修改時，會複製一份到讀寫層
```
$ UPPER_DIR=$(docker inspect nginx | jq -r '.[0].GraphDriver.Data.UpperDir')
$ find "$UPPER_DIR" -type f -name "nginx.conf" | xargs ls -i

1884494 /var/snap/docker/common/var-lib-docker/overlay2/649e271405128d7d52d1b53ea1fef0c7589c191529afe0f89be89ef28778efcf/diff/etc/nginx/nginx.conf
```

```
root@jason-1:/# tail -10 /var/snap/docker/common/var-lib-docker/overlay2/3434bfba1c899d2d7f5bdf407e284410561dc3c5e5b88143072739bbc33e791c/diff/etc/nginx/nginx.conf
    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
123 <-- 被修改
```
原本處於鏡像層的檔案還是沒改變
```
# find $(docker inspect nginx | jq -r '.[0].GraphDriver.Data.LowerDir' | tr ':' ' ') -type f -name "nginx.conf" | xargs ls -i

1882130 /var/snap/docker/common/var-lib-docker/overlay2/227a9d016be667d656c65c1c5440071d45faeb83db145c5482ef812b6c47979f/diff/etc/nginx/nginx.conf
```

```
root@jason-1:/# tail -10 /var/snap/docker/common/var-lib-docker/overlay2/227a9d016be667d656c65c1c5440071d45faeb83db145c5482ef812b6c47979f/diff
/etc/nginx/nginx.conf

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```
原因:
- 在容器內部修改檔案時，**inode 不會更新**，因為 OverlayFS 只是在上層可寫層中覆蓋檔案內容，而不會對原始鏡像層中的檔案結構進行修改。只有當檔案被刪除或完全替換時，inode 才會更新。

```
$ MERGED_DIR=$(docker inspect nginx | jq -r '.[0].GraphDriver.Data.MergedDir')
$ find "$MERGED_DIR" -type f -name "nginx.conf" | xargs ls -i
```

結論 :
vim 會在修改檔案時，預設會刪除並覆蓋，所以會改變 inode id，當在 container 內改檔案時， inode id 改變造成 overlay2 無法同步跟外部這份檔案