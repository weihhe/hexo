title: C/C++笔试题总结
author: weihehe
date: 2024-10-07 18:15:51
tags:
---
笔试题
<!--more-->
1. 内存对齐
2. 联合体大小
3. size的计算

```cpp
#include <stdio.h>

int main() {
    char *al[] = {"hello!", "tclmob"};
    char a2[] = "\0";
    char a3[][8] = {"hello!", "tclmob"};
    char *p1 = "hello";
    char **p2 = al;
    char a4[] = {'h', '\0'};
    int a5[] = {'h', '\0'};

    printf("sizeof(al) = %zu\n", sizeof(al));
    printf("sizeof(a2) = %zu\n", sizeof(a2));
    printf("sizeof(a3) = %zu\n", sizeof(a3));
    printf("sizeof(p1) = %zu\n", sizeof(p1));
    printf("sizeof(p2) = %zu\n", sizeof(p2));
    printf("sizeof(a4) = %zu\n", sizeof(a4));
    printf("sizeof(a5) = %zu\n", sizeof(a5));

    return 0;
}

```
4. `double`是联合体中占用空间最多的成员。
5. a = (b = 4)+ (c = 3); //res: a == 7。
6. 逗号表达式以最后一个表达式的值为准。
7. 有关宏的复合表达式计算。
8. 构造函数不能是虚函数。
9. 


