---
title: Google C++风格指南(七)——命名规范
date: 2021-03-25 21:07:16
tags:
- C++编码规范
categories:
- C++
---

# 通用命名规则
描述性命名，如果你也曾被前人项目中的缩写整懵比那就少用缩写，如果实在需要，请在声明位置加注释。

# 文件命名
全部小写，下划线`_`连接。不要与`/usr/include`下的文件重名。

内联函数放在`.h`文件中。

# 类型命名
类、结构体、类型定义、枚举、类型模板参数。每个单词首字母大写，不应包含下划线。

# 变量命名
变量与数据成员一律小写，单词之间用下划线连接，类的成员变量用下划线结尾，结构体变量与普通变量一致，无需加下划线。

# 常量命名
声明为`constexpr`和`const`的变量，或在程序运行i期间其值始终保持不变的以`k`开头，大小写混合。

# 函数命名
常规函数大小写混合，取值或设值要求与变量名匹配，如`set_num()`，`get_num()`。

对于函数名中出现的单词缩写倾向于全部用大写`StartRPC()`。

# 命名空间命名
命名空间以小写字母命名，最高级命名空间取决于项目名称或团队名，避免嵌套命名空间与上层命名空间之间存在冲突。

命名空间中的代码，应当存放于与命名空间的名字匹配的文件夹或其子文件夹中。

不要使用缩写。

# 枚举命名
枚举命名与常量或宏保持一致`kEnumName`、`ENUM_NAME`。优先采用常量命名方式。

# 宏命名
参见[不要用宏]()，如果不得不适用，全部大写，下划线连接。


----


# 注释风格
项目统一风格`//`或`/* */`

写好`TODO`、`FIX`、`DEPRECATED`

# 我是不可能写注释的，告辞

