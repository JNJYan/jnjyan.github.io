---
title: Google C++风格指南
date: 2021-03-24 17:53:57
tags: C++编码规范
categories:
- C++
---

## 头文件

### self-contained头文件

头文件需要自给自足，头文件本身应当是能编译的（个人理解时在没有源文件`.cxx`的情况也能编译通过）。

头文件应当用`#define`保护，并包含其所需要的所有其他头文件。

模板和内联函数的定义和声明应当处于同一文件内。

### define保护

所有头文件都应该使用`#define`防止文件被多重包含，并且为了保证唯一性，宏命名应当基于所在项目源码的全路径，格式应当为`<PROJECT>_<PATH>_<FILE>_H_`。

### 引入所有使用的头文件

不要依赖于`include`传递，如果`foo.cxx`文件使用了`bar.h`中的符号，就应当在`foo.cxx`中引入`bar.h`，即使在`bar.h`已经引入了。

也就是说，尽量在源文件中引入库，而不是在对应的头文件中引入，目的是为了减少引入依赖。例如当`A.cxx`依赖于`B.h`时，对`B.h`的任何修改都会导致`A.cxx`的重新构建，而此时`B.cxx`依赖于`C.h`，那么我们现在有两个选择，一是在`B.cxx`中引入`C.h`，二是在`B.h`中引入`C.h`。若选择第二种，我们对`C.h`的任何修改都会导致`A.cxx`的重新构建，若选择第一种，则不会触发`A.cxx`的重新构建。

### 前置声明

前置声明是指类、函数和模板不带有定义的纯粹声明，应当避免使用前置声明，将声明放在头文件中，然后`#include`。

- 优点
  - 前置声明能够节省编译时间。
  - 前置声明能节省不必要的重新编译时间，使用`#include`会使得代码由于头文件中无关改动而被重新编译。
- 缺点
  - 前置声明隐藏依赖关系，头文件改动时，源文件会跳过必要的重新编译过程。
  - 前置声明可能被库的后续更改所破坏。
  - 前置声明来自`std::`中的符号时，行为未定义。
  - 前置声明甚至会改变代码含义。
  - 重构代码更为复杂和困难。

### 内联函数

- 经验：
  - 只有函数不多于10行时才将其定义为内联函数
  - 不要内联包含循环和`switch`的函数
  - 不要内联递归函数
  - 即使声明为内联函数，编译器也不一定会接受
  - 类声明内定义的函数默认为内联函数

### include的路径和顺序

避免使用`.`和`..`，按照项目源码路径完整`include`。

- 内联顺序：
  1. 同名头文件
  2. C系统文件
  3. C++系统文件
  4. 其他库`.h`
  5. 本项目`.h`

这种优先顺序保证了当`.h`文件遗漏某些库时，源文件的构建会立刻终止。

不同类型的头文件空格分隔。

例外情况，条件编译绝对是否引入库，如不同平台。

## 作用域

### 命名空间

C++中的全局函数和全局变量的作用域是整个项目，为了限制其作用域，一种做法是使用`static`关键字修饰，另一种是使用命名空间。

在不需要外部访问的情况下，C++标准提倡使用匿名命名空间，禁止使用`using`引入命名空间，禁止使用内联命名空间。

```c++
namespace X{
    inline namespace Y{
        void foo();
    }
}

// 别名
namespace baz = ::foo::bar::baz;
```

在内联命名空间中，`X::Y::foo()`与`X::foo()`等价，其目的主要是用来兼容跨版本API。

命名空间使用策略：

- 遵循命名空间命名规范
- 命名空间结束时，使用注释
- 命名空间应当在头文件、gflag声明与定义、其他空间的类前置声明之后
- 禁止在`std`内声明任何东西，属于未定义行为，导致不可移植
- 禁止在头文件中使用命名空间别名，除非限制在内部命名空间中使用
- 禁止使用`using`引入命名空间
- 禁止使用内联命名空间

### 匿名命名空间与静态变量

在`cxx`中定义不需要被外部引用的符号时，可以放在匿名命名空间或声明为`static`，但不要在头文件中这样做。

### 非成员函数、静态成员函数、全局函数

使用静态成员或命名空间内的非成员函数，不要使用裸的全局函数，不要用类的静态方法模拟命名空间效果，类的静态方法应当与类的实例或静态数据密切相关。

非成员函数置于命名空间避免污染全局作用域。

### 局部变量

将函数变量尽可能放在最小作用域内，在变量声明时进行初始化。

除非变量是一个对象，每次进入作用域都需要调用构造函数，退出作用域调用析构函数。

### 静态和全局变量

静态生存周期的对象，包括全局变量、静态变量、静态类成员、函数静态变量，必须是原生数据类型(POD, Plain Old Data)。

在多编译单元中，静态变量的构造、析构和初始化顺序在C++中只有部分是明确的，甚至随着构建发生变化，导致难以发现的bug，禁止使用类的静态存储周期变量。

在同一编译单元中顺序是明确的，静态初始化优于动态初始化、初始化顺序按照声明顺序进行，逆序销毁，不同编译单元内，属于未明确行为。

多线程时，静态生存周期不要使用非POD的对象以及STL容器。

## 类

### 构造函数

不要在构造函数中调用虚函数，也不要在无法报错时进行可能失败的初始化。

构造函数内调用自身的虚函数并不会重定向到子类的虚函数实现。

如果执行失败，可能会得到一个初始化失败的对象，这个对象可能无法进入到正常状态。

如果对象需要进行初始化，通过定义`Init()`方法或工厂方法进行创建并初始化。

### 隐式类型转换

一脸懵比

### 可拷贝类型和可移动类型

如果不需要，禁用隐式生成的拷贝和移动构造函数。

### 结构体/类

仅当只有数据成员和重载运算符时使用`struct`，其他一概使用`class`。

## TODO

## 函数

### 参数顺序

输入参数在前，输出参数在后。

### 简短函数

优先编写简短函数，若行数超过四十行考虑分离。

### 引用参数

引用必须用`const`，对变量进行修改传指针。除非特殊要求，如`swap()`。

### 函数重载

函数重载尽量能够简单明了，尽量不要改变相同数量的参数类型来进行重载。可以考虑在函数名中加入类型信息。

### 缺省参数

只允许在非虚函数中使用缺省参数，尽可能使用函数重载而非缺省参数。

优点：

- 降低代码量，降低代码修改时的工作量，函数重载需要修改多个函数。

缺点：

- 虚函数调用的缺省参数取决于目标对象的静态类型，无法保证给定函数的所有重载声明的都是同样的缺省参数。
- 缺省参数会干扰函数指针，导致函数签名与调用点签名不一致。
- 缺省参数每次调用都需要重新求值,导致生成的代码膨胀。

### 函数返回类型后置语法

只有常规写法不便于书写或阅读时才使用返回类型后置语法。

```c++
int foo(int x);
auto foo(int x) -> int;
```

优点：

- 后置返回类型是显示指定Lambda表达式返回值类型的唯一方式，通常情况下编译器能够自动推导出Lambda表达式的返回类型，但并不是所有情况。

缺点：

- 陌生，与原始代码看起来不协调（黑人问号？）

## G式奇淫技巧

### 所有权与智能指针

C++11后时代C++程序员的常识题，不要使用`std::auto_ptr`，使用`std::unique_ptr`或`std::shared_ptr`，倾向于前者。

### Cpplint

风格检查[cpplint.py](https://github.com/google/styleguide/blob/gh-pages/cpplint/cpplint.py)

## 命名规范

### 通用命名规则

描述性命名，如果你也曾被前人项目中的缩写整懵比那就少用缩写，如果实在需要，请在声明位置加注释。

### 文件命名

全部小写，下划线`_`连接。不要与`/usr/include`下的文件重名。

内联函数放在`.h`文件中。

### 类型命名

类、结构体、类型定义、枚举、类型模板参数。每个单词首字母大写，不应包含下划线。

### 变量命名

变量与数据成员一律小写，单词之间用下划线连接，类的成员变量用下划线结尾，结构体变量与普通变量一致，无需加下划线。

### 常量命名

声明为`constexpr`和`const`的变量，或在程序运行i期间其值始终保持不变的以`k`开头，大小写混合。

### 函数命名

常规函数大小写混合，取值或设值要求与变量名匹配，如`set_num()`，`get_num()`。

对于函数名中出现的单词缩写倾向于全部用大写`StartRPC()`。

### 命名空间命名

命名空间以小写字母命名，最高级命名空间取决于项目名称或团队名，避免嵌套命名空间与上层命名空间之间存在冲突。

命名空间中的代码，应当存放于与命名空间的名字匹配的文件夹或其子文件夹中。

不要使用缩写。

### 枚举命名

枚举命名与常量或宏保持一致`kEnumName`、`ENUM_NAME`。优先采用常量命名方式。

## 宏命名

参见[不要用宏]()，如果不得不适用，全部大写，下划线连接。

### 注释风格

项目统一风格`//`或`/* */`

写好`TODO`、`FIX`、`DEPRECATED`

### 我是不可能写注释的，告辞
