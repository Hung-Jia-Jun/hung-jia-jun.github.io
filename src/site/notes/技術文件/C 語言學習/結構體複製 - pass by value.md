---
{"dg-publish":true,"permalink":"/技術文件/C 語言學習/結構體複製 - pass by value/","tags":["C-lang"],"created":"2025-08-27T08:38:57.078+08:00","updated":"2025-12-18T00:22:21.131+08:00"}
---

``` C
#include <string.h>
#include <stdio.h>

struct Car
{
    char name[30];
    double price;
} a = {.name="audi AAA", .price=100};

int main()
{
    // 複製行為
    struct Car  b = a; // 結構體複製是 pass by value
    printf("a: %p\n", &a); 
    printf("b: %p\n", &b); // 地址不同

    printf("a => name: %s, price: %f\n", a.name, a.price); // 值是一樣
    printf("b => name: %s, price: %f\n", b.name, b.price); // 值是一樣
    
}
```

結構體的複製是 pass by value

![Pasted image 20250830153021.png](/img/user/images/Pasted%20image%2020250830153021.png)

printf("%p\n", &a.name) 輸出的是自己的變數地址
printf("%p\n", a.name) 輸出指針變數指向的變數地址

在 C 語言中，字符串屬於常量，只要定義一個字符串 "Hello", 不管後續誰指向，都是指到同個記憶體位置

```C
#include <string.h>
#include <stdio.h>

struct Car
{
    char *name;
    double price;
} a = {.name="audi AAA", .price=100};

int main()
{
    // 複製行為
    struct Car  b = a; // 結構體複製是 pass by value，會產生新地址
    printf("a: %p\n", &a); 
    printf("b: %p\n", &b); // 地址不同

    /*
    結構體本身複製的是值，當值是指針變數時，複製的是該指針變數指向的地址
    */
    printf("a => name: %p, price: %p\n", &a.name, &a.price); // &a.name 顯示指針變數的地址 a => name: 0x102480000, price: 0x102480008
    printf("b => name: %p, price: %p\n", &b.name, &b.price); // &b.name 顯示指針變數的地址 b => name: 0x16d986a10, price: 0x16d986a18
    printf("a => name: %p, price: %f\n", a.name, a.price); // a.name 指向的變數地址 a => name: 0x102478600, price: 100.000000
    printf("b => name: %p, price: %f\n", b.name, b.price); // b.name 指向的變數地址 b => name: 0x102478600, price: 100.000000
    
}
```