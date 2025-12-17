---
{"tags":["nginx","linux","gdb"],"dg-publish":true,"permalink":"/技術文件/使用 pmap、smaps 與 gdb Dump 出 Nginx Worker 記憶體區段教學/","dgPassFrontmatter":true}
---




# 主旨

```
本教學示範如何透過 Linux 的 pmap、/proc/<pid>/smaps 與 gdb 取得指定 process 的匿名記憶體區段，並導出成 dump 檔供後續分析。
```
---

## 0. 找出 nginx process
![Pasted image 20251110210252.png](/img/user/images/Pasted%20image%2020251110210252.png)

## 1. 找出記憶體佔用最高的區段

首先使用 pmap 檢查 Nginx worker 的 memory map，並依 RSS 由大到小排序：

```
$ pmap -x 135 | sort -k 3 -n -r | head -3
total kB         2709340  412896  407496
00007fe347600000  468992  393644  393644 rw---   [ anon ]
00007fe3423ee000    5632    3860    3860 rw---   [ anon ]
```

重點：

- `RSS` 欄位表示實際佔用的實體記憶體。
- 第一筆最大者：`00007fe347600000`，RSS 約 393MB。
- 標記為 `[ anon ]`，通常代表匿名 mmap 或 GC heap、buffer、cache 等。
---

## 2. 用 smaps 確認該地址區段的完整範圍
pmap 顯示的是區段起始地址，但實際區段長度需從 smaps 查詢。

```
$ cat /proc/135/smaps | grep 7fe347600000
7fe34296e000-7fe347600000 r--s 00000000 00:36 2640400 /data/nginx/ip2proxy/PX2_CUSTOMER.mmdb
7fe347600000-7fe364000000 rw-p 00000000 00:00 0
```

解讀：

- 第一段：`7fe34296e000-7fe347600000` 是 memory-mapped file (`PX2_CUSTOMER.mmdb`)
- 第二段：`7fe347600000-7fe364000000` 是匿名記憶體 (`rw-p ... 0`)  
    這就是 pmap 顯示為 `[ anon ]` 的那一段。

因此要 dump 的實際區段範圍為：
```
start: 0x7fe347600000 
end:   0x7fe364000000
```

---

## 3. 使用 gdb attach 進程

進入 gdb：

`$ gdb -pid 135`

若出現權限問題，需確認：

- 容器需加上 `--cap-add=SYS_PTRACE --security-opt seccomp=unconfined`
    
- 或需在宿主機設置 `kernel.yama.ptrace_scope=0`
    

---

## 4. 在 gdb 裡 dump 出記憶體檔案

進入 gdb 之後，執行：

`(gdb) dump memory /tmp/memdump_1 0x7fe347600000 0x7fe364000000`

說明：

- `/tmp/memdump_1` 為輸出檔案路徑
    
- 開始地址與結束地址對應 smaps 的匿名記憶體範圍
    

假如 dump 成功，會在 `/tmp` 看到數百 MB 至數 GB 的檔案。

---

## 5. 後續分析 dump 檔案

- 讀取可見字串：
    

`$ strings /tmp/memdump_1 | less`

![Pasted image 20251110210134.png](/img/user/images/Pasted%20image%2020251110210134.png)