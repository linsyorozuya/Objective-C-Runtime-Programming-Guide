---
description: 内省：introspect
---

# 与运行时交互

Objective-C 程序在三个不同的级别与运行时系统交互：通过 Objective-C 源代码; 通过`NSObject`Foundation 框架类中定义的方法; 并通过直接调用运行时函数。

### Objective-C源代码

在大多数情况下，运行时系统会在后台自动运行。您只需编写和编译 Objective-C 源代码即可使用它。

编译包含 Objective-C 类和方法的代码时，编译器会创建实现该语言动态特性的数据结构和函数调用。数据结构捕获在类和类别定义以及协议声明中找到的信息; 它们包括在 [_Objective-C编程语言_](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html#//apple_ref/doc/uid/TP30001163) __中 [定义类](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Chapters/ocDefiningClasses.html#//apple_ref/doc/uid/TP30001163-CH12) 和 [协议](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Chapters/ocProtocols.html#//apple_ref/doc/uid/TP30001163-CH15) 中讨论的类和协议对象，以及方法选择器，实例变量模板和从源代码中提取的其他信息。主要运行时函数是发送消息的函数，如 [Messaging中所述](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html#//apple_ref/doc/uid/TP40008048-CH104-SW1) 。它由源代码消息表达式调用。

### NSObject方法

Cocoa中的大多数对象都是类的子`NSObject`类，因此大多数对象都继承它定义的方法。（值得注意的例外是`NSProxy`类; 有关更多信息，请参阅 [消息转发](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-SW1)。）因此，其方法的建立行为是每个实例和每个类对象都固有的。但是，在少数情况下，`NSObject`该类仅定义了应该如何完成某事的模板; 它本身并不提供所有必要的代码。

例如，`NSObject`该类定义一个`description`实例方法，该方法返回描述该类内容的字符串。这主要用于调试--GDB `print-object`命令打印从此方法返回的字符串。`NSObject`这个方法的实现不知道该类包含什么，因此它返回一个字符串，其中包含该对象的名称和地址。子类`NSObject`可以实现此方法以返回更多详细信息。例如，Foundation类`NSArray`返回其包含的对象的描述列表。

一些`NSObject`方法只是查询运行时系统的信息。这些方法允许对象执行内省。这种方法的例子是`class` 方法，要求对象识别其类; `isKindOfClass:` 和 `isMemberOfClass:`，测试对象在继承层次结构中的位置; `respondsToSelector:`，表示对象是否可以接受特定消息; `conformsToProtocol:`，表示对象是否声称实现特定协议中定义的方法; 和`methodForSelector:`，它提供方法实现的地址。像这样的方法使对象能够对自己进行内省。

### 运行时功能

运行时系统是一个动态共享库，其公共接口由位于目录中的头文件中的一组函数和数据结构组成`/usr/include/objc`。其中许多函数允许您使用plain C来复制编写Objective-C代码时编译器所执行的操作。其他形成了通过`NSObject`类方法导出的功能的基础。这些函数可以开发运行时系统的其他接口，并生成增强开发环境的工具; 在Objective-C中编程时不需要它们。但是，在编写Objective-C程序时，一些运行时函数有时可能会有用。所有这些功能都记录在[_Objective-C运行时参考中_](https://developer.apple.com/documentation/objectivec/objective_c_runtime)。

