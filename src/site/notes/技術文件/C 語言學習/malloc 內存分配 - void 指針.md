---
{"dg-publish":true,"permalink":"/技術文件/C 語言學習/malloc 內存分配 - void 指針/","tags":["C-lang"],"created":"2025-08-30T18:45:03.103+08:00","updated":"2025-12-18T00:22:48.168+08:00"}
---

![Pasted image 20250830184511.png](/img/user/images/Pasted%20image%2020250830184511.png)

![Pasted image 20250830190855.png](/img/user/images/Pasted%20image%2020250830190855.png)

```C
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

// 無類型指針

// 結構體指針
int main()
{
    int x = 10;
    int *p = &x;
    printf("%d\n", *p);

    
    // 無類型的指針變數可以與其他類型的指針相賦值
    void *q = &x; // 無類型指針
    q = p;
    int *r = q;
    double *s = q;
    
    // int y = *q; // 無法解讀 void 指針類型的數據
    int y = *((int *)q); // 強制轉型
    printf("%d\n", *r);
    printf("%d\n", y);
    printf("%f\n", *s); // 不要嘗試讀取其他類型的指針，會導致不可預期的行為
}
```