# 动态方法解析

本章介绍如何提供动态方法的实现。

### 动态方法解析

在某些情况下，您可能希望动态地提供方法的实现。例如，Objective-C声明的属性功能（参见[_Objective-C编程语言_](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html#//apple_ref/doc/uid/TP30001163)中[_的_](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html#//apple_ref/doc/uid/TP30001163)[声明属性](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Chapters/ocProperties.html#//apple_ref/doc/uid/TP30001163-CH17)）包含指令：`@dynamic`

```swift
@dynamic propertyName;
```

它告诉编译器将动态提供与属性关联的方法。

您可以实现这些方法[`resolveInstanceMethod:`](https://developer.apple.com/documentation/objectivec/nsobject/1418500-resolveinstancemethod)，[`resolveClassMethod:`](https://developer.apple.com/documentation/objectivec/nsobject/1418889-resolveclassmethod)并分别为实例和类方法动态提供给定选择器的实现。

Objective-C方法只是一个C函数，至少需要两个参数 - `self`和`_cmd`。您可以使用该函数将函数作为方法添加到类中[`class_addMethod`](https://developer.apple.com/documentation/objectivec/1418901-class_addmethod)。因此，给出以下功能：

```swift
void dynamicMethodIMP(id self, SEL _cmd) {
    // implementation ....
}
```

您可以`resolveThisMethodDynamically`使用以下方法将其作为方法（调用）动态添加到类中`resolveInstanceMethod:`：

```swift
@implementation MyClass
+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if (aSEL == @selector(resolveThisMethodDynamically)) {
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
          return YES;
    }
    return [super resolveInstanceMethod:aSEL];
}
@end
```

转发方法（如[消息转发中所述](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-SW1)）和动态方法解析在很大程度上是正交的。类有机会在转发机制启动之前动态解析方法。如果[`respondsToSelector:`](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Protocols/NSObject/Description.html#//apple_ref/occ/intfm/NSObject/respondsToSelector:)或[`instancesRespondToSelector`](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Classes/NSObject/Description.html#//apple_ref/occ/clm/NSObject/instancesRespondToSelector:)被调用，动态方法解析器有机会首先为选择器提供`IMP`。如果您实现[`resolveInstanceMethod:`](https://developer.apple.com/documentation/objectivec/nsobject/1418500-resolveinstancemethod)但希望通过转发机制实际转发特定选择器，则返回`NO`这些选择器。

### 动态加载

Objective-C程序可以在它在运行时加载和链接新类 和类别。新代码被合并到程序中，并且与开始时加载的类和类别完全相同。

动态加载可用于执行许多不同的操作。例如，系统首选项应用程序中的各种模块是动态加载的。

在Cocoa环境中，动态加载通常用于允许自定义应用程序。其他人可以编写程序在运行时加载的模块 - 就像Interface Builder加载自定义调色板并且OS X系统首选项应用程序加载自定义首选项模块一样。可加载模块扩展了应用程序的功能。他们以您允许的方式为其做出贡献，但无法预测或定义自己。您提供框架，但其他人提供代码。

虽然有一个运行时函数可以在Mach-O文件中动态加载Objective-C模块（`objc_loadModules`，定义于`objc/objc-load.h`），可可的`NSBundle`class为动态加载提供了一个非常方便的接口 - 一个面向对象并与相关服务集成的接口。有关[`NSBundle`](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Classes/NSBundle/Description.html#//apple_ref/occ/cl/NSBundle)类`NSBundle`及其用法的信息，请参阅Foundation框架参考中的类规范。请参见_OS X ABI Mach-O文件格式参考_ 有关Mach-O文件的信息。

