---
layout: post
title: "CPP get list of files in directory"
date: "2021-11-15 14: 19: 00"
tag: cpp filesystem
---
- [CPP 获取目录下的文件](#org8875930)
  - [filesystem 标准库介绍](#orgce9af53)
  - [实现代码](#org2a5a196)


<a id="org8875930"></a>

# CPP 获取目录下的文件

在 Python 中获取目录下的所有文件可以用 `os.listdir(path)` 实现，在CPP中有没有类似的方法呢？答案是肯定的。


<a id="orgce9af53"></a>

## filesystem 标准库介绍

**filesystem** 库提供了对文件系统及其组件（例如路径、常规文件和目录）执行操作的工具。 文件系统库最初开发为 boost.filesystem，作为技术规范 ISO/IEC TS 18822:2015 发布，最终从 C++17 合并到 ISO C++。 boost 实现目前在比 C++17 库更多的编译器和平台上可用。 所用的命名空间是 `std::filesystem` 。我们用到了库里提供的 **directory_iterator** 类 (C++17), 它提供目录内容的迭代器。


<a id="org2a5a196"></a>

## 实现代码

```cpp
void listDir(const std::string &path, std::vector<std::string> &files,
	     std::string suffix = "") {

  for (auto &entry : std::filesystem::directory_iterator(path)) {
    std::string fileName = entry.path();

    if (suffix.empty()) {
      files.push_back(fileName);
    } else {
      int pos = fileName.rfind(suffix);
      if (pos != std::string::npos &&
	  fileName.compare(pos, suffix.size() - 1, suffix)) {
	files.push_back(fileName);
      }
    }
  }
}
```

-   结果

```shell
All files: 
 |- /home/larry/图片
 |- /home/larry/.bashrc-anaconda3.bak
 |- /home/larry/文档
 |- /home/larry/音乐
 |- /home/larry/桌面
 |- /home/larry/.bashrc.2021-10-01-17-03.bak
 |- /home/larry/下载

File names ending in .bak
 |- /home/larry/.bashrc.2021-10-01-17-03.bak
 |- /home/larry/.bashrc-anaconda3.bak
```

-   完整代码 [在这里](https://github.com/combofish/chips-get-cpp/blob/main/Get_List_of_Files_in_Directory/listdir.hpp)
