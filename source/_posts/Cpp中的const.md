title: C++中的const和临时变量
author: weihehe
tags:
  - Const
categories:
  - C++
date: 2024-07-01 15:39:00
---
临时变量，实例化
<!-- more -->
在CPP中，隐式类型转换通常会产生一个临时变量，例如

```C++
	int i = 0;
   //double& f = i; error!
   const double& f = i;
	/*由于产生了临时变量,而临时变量具有常量性，保证传递的值不发生改变，所以要加上const*/   
    
```
**在上述代码过程中，产生了const double类型的临时变量，并且f是它的别名，因此这个临时变量还被实例化了**

#### 匿名对象和const
![匿名对象](/images/Anonymous_object.png)

#### 各种位置上的const

![const](/images/const.png)
第一个const代表返回的const引用，调用方无法通过返回的引用修改字符值，后者的第二个const 修饰的是*this（被隐藏起来的参数），代表是一个常量成员函数 常量成员函数在执行期间不去改变原对象的数据。

#### 权限可以缩小，但是不能放大

![下面的Date方法的权限被放大了，所以是错误的](/images/limited.png)

*正确的写法是将 void print() 改为 void print() **const** *。并且因为权限可以缩小，所以d1也可以正确调用print。

**只要是成员函数且函数内部不修改成员变量，都应该加上const，这样const对象和普通对象都可以调用**。