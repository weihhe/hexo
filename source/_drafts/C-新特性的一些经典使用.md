title: C++新特性的一些经典使用
author: weihehe
date: 2025-12-31 11:21:57
tags:
---
## 保留那些内部包含`.png` 文件的子目录

```
auto input_metadata_dir_local8 = getSubDir(toStdString(configure.get_first_directory()), true);
		if (input_metadata_dir_local8.size() == 0) {
			u8log::error() << u8"输入路径中没有数据..." << u8log::endl;
			return 1;
		}
		for (auto iterator = input_metadata_dir_local8.begin(); iterator != input_metadata_dir_local8.end();)
		{
			try
			{
				std::cout<<*iterator;
				if (std::vector<std::string> files;
					(files = getAllFiles(*iterator,false),
					std::any_of(files.begin(),files.end(),[](const std::string& file)
					{
						return getFileSuffix(file) == "png";
					})))
					{
						++iterator;
					}
					else
					{
						iterator = input_metadata_dir_local8.erase(iterator);
					}
			}
			catch (const std::exception &ex)
			{
				std::cout << ex.what() << std::endl;
				++iterator;
			}
		}
```
## 判断是否为枚举，并打印枚举值

```cpp
#include <iostream>
template<typename T>
std::ostream& operator<<(
    typename std::enable_if<std::is_enum<T>::value,
        std::ostream>::type& stream, const T& e)
{
    return stream << static_cast<typename std::underlying_type<T>::type>(e);
}
```