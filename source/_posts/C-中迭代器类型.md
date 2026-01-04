title: C++中迭代器类型与使用
author: weihehe
date: 2024-07-01 10:33:14
tags:
---
## 一、迭代器

迭代器本质上是一个**行为类似指针的对象**，支持：

* 解引用：`*it`
* 移动：`++it`
* 比较：`it != end`

```cpp
std::vector<int> v = {1, 2, 3};

for (auto it = v.begin(); it != v.end(); ++it)
{
    std::cout << *it << std::endl;
}
```
`it` 就是一个迭代器。

---

## 二、C++ 中的五种迭代器类型



| 类型                         | 能力         | 典型容器                       |
| -------------------------- | ---------- | -------------------------- |
| **Input Iterator**         | 只读、单向、一次遍历 | `istream_iterator`         |
| **Output Iterator**        | 只写、单向      | `ostream_iterator`         |
| **Forward Iterator**       | 读写、单向、多次遍历 | `forward_list`             |
| **Bidirectional Iterator** | 可前进、可后退    | `list`, `map`, `set`       |
| **Random Access Iterator** | 支持下标、算术运算  | `vector`, `deque`, `array` |

---

### Input Iterator（输入迭代器）

* 只能向前
* 只能读
* 常用于流

```cpp
std::istream_iterator<int> it(std::cin);
std::istream_iterator<int> end;

while (it != end)
{
    std::cout << *it << std::endl;
    ++it;
}
```

---

### Output Iterator（输出迭代器）

* 只能写
* 不能读

```cpp
std::ostream_iterator<int> out(std::cout, ", ");
*out = 10;
```

---

### Forward Iterator（前向迭代器）

* 读写
* 单向
* 可多次遍历

```cpp
std::forward_list<int> fl = {1, 2, 3};
for (auto it = fl.begin(); it != fl.end(); ++it)
{
    *it += 10;
}
```

---

### Bidirectional Iterator（双向迭代器）

* 支持 `++` 和 `--`

```cpp
std::list<int> lst = {1, 2, 3};
auto it = lst.end();
--it;
std::cout << *it; // 3
```

- `map / set` 也是双向迭代器（**不能随机访问**）

---

### Random Access Iterator（随机访问迭代器）

能力最强，支持很多操作

* `it + n`
* `it[n]`
* `<`, `>`

```cpp
std::vector<int> v = {10, 20, 30};
auto it = v.begin();
std::cout << it[1];      // 20
std::cout << *(it + 2);  // 30
```
## 

### const_iterator（只读迭代器）

```cpp
for (std::vector<int>::const_iterator it = v.cbegin(); it != v.cend(); ++it)
{
    // *it = 10; 
}
```

### reverse_iterator（反向迭代器）

```cpp
std::vector<int> v = {1, 2, 3};

for (auto it = v.rbegin(); it != v.rend(); ++it)
{
    std::cout << *it;
}
```

- 注意：

* `rbegin()` 指向最后一个元素
* `rend()` 指向第一个元素前

---

## 三、迭代器失效

### vector 中会导致失效的操作：

* `push_back()`（可能扩容）
* `insert()`
* `erase()`

- 错误示例：

```cpp
for (auto it = v.begin(); it != v.end(); ++it)
{
    if (*it == 2)
        v.erase(it); // 迭代器失效
}
```

- 正确写法：

```cpp
for (auto it = v.begin(); it != v.end(); )
{
    if (*it == 2)
        it = v.erase(it);
    else
        ++it;
}
```

## 四、迭代器与 STL 算法

STL 算法**不关心容器，只关心迭代器**。

```cpp
std::vector<int> v = {1, 2, 3};

std::find(v.begin(), v.end(), 2);
std::sort(v.begin(), v.end());  // 需要随机访问迭代器
```

## 五、以链表代码为例，实现双向迭代器

```cpp
#include <iostream>

template<typename T>
struct Node {
    T data;
    Node* prev;
    Node* next;
    Node(const T& val) : data(val), prev(nullptr), next(nullptr) {}
};

template<typename T>
class ListIterator {
public:
    ListIterator(Node<T>* node) : node(node) {}

    T& operator*() const { return node->data; }  // 建议加 const

    ListIterator& operator++() {
        node = node->next;
        return *this;
    }

    ListIterator operator++(int) {
        ListIterator temp = *this;
        ++(*this);
        return temp;
    }
	//支持迭代器for循环遍历
    bool operator==(const ListIterator& other) const {
        return node == other.node;
    }

    bool operator!=(const ListIterator& other) const {
        return !(*this == other);
    }

private:
    Node<T>* node;
};

template<typename T>
class List {
public:
    using iterator = ListIterator<T>;

    List() : head(nullptr), tail(nullptr) {}

    void push_back(const T& value) {
        Node<T>* newNode = new Node<T>(value);
        if (!tail) {
            head = tail = newNode;
        } else {
            tail->next = newNode;
            newNode->prev = tail;
            tail = newNode;
        }
    }
	// 支持范围for
    iterator begin() { return iterator(head); }
    iterator end() { return iterator(nullptr); }

    // 可选：析构函数（简化，实际需 delete 所有节点）
    ~List() {
        while (head) {
            Node<T>* temp = head;
            head = head->next;
            delete temp;
        }
    }

private:
    Node<T>* head;
    Node<T>* tail;
};

// 测试
int main() {
    List<int> lst;
    lst.push_back(1);
    lst.push_back(2);
    lst.push_back(3);

    // 范围 for 循环 
    for (const auto& x : lst) {
        std::cout << x << " ";
    }
    // 输出: 1 2 3

    return 0;
}
```

