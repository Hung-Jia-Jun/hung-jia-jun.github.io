---
{"dg-publish":true,"permalink":"/技術文件/C 語言學習/malloc 內存分配/","tags":["C-lang"],"created":"2025-12-18T21:27:34.271+08:00","updated":"2025-12-18T21:27:34.271+08:00"}
---

malloc
函數原型
```C
void *malloc(unsigned int size); // size 類型為無符號整型
```
無符號整型 = 0 or 正整數

因為少了負數，可以將所有位元用於顯示正整數

malloc 函數返回的是一個指針，所以他是一個指針函數

作用: 在內存的動態儲存區(stack 區) 中分配一個長度為 size 的連續空間，並將該空間的首地址做為函數值返回，即此函數為==指針函數==