---
title: Google C++风格指南(三)——类
date: 2021-03-25 14:02:10
tags:
- C++编码规范
categories:
- C++
---

# 构造函数
不要在构造函数中调用虚函数，也不要在无法报错时进行可能失败的初始化。

构造函数内调用自身的虚函数并不会重定向到子类的虚函数实现。

如果执行失败，可能会得到一个初始化失败的对象，这个对象可能无法进入到正常状态。

如果对象需要进行初始化，通过定义`Init()`方法或工厂方法进行创建并初始化。

# 隐式类型转换
一脸懵比

# 可拷贝类型和可移动类型
如果不需要，禁用隐式生成的拷贝和移动构造函数。

# 结构体/类
仅当只有数据成员和重载运算符时使用`struct`，其他一概使用`class`。

#TODO












