# 消息

本章介绍如何将消息表达式转换为[`objc_msgSend`](https://developer.apple.com/documentation/objectivec/1456712-objc_msgsend)函数调用，以及如何按名称引用方法。然后，它解释了如何利用`objc_msgSend`，以及如何 - 如果需要 - 您可以绕过动态绑定。

### objc\_msgSend函数

在Objective-C中，消息直到运行时才绑定到方法实现。编译器转换消息表达式，

```text
[receiver message]
```

对消息功能的调用， [`objc_msgSend`](https://developer.apple.com/documentation/objectivec/1456712-objc_msgsend)。此功能接收 接收器（receiver） 和消息中提到的方法的名称 （ 即方法选择器） 作为它的两个主要参数：

```text
objc_msgSend(receiver, selector)
```

任何参数传递给消息页递交给`objc_msgSend`：

```text
objc_msgSend(receiver, selector, arg1, arg2, ...)
```

消息传递功能可以完成动态绑定所需的一切：

* 它首先找到选择器的过程（方法实现）。由于相同的方法可以通过单独的类以不同方式实现，因此它找到的精确过程取决于接收器的类。
* 然后它调用该过程，将接收对象（指向其数据的指针）以及为该方法指定的任何参数传递给它。
* 最后，它将过程的返回值作为自己的返回值传递。

**注意：**  编译器生成对消息传递功能的调用。你永远不应该直接在你编写的代码中调用它。

消息传递的关键在于编译器为每个类和对象构建的结构。每个类结构都包括以下两个基本要素：

* 指向超类的指针。
* 类_调度表（dispatch table）_。此表具有将方法选择器与它们标识的特定于类的地址的方法相关联的条目。`setOrigin::`方法的选择器与（实现的过程）的地址相关联，方法选择器`display`与`display`地址相关联，依此类推。

创建新对象时，将分配其内存，并初始化其实例变量。对象变量中的第一个是指向其类结构的指针。这个被调用的指针`isa`使对象可以访问它的类，并通过该类访问它继承的所有类。

**注意：**  虽然严格来说不是语言的一部分，但`isa`指针是对象使用Objective-C运行时系统所必需的。对象需要与“等效”`struct objc_object`（定义于`objc/objc.h`）结构定义的任何字段中。但是，您很少（如果有的话）需要创建自己的根对象，只要继承`NSObject`或`NSProxy`自动拥有`isa`变量的对象。

类和对象结构的这些元素如图3-1所示。**图3-1**   消息传递框架

![](media/messaging1.gif)

当消息发送到对象时，消息传递功能通过对象的`isa` 指针指向类结构，然后在调度表中查找方法选择器。如果它找不到那里的选择器，则`objc_msgSend`跟随指向超类的指针并尝试在其调度表中找到选择器。连续的失败导致`objc_msgSend`爬升类层次结构直到它到达`NSObject`类。一旦找到选择器，该函数就会调用表中输入的方法并将接收对象的数据结构传递给它。

这是在运行时选择方法实现的方式 - 或者在面向对象编程的术语中，方法动态地绑定到消息。

为了加速消息传递过程，运行时系统在使用它们时缓存方法的选择器和地址。每个类都有一个单独的缓存，它可以包含继承方法的选择器以及类中定义的方法。在搜索调度表之前，消息传递例程首先检查接收对象类的高速缓存（理论上可能会再次使用一次使用的方法）。如果方法选择器位于缓存中，则消息传递仅比函数调用稍慢。一旦程序运行了足够长的时间来“预热”其缓存，它发送的几乎所有消息都会找到一个缓存的方法。随着程序的运行，缓存会动态增长以容纳新消息。

### 使用隐藏的参数

当[`objc_msgSend`](https://developer.apple.com/documentation/objectivec/1456712-objc_msgsend)找到实现方法的过程时，它会调用该过程并传递给消息中的所有参数。它还传递了两个隐藏的参数：

* 接收对象
* 选择器 对于该方法

这些参数为每个方法实现提供有关调用它的消息表达式的两半的显式信息。它们被称为“隐藏”，因为它们未在定义方法的源代码中声明。它们在编译代码时插入到实现中。

虽然这些参数没有显式声明，但源代码仍然可以引用它们（就像它可以引用接收对象的实例变量一样）。方法将接收对象称为`self`，以及它自己的选择器 `_cmd`。在下面的示例中，`_cmd`引用`strange`方法的选择器和`self`接收`strange`消息的对象。

```text
- strange
{
    id  target = getTheReceiver();
    SEL method = getTheMethod();
 
    if ( target == self || method == _cmd )
        return nil;
    return [target performSelector:method];
}
```

`self`在两个参数中是更有用的那个。事实上，它是接收对象的实例变量可用于方法定义的方式。

### 获取方法地址

规避动态绑定的唯一方法是获取方法的地址并直接调用它就好像它是一个函数。这种情况可能适用于极少数情况下，特定方法将连续多次执行，并且您希望每次执行方法时都避免消息传递的开销。

使用`NSObject`类中定义的方法，`methodForSelector:`，您可以获取指向实现方法的过程的指针，然后使用指针调用该过程。`methodForSelector:`返回的指针必须小心地转换为正确的函数类型。返回和参数类型都应该包含在强制转换中。

下面的示例显示了如何`setFilled:`调用实现该方法的过程：

```text
void (*setter)(id, SEL, BOOL);
int i;
 
setter = (void (*)(id, SEL, BOOL))[target
    methodForSelector:@selector(setFilled:)];
for ( i = 0 ; i < 1000 ; i++ )
    setter(targetList[i], @selector(setFilled:), YES);
```

传递给过程的前两个参数是接收对象（`self`）和方法选择器（`_cmd`）。这些参数隐藏在方法语法中，但在将方法作为函数调用时必须使其显式化。

使用`methodForSelector:`规避动态绑定可以节省大部分通过短信所需的时间。但是，只有在特定消息重复多次的情况下，节省才会显着，如`for`上面所示的循环。

注意，它`methodForSelector:`是由Cocoa运行时系统提供的; 它不是Objective-C语言本身的一个特性。

