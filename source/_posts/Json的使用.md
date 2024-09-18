title: Json的简单使用
author: weihehe
date: 2024-09-14 22:40:11
tags:
---
Json 原始字符串字面量
<!--more-->

## 概念

- JSON (JavaScript Object Notation) 是一种轻量级的数据交换格式，主要用于在服务器和客户端之间传输数据。
	- `Json::Value` 是 JsonCpp 库中的一个类，`Json::Value` 可以表示 JSON 中的各种数据类型，包括对象（object）、数组（array）、字符串（string）、数字（int、float）、布尔值（bool）。

### Json对象 —— 其实就是一种将数据结构化表示的一种规范

- JSON对象: 用 {} 包裹的一组键值对，键是字符串，值可以是任何合法的 JSON 数据类型。

- **默认构造函数**：创建一个空的 `Json::Value` 对象，类型为 null。
  
  ```cpp
  Json::Value value;
  ```
  
- **带类型的构造函数**：可以直接构造特定类型的 `Json::Value` 对象。
  
  ```cpp
  Json::Value intValue(123);  
  Json::Value stringValue("hello");  
  Json::Value booleanValue(true);
  ```
  

### 类型检查

- **isNull**：检查是否为 null。
  
  ```cpp
  bool isNull() const;
  ```
  
- **isBool**：检查是否为布尔值。
  
  ```cpp
  bool isBool() const;
  ```
  
- **isInt** / **isInt64**：检查是否为整数。
  
  ```cpp
  bool isInt() const;  bool isInt64() const;
  ```
  
- **isDouble**：检查是否为浮点数。
  
  ```cpp
  bool isDouble() const;
  ```
  
- **isString**：检查是否为字符串。
  
  ```cpp
  bool isString() const;
  ```
  
- **isArray**：检查是否为数组。
  
  ```cpp
  bool isArray() const;
  ```
  
- **isObject**：检查是否为对象。
  
  ```cpp
  bool isObject() const;
  ```
  

### 获取值

- **asBool**：获取布尔值。
  
  ```cpp
  bool asBool() const;
  ```
  
- **asInt** / **asInt64**：获取整数。
  
  ```cpp
  int asInt() const;  Json::Int64 asInt64() const;
  ```
  
- **asDouble**：获取浮点数。
  
  ```cpp
  double asDouble() const;
  ```
  
- **asString**：获取字符串。
  
  ```cpp
  std::string asString() const;
  ```
  

### 设置值

- **operator[]**：用于访问和设置对象成员或**数组元素**。
  
  ```cpp
  Json::Value obj;  
  obj["key"] = "value";  
  // 设置对象成员    
  Json::Value arr;  arr[0] = 1;  // 设置数组元素
  ```
  
- **set**：直接设置值，可以指定数据类型。
  
  ```cpp
  void set(const std::string& key, const Json::Value& value);  
  // 设置对象成员  
  void set(int index, const Json::Value& value);  
  // 设置数组元素
  ```
  

### 数组操作

- **append**：向数组添加元素。
  
  ```cpp
  void append(const Json::Value& value);
  ```
  
- **size**：获取数组大小。
  
  ```cpp
  unsigned int size() const;
  ```
  

### 对象操作

- **isMember**：检查对象是否有特定键。
  
  ```cpp
  bool isMember(const std::string& key) const;
  ```
  
- **removeMember**：移除对象成员。
  
  ```cpp
  void removeMember(const std::string& key);
  ```
  
- **getMemberNames**：获取对象所有键。
  
  ```cpp
  std::vector<std::string> getMemberNames() const;
  ```
  

### 序列化与反序列化 ——内部使用了日志宏

- **write**：将 `Json::Value` 对象序列化为 JSON 格式的字符串。
  
  ```cpp
  static bool serialize(const Json::Value &val, std::string &body)
  {
      std::stringstream ss;
      // 先实例化一个工厂类对象
      Json::StreamWriterBuilder swb;
      // 通过工厂类对象来生产派生类对象
      std::unique_ptr<Json::StreamWriter> sw(swb.newStreamWriter());
      int ret = sw->write(val, &ss);
      if (ret != 0)
      {
          ELOG("json serialize failed!");
          return false;
      }
      body = ss.str();
      return true;
  }
  ```
  
- **parse**：从 JSON 格式的字符串解析出 `Json::Value` 对象。
  

```cpp
static bool unserilize(const std::string &body, Json::Value &val)
{
    // 实例化工厂对象
    Json::CharReaderBuilder crb;
    // 生产CharReader对象
    std::unique_ptr<Json::CharReader> cr(crb.newCharReader());
    std::string errs;
    bool ret1 = cr->parse(body.c_str(), body.c_str() + body.size(), &val, &errs);
    if (ret1 == false)
    {
        ELOG("UnSerialize failed!\n:%s", errs.c_str());
        return false;
    }
    return true;
}
```

### 原始字符串字面量

- 一般格式：`R("")`。这两个括号之间的内容会被原样复制到字符串中，包括反斜杠、双引号等特殊字符，而不需要进行转义。

- 使用：`std::string str = R"({"姓名":"小黑","年龄":19,"成绩":[32,45,56]})";`就创建了一个`Json`格式的字符串。

## 字段宏（`field macros`）

- 提高可维护性：当数据名称发生变化时，只需更新宏定义或相关部分，不必在多个地方修改，从而更容易维护代码。

## 需要注意的点

```cpp
//其中KEY_HOST_IP是KEY_HOST的子元素，因此要使用二维数组的方式来访问
_body[KEY_HOST][KEY_HOST_IP].isNull() == true 
```