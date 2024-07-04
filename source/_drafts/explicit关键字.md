title: explicit关键字
author: weihehe
date: 2024-07-04 20:37:47
tags:
---

### 构造函数的隐式类型转换


```c++
...
My_class tmp_1(1);//构造函数
My_class tmp_2 = 2;//发生了类型转换
/*tmp2中，先用2构造了一个My_class的临时对象,临时对象再拷贝构造tmp2。
*/
...
```
