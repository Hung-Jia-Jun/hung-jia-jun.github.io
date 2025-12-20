---
{"dg-publish":true,"permalink":"/技術文件/安裝 Obsidian 語意搜尋套件/","tags":["obsidian"],"created":"2025-12-17T21:20:25.602+08:00","updated":"2025-12-17T22:59:46.961+08:00"}
---


Plugin repo: https://github.com/bbawj/obsidian-semantic-search?tab=readme-ov-file#demo

在 Settings -> Community plugins -> Browse
![Pasted image 20251217181636.png](/img/user/images/Pasted%20image%2020251217181636.png)


搜尋 Semantic Search
![Pasted image 20251217181747.png](/img/user/images/Pasted%20image%2020251217181747.png)


安裝套件
![Pasted image 20251217181834.png](/img/user/images/Pasted%20image%2020251217181834.png)

啟用 Semantic 套件
![Pasted image 20251217181933.png](/img/user/images/Pasted%20image%2020251217181933.png)


安裝 ollama
https://ollama.com/download

下載 embeding model
```bash
ollama pull nomic-embed-text  
```


檢查 model 是否有安裝成功
```bash
$ ollama list
NAME                       ID              SIZE      MODIFIED   
nomic-embed-text:latest    0a109f422b47    274 MB    About an hour ago    
```


| 參數名稱    | 設定值                                |
| ------- | ---------------------------------- |
| API URL | `http://localhost:11434/api/embed` |
| Model   | nomic-embed-text                   |

設定參數
![Pasted image 20251217182540.png](/img/user/images/Pasted%20image%2020251217182540.png)

打開 command palette
![Pasted image 20251217182721.png](/img/user/images/Pasted%20image%2020251217182721.png)

按照以下順序執行
![Pasted image 20251217182834.png](/img/user/images/Pasted%20image%2020251217182834.png)

之後就可以用這個進行語意搜尋
![Pasted image 20251217183031.png](/img/user/images/Pasted%20image%2020251217183031.png)


整體語意搜尋結果還不錯
![Pasted image 20251217183146.png](/img/user/images/Pasted%20image%2020251217183146.png)

# 總結
透過這個 obsidian 工具，可以做到語意級別的模糊搜尋，有時候可能就是模糊的感覺，沒有明確的關鍵字，就可以考慮用這個搜尋工具進行搜尋。