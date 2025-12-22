---
{"dg-publish":true,"dg-created_time":"2025-12-22","dg-updated_time":"2025-12-22","tags":["演算法","queue","linkedlist"],"permalink":"/技術文件/其他/演算法/實作 queue (Linked list)/","dgPassFrontmatter":true,"created":"2025-12-22","updated":"2025-12-22"}
---





以雙向鍊表做這個 queue 資料結構
## 流程

### 新增物件
1. 建立 HEAD 與 Oldest 作為基礎框架
![Pasted image 20241119192535.png](/img/user/images/Pasted%20image%2020241119192535.png)
2. 新增物件到頭位置
![Pasted image 20241119192725.png](/img/user/images/Pasted%20image%2020241119192725.png)
3. 調整鍊表 node 的 next 與 previous reference
 ![Pasted image 20241119192802.png](/img/user/images/Pasted%20image%2020241119192802.png)
 4. 更新 HEAD 位置
 ![Pasted image 20241119192911.png](/img/user/images/Pasted%20image%2020241119192911.png)

完成新增 node 到 linked list 頭部的部分

### 移除物件
1. 目前狀態
![Pasted image 20241119193023.png](/img/user/images/Pasted%20image%2020241119193023.png)
2. 移動 Oldest 指針到 Oldest 物件的上一個節點
![Pasted image 20241119194633.png](/img/user/images/Pasted%20image%2020241119194633.png)
3. 移除參考
![Pasted image 20241119194708.png](/img/user/images/Pasted%20image%2020241119194708.png)


Example code
```
# FIFO calss
class LinkElement:
    def __init__(self) -> None:
        self.next = None
        self.previous = None
        self.message = ''

class Queue:
    def __init__(self) -> None:
        self.head = LinkElement()
        self.oldest = LinkElement()
        self.head.next = self.oldest
        self.oldest.previous = self.head


    def push(self, message: str):
        link_ele = LinkElement()
        link_ele.message = message
        link_ele.next = self.head
        self.head.previous = link_ele
        self.head = link_ele

    def pull(self):
        try:
            msg = self.oldest.message
            if self.oldest.previous:
                self.oldest = self.oldest.previous
                self.oldest.next = None
            return msg
        except Exception as e:
            return None

queue = Queue()

for i in range(10):
    print (str(i))
    queue.push(str(i))

for i in range(12):
    print(queue.pull())
```

output
```
0
1
2
3
4
5
6
7
8
9


0
1
2
3
4
5
6
7
8
9
```