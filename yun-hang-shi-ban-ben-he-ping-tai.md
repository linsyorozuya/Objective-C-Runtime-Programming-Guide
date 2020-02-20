---
description: 布局：layout
---

# 运行时版本和平台

在不同平台上有不同版本的Objective-C运行时。

### 遗产和现代版本

Objective-C运行时有两个版本 - “现代”和“传统”。现代版本随Objective-C 2.0一起推出，包含许多新功能。_Objective-C 1运行时参考中_描述了遗留版本的运行时的编程接口; [_Objective-C运行时参考中_](https://developer.apple.com/documentation/objectivec/objective_c_runtime) __描述了现代版本的运行时的编程接口。

最值得注意的新功能是现代运行时中的实例变量是“非脆弱的”：

* 在遗留运行时中，如果更改类中实例变量的布局，则必须重新编译从其继承的类。
* 在现代运行时，如果更改类中实例变量的布局，则不必重新编译从其继承的类。

此外，现代运行时支持声明属性的实例变量合成（请参阅[_Objective-C编程语言_](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html#//apple_ref/doc/uid/TP30001163)中[_的_](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html#//apple_ref/doc/uid/TP30001163)[声明属性](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Chapters/ocProperties.html#//apple_ref/doc/uid/TP30001163-CH17)）。

### 平台

OS X v10.5及更高版本上的iPhone应用程序和64位程序使用现代版本的运行时。

其他程序（OS X桌面上的32位程序）使用运行时的旧版本。

