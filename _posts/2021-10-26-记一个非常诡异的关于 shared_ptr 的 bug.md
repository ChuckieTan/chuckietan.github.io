---
layout: post
title: 记一个非常诡异的关于 shared_ptr 的 bug
date: 2021-10-26
categories:
  - C++
tags:
  - C++
  - 智能指针
  - shared_ptr
keywords:
  - C++
  - 智能指针
  - shared_ptr
---

## 问题描述

今天写项目的时候遇见一个特别诡异的 bug，体现在在执行某条语句时，程序会莫名崩溃，并且给出的错误信息也非常难懂，只有一个`malloc(): invalid size (unsorted)`错误信息，从直观上看起来是 malloc 函数无法分配到内存，就想着应该是哪个动态分配内存的地方变量没获取到值，但是调试的时候才发现没这么简单。

<!--more-->

## 问题排查

调试的时候，发现程序崩溃的时候的调用栈最后竟然是一个`vector`，并且是在`push_back`的时候，心里面就隐隐感觉不对了，因为这个程序中的数据远远达不到内存超限的地步，而 vector 的内存是动态分配的，所以说基本上不可能获取不到足够的内存。

看源码时注意到另一个 vector 就不会崩溃，于是就增加几个类型的 vector，逐一试验，发现在基本数据类型中，`std::int32_t`，`std::int64_t`和他对应的无符号类型就不会导致程序崩溃，但是`std::int16_t`，`std::int8_t`，`bool`和`char`就会导致程序崩溃，分析到这里，看起来好像是大于等于 32 个字节的数据类型就不会崩溃。但是，我自定义的一个 struct 也会导致崩溃，而这个 struct 有 48 个字节，调试到这，感觉这个 bug 越来越诡异了。

按理来说，在 C++ 里面，普通的结构体如果没有虚函数的话和自带的数据类型是完全相同的，都是一个内存地址，对应着其大小的字节流，但是在这，不同大小的类型竟然有不同的反应。

于是就调试到 STL 的源码，发现最后一个调用的语句是`::operator new`，这个是一个按照字节分配内存的语句，语句把语句单独拿出来，放到崩溃语句的前面，发现程序的确会直接报`malloc(): invalid size (unsorted)`错误，但如果放在 main 函数最前面的话，却不会崩溃，最后反复定位，定位到最终会引起 bug 的地方。这个函数如下：

```cpp
storage::SQLBinaryData Pager::readRow(std::uint32_t pos) {
    if (pos <= getFileSize()) {
        std::uint32_t size;
        dataFile.seekg(pos, dataFile.beg);
        dataFile.read((char *) &size, sizeof(size));
        // data 里面是一个 new 出来的 char 数组 的 shared_ptr
        SQLBinaryData data(size);

        auto addr = data.data.get();
        dataFile.read(addr, size);

        // 这就是能造成崩溃的 ::operator new 语句
        auto test = ::operator new(1);
        return data;
    } else {
        spdlog::error("read file out of file size");
        return SQLBinaryData(0);
    }
}
```

## 解决方案

可以看出，这个 bug 大概率和`shared_ptr`有关，在网上查阅了很长时间资料，最后才知道在 C++17 之前，`shared_ptr` 并不支持动态数组，在析构的时候`shared_ptr`只会调用`delete`，而不是`delete[]`，如果要管理`new[]`构造出来的数组，需要在构造的时候传入自定义的 delete 删除器`std::default_delete`，要么就使用`unique_ptr`。

其实大部分情况下智能指针并不需要`shared_ptr`，用`unique_ptr`就够了，没有这么多要共享的东西。

还有一种比较简便的做法，就是直接用`vector`来管理动态数组，这就已经能满足很多 `new[]` 的情况了。一般情况下，写 C++ 的时候，还是得遵循能不用指针就不用指针的原则。
