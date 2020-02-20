# 消息转发

将消息发送到不处理该消息的对象是一个错误。但是，在宣布错误之前，运行时系统为接收对象提供了第二次处理消息的机会。

### 转发

如果您向不处理该消息的对象发送消息，则在宣布错误之前，运行时会向对象发送一条`forwardInvocation:`消息，其中包含`NSInvocation`object作为唯一参数 - 该`NSInvocation`对象封装了原始消息和随之传递的参数。

你可以实现一个 `forwardInvocation:`提供对消息的默认响应的方法，或以其他方式避免错误的方法。顾名思义，`forwardInvocation:`通常用于将消息转发给另一个对象。

要查看转发的范围和意图，请设想以下方案：首先，假设您正在设计一个可以响应叫`negotiate`消息的对象，并且您希望其响应包含另一种对象的响应。您可以通过将`negotiate`消息传递给您其他有实现`negotiate`的方法主体中的对象来轻松完成此操作。

更进一步，假设您希望对象对`negotiate`消息的响应完全是在另一个类中实现的响应。实现此目的的一种方法是让您的类从其他类继承该方法。但是，可能无法以这种方式安排事情。您的类和实现的类可能有充分的理由实现`negotiate`在继承层次结构的不同分支中。

即使你的类不能继承这个`negotiate`方法，你仍然可以通过实现一个简单地将消息传递给另一个类的实例的方法来“借用”它：

```swift
- (id)negotiate
{
    if ( [someOtherObject respondsTo:@selector(negotiate)] )
        return [someOtherObject negotiate];
    return self;
}
```

这种做事方式可能会有点麻烦，特别是如果有许多消息要将对象传递给另一个对象。您必须实现一种方法来涵盖您希望从其他类借用的每种方法。此外，在您编写代码时，您可能无法处理您可能想要转发的完整邮件集。该集合可能取决于运行时的事件，并且可能会在将来实现新方法和类时发生变化。

`forwardInvocation:`消息提供的第二次机会为这个问题提供了一个不那么特别的解决方案，而且是一个动态而非静态的解决方案。它的工作方式如下：当一个对象无法响应消息，因为它没有与消息中的选择器匹配的方法时，运行时系统通过向对象发送`forwardInvocation:`消息来通知该对象。每个对象都从`NSObject`类继承一个`forwardInvocation:`方法。但是，`NSObject`该方法的版本只是调用`doesNotRecognizeSelector:`。通过覆盖`NSObject`版本并实现自己的版本，您可以利用`forwardInvocation:`消息提供的机会将消息转发到其他对象。

要转发消息，所有`forwardInvocation:`方法需要做的是：

* 确定消息的去向，以及
* 用它的原始参数发送它。

可以使用以下`invokeWithTarget:`方法发送消息：

```swift
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    if ([someOtherObject respondsToSelector:
            [anInvocation selector]])
        [anInvocation invokeWithTarget:someOtherObject];
    else
        [super forwardInvocation:anInvocation];
}
```

转发的信息的返回值将返回给原始发件人。可以将所有类型的返回值传递给发送方，包括`id`s，结构和双精度浮点数。

`forwardInvocation:`方法可以作为一个未确认消息的分配中心，包裹出来到不同的接收器。或者它可以是转移站，将所有消息发送到同一目的地。它可以将一条消息翻译成另一条消息，或者只是“吞下”一些消息，这样就没有响应也没有错误。`forwardInvocation:`方法也可以合并几个消息到单个响应。`forwardInvocation:`能做什么取决于实施者。但是，它为转发链中的对象链接提供了机会，为程序设计开辟了可能性。

**注意：**`forwardInvocation:`只有当方法不调用标称接收器中的现有方法时， 该方法才会处理消息。例如，如果您希望对象将`negotiate`消息转发到另一个对象，则它不能拥有`negotiate`自己的方法。如果是，则消息永远不会到达`forwardInvocation:`。

有关转发和调用的更多信息，请参阅`NSInvocation`Foundation框架参考中的类规范。

### 转发和多重继承

转发模仿继承，并可用于向Objective-C程序提供多重继承的一些效果。[如图5-1](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-87317)所示，通过转发消息来响应消息的对象似乎借用或“继承”另一个类中定义的方法实现。**图5-1**   转发

![](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Art/forwarding.gif)

在此图示中，Warrior类`negotiate`的实例将消息转发给Diplomat类的实例。战士似乎会像外交官一样进行谈判。它似乎会对这个`negotiate`信息做出回应，并且出于所有实际目的，它确实会做出回应（尽管它确实是一个正在做这项工作的外交官）。

转发消息的对象因此从继承层次结构的两个分支“继承”方法 - 它自己的分支和响应消息的对象的分支。在上面的例子中，似乎Warrior类继承自Diplomat以及它自己的超类。

转发提供了您通常希望从多个继承中获得的大多数功能。但是，两者之间存在重要差异：多重继承在单个对象中组合了不同的功能。它倾向于大型，多方面的对象。另一方面，转发为不同的对象分配不同的职责。它将问题分解为较小的对象，但以对信息发件人透明的方式关联这些对象。

### 代理对象

转发不仅模仿多重继承，还可以开发代表或“覆盖”更多实质对象的轻量级对象。代理人代表其他对象并向其传达消息。

代理人在[_Objective-C编程语言_](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html#//apple_ref/doc/uid/TP30001163)中的“远程消息传递”中讨论过这样的代理。代理负责将消息转发到远程接收器的管理细节，确保通过连接复制和检索参数值，等等。但它并没有尝试做太多其他事情; 它不会复制远程对象的功能，只是简单地为远程对象提供一个本地地址，一个可以在另一个应用程序中接收消息的地方。

其他种类的替代物也是可能的。例如，假设您有一个操纵大量数据的对象 - 可能会创建一个复杂的图像或读取磁盘上文件的内容。设置此对象可能非常耗时，因此您更喜欢懒惰地执行此操作 - 在确实需要时或系统资源暂时空闲时。同时，您至少需要此对象的占位符才能使应用程序中的其他对象正常运行。

在这种情况下，你最初可以创建，而不是完整的对象，但它是一个轻量级的代理。这个对象可以自己做一些事情，比如回答有关数据的问题，但大多数情况下它只是为较大的对象保留一个位置，并且当时间到来时，将消息转发给它。当代理`forwardInvocation:`方法首先收到发往另一个对象的消息时，它将确保该对象存在并且如果不存在则创建它。较大对象的所有消息都通过代理，因此，就程序的其余部分而言，代理和较大的对象将是相同的。

### 转发和继承

转发模仿继承，这个`NSObject`类永远不会混淆两者。类似的方法`respondsToSelector:`和`isKindOfClass:`看只能在继承层次，从来没有在转发链。例如，如果询问Warrior对象是否响应`negotiate`消息，

```swift
if ( [aWarrior respondsToSelector:@selector(negotiate)] )
    ...
```

答案是`NO`，即使它可以毫无错误地接收`negotiate`消息并在某种意义上通过将它们转发给外交官来回应它们。（[见图5-1](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-87317)。）

在许多情况下，`NO`是正确的答案。但它可能不是。如果使用转发来设置代理对象或扩展类的功能，则转发机制可能应该像继承一样透明。如果您希望对象的行为就像它们真正继承了行为一样在他们转发消息的对象中，您需要重新实现`respondsToSelector:`和`isKindOfClass:`方法以包含转发算法：

```swift
- (BOOL)respondsToSelector:(SEL)aSelector
{
    if ( [super respondsToSelector:aSelector] )
        return YES;
    else {
        /* Here, test whether the aSelector message can     *
         * be forwarded to another object and whether that  *
         * object can respond to it. Return YES if it can.  */
    }
    return NO;
}
```

除了`respondsToSelector:`和`isKindOfClass:`之外，该`instancesRespondToSelector:`方法还应该镜像转发算法。如果使用协议，则`conformsToProtocol:`同样应将该方法添加到列表中。类似地，如果一个对象转发它收到的任何远程消息，它应该有一个版本，`methodSignatureForSelector:`它可以返回最终响应转发消息的方法的准确描述; 例如，如果一个对象能够将消息转发给其代理，您将按`methodSignatureForSelector:`如下方式实现：

```swift
- (NSMethodSignature*)methodSignatureForSelector:(SEL)selector
{
    NSMethodSignature* signature = [super methodSignatureForSelector:selector];
    if (!signature) {
       signature = [surrogate methodSignatureForSelector:selector];
    }
    return signature;
}
```

您可以考虑将转发算法放在私有代码中的某个位置，并将所有这些方法（`forwardInvocation:`包括在内）调用它。

**注意：**   这是一项高级技术，仅适用于无法提供其他解决方案的情况。它不是作为继承的替代品。如果必须使用此技术，请确保完全了解执行转发的类的行为以及要转发的类。

本节中提到的方法在[`NSObject`](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Classes/NSObject/Description.html#//apple_ref/occ/cl/NSObject)Foundation框架参考中的类规范中进行了描述。有关信息`invokeWithTarget:`，请参阅[`NSInvocation`](https://developer.apple.com/documentation/foundation/nsinvocation)Foundation框架参考中的类规范。

