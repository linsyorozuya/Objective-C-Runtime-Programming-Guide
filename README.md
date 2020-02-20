# 介绍

**重要提示：**此文档不再更新。有关Apple SDK的最新信息，请访问[文档网站](https://developer.apple.com/documentation)。

Objective-C语言从编译时间和链接时间到运行时推迟了尽可能多的决策。只要有可能，它就会动态地完成任务。这意味着该语言不仅需要编译器，还需要运行时系统来执行编译代码。运行时系统扮演着 Objective-C 语言的一种操作系统， 运行时是让改语言运作的原因。

本文档介绍了`NSObject`该类以及Objective-C程序如何与运行时系统交互。特别是，它检查了在运行时动态加载新类并将消息转发给其他对象的范例。它还提供有关在程序运行时如何查找有关对象的信息的信息。

您应该阅读本文档以了解Objective-C运行时系统的工作原理以及如何利用它。但是，通常，您应该没有理由需要了解和理解这些材料来编写Cocoa应用程序。

### 本文件的组织

本文档包括以下章节：

* [运行时版本和平台](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtVersionsPlatforms.html#//apple_ref/doc/uid/TP40008048-CH106-SW1)
* [与运行时交互](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtInteracting.html#//apple_ref/doc/uid/TP40008048-CH103-SW1)
* [消息](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html#//apple_ref/doc/uid/TP40008048-CH104-SW1)
* [动态方法解析](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtDynamicResolution.html#//apple_ref/doc/uid/TP40008048-CH102-SW1)
* [消息转发](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-SW1)
* [键入编码](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)
* [声明的属性](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html#//apple_ref/doc/uid/TP40008048-CH101-SW1)

### 也可以看看

[_Objective-C运行时参考_](https://developer.apple.com/documentation/objectivec/objective_c_runtime)描述了Objective-C运行时支持库的数据结构和功能。您的程序可以使用这些接口与Objective-C运行时系统进行交互。例如，您可以添加类或方法，或获取已加载类的所有类定义的列表。

[_使用Objective-C编程_](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011210)描述了Objective-C语言。

[_Objective-C发行说明_](https://developer.apple.com/library/archive/releasenotes/Cocoa/RN-ObjectiveC/index.html#//apple_ref/doc/uid/TP40004309)描述了最新版OS X中Objective-C运行时的一些更改。  
  


