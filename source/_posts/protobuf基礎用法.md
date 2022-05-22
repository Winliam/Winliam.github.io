---
title: protobuf基础用法
categories:
  - - 工作技能
    - C/C++笔记
date: 2021-04-01 12:18:45
tags:
  - protobuf
---
## 前言
自动驾驶框架常用protobuf作为节点之间通讯的工具，出现频率很高，了解一下基本情况。

我们知道，**数据只要出内存就要进行序列化，进内存就要进行反序列化**。

这是因为，所有数据在内存中存放的形式都是一串0/1。比如，我们定义一个结构体，然后对其实例化，我们就在内存中的某个位置写入了这串0/1序列。当我们想把这一个结构体所表示的“信息”给到另外一个用户时，应该怎么做呢？

我们可以把这一串0/1序列从内存中写到硬盘上，形成一个文件，然后把这个文件发送给其他用户。由内存中的0/1序列（或者说一个结构体实例）得到这个文件的过程，就是序列化过程。

用户在拿到这一文件后想使用其中包含的信息，就必须先将其恢复为结构体，才能方便地进行操作。由二进制序列（或者说拿到的这个文件）得到结构体实例的过程，就是反序列化过程。

那么protobuf做了什么呢？首先，它允许我们以最简的方式定义待传递的信息结构（`.proto文件`），然后使用`protoc`程序处理`.proto`文件，自动生成C++结构体，并配套一系列操作这个结构体的方法（API）。

所以说，protobuf的使用围绕三个核心要素:
- `.proto`文件
- compiler
- API

## `.proto`文件
```protobuf
// protobuf有2和3两个版本，语法不同，所以要声明版本
syntax = "proto3";
// package名称经过compile之后成为生成类所在的namespace
package tutorial;
import "google/protobuf/timestamp.proto";

// 一个message生成一个类
message Person {
  string name = 1;
  int32 id = 2; 
  string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  // message元素的四个组成部分, modifier type name = uniq_tag，modifier默认为optional
  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

message AddressBook {
  repeated Person people = 1;
}
```
## compile
```shell
# 编译.proto文件，生成类的头文件和源文件
protoc protoc --cpp_out=. addressbook.proto

#在应用中使用生成类
c++ -std=c++11 add_person.cc addressbook.pb.cc -o add_person_cpp `pkg-config --cflags --libs protobuf`
```

## API
### 序列化API
  - `bool SerializeToString(string* output) const;`
  - `bool SerializeToOstream(ostream* output) const;`
一个生成类的实例调用这两种方法，将会把自身的内容以二进制串的形式写入string或ostream中，如
```cpp
fstream output(argv[1], ios::out | ios::trunc | ios::binary);
if (!address_book.SerializeToOstream(&output)) {
    cerr << "Failed to write address book." << endl;
    return -1;
}
```

### 反序列化API
  - `bool ParseFromString(const string& data);`
  - `bool ParseFromIstream(istream* input);`
从一个istream或string读取一个二进制串，然后这个生成类的实例中的内容便得到了填充。如，
```cpp
tutorial::AddressBook address_book;
fstream input(argv[1], ios::in | ios::binary);
if (!address_book.ParseFromIstream(&input)) {
    cerr << "Failed to parse address book." << endl;
    return -1;
}
    }
```

### 元素获取与操作API
```cpp
// id
inline bool has_id() const;
inline void clear_id();
inline void set_id(int32_t value);
```

### 通用API
- `bool IsInitialized() const;`: checks if all the required fields have been set.
- `void Clear();`: clears all the elements back to the empty state.
- `void CopyFrom(const Person& from);`: overwrites the message with the given message's values.




## 参考链接
1. https://developers.google.com/protocol-buffers/docs/cpptutorial
2. https://stackoverflow.com/questions/26826421/protocol-buffers-unique-numbered-tag-clarification
