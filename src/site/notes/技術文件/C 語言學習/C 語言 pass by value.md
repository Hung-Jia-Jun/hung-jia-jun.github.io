---
{"dg-publish":true,"permalink":"/技術文件/C 語言學習/C 語言 pass by value/","tags":["C-lang"],"created":"2025-08-31T19:14:18.406+08:00","updated":"2025-12-18T00:22:37.093+08:00"}
---

C 語言沒有 pass by reference, 只有 pass by value，這是語言特性

就算是 pointer 也是 pass by value

若要實現 pass by reference, 需借助指針的功能

```C
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

void setNum(int *p)
{
    *p = 10; // 解引用後修改其值
}
int main()
{
    int a=100;
    printf("a: %d\n", a); // 100
    setNum(&a); // 將 a 的地址傳進 function
    printf("a: %d\n", a); // 10
}
```