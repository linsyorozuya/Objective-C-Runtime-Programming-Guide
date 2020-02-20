# 声明的属性

当编译器遇到属性声明时（请参阅[_Objective-C编程语言_](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html#//apple_ref/doc/uid/TP30001163)中[_的_](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html#//apple_ref/doc/uid/TP30001163)[声明属性](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Chapters/ocProperties.html#//apple_ref/doc/uid/TP30001163-CH17)），它会生成与封闭类，类别或协议相关联的描述性元数据。您可以在类或协议上按名称查找属性，将属性类型作为`@encode`字符串获取，以及将属性列表作为C字符串数组复制的函数来访问此元数据。每个类和协议都有一个声明的属性列表。

### 属性类型和功能

该`Property`结构定义了属性描述符的不透明句柄。

```text
typedef struct objc_property *Property;
```

您可以使用这些函数`class_copyPropertyList 和 protocol_copyPropertyList`分别检索与类关联的属性数组（包括已加载的类别）和协议：

```text
objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)
objc_property_t *protocol_copyPropertyList(Protocol *proto, unsigned int *outCount)
```

例如，给定以下类声明：

```text
@interface Lender : NSObject {
    float alone;
}
@property float alone;
@end
```

您可以使用以下方式获取属性列表：

```text
id LenderClass = objc_getClass("Lender");
unsigned int outCount;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
```

您可以使用该`property_getName`函数来发现属性的名称：

```text
const char *property_getName(objc_property_t property)
```

您可以使用这些函数`class_getProperty`并`protocol_getProperty`分别获取对类和协议中具有给定名称的属性的引用：

```text
objc_property_t class_getProperty(Class cls, const char *name)
objc_property_t protocol_getProperty(Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty)
```

您可以使用该`property_getAttributes`函数来发现`@encode`属性的名称和类型字符串。有关编码类型字符串的详细信息，请参阅[类型编码](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1) ; 有关此字符串的详细信息，请参阅[属性类型字符串](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html#//apple_ref/doc/uid/TP40008048-CH101-SW6)和[属性属性描述示例](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html#//apple_ref/doc/uid/TP40008048-CH101-SW5)。

```text
const char *property_getAttributes(objc_property_t property)
```

将这些放在一起，您可以使用以下代码打印与类关联的所有属性的列表：

```text
id LenderClass = objc_getClass("Lender");
unsigned int outCount, i;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
for (i = 0; i < outCount; i++) {
    objc_property_t property = properties[i];
    fprintf(stdout, "%s %s\n", property_getName(property), property_getAttributes(property));
}
```

### 属性类型字符串

您可以使用该`property_getAttributes`函数来发现名称，`@encode`属性的类型字符串以及属性的其他属性。

该字符串以a开头，`T`后跟`@encode`类型和逗号，并以a 结尾，后跟`V`后备实例变量的名称。在这些属性之间，属性由以下描述符指定，用逗号分隔：

| **Table 7-1** |  |
| :--- | :--- |
|   Declared property type encodings |  |
| Code | Meaning |
| `R` | The property is read-only \(`readonly`\). |
| `C` | The property is a copy of the value last assigned \(`copy`\). |
| `&` | The property is a reference to the value last assigned \(`retain`\). |
| `N` | The property is non-atomic \(`nonatomic`\). |
| `G<name>` | The property defines a custom getter selector name. The name follows the `G` \(for example, `GcustomGetter,`\). |
| `S<name>` | The property defines a custom setter selector name. The name follows the `S` \(for example, `ScustomSetter:,`\). |
| `D` | The property is dynamic \(`@dynamic`\). |
| `W` | The property is a weak reference \(`__weak`\). |
| `P` | The property is eligible for garbage collection. |
| `t<encoding>` | Specifies the type using old-style encoding. |

有关示例，请参见[属性属性描述示例](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html#//apple_ref/doc/uid/TP40008048-CH101-SW5)。

### 属性属性描述示例

鉴于这些定义：

```text
enum FooManChu { FOO, MAN, CHU };
struct YorkshireTeaStruct { int pot; char lady; };
typedef struct YorkshireTeaStruct YorkshireTeaStructType;
union MoneyUnion { float alone; double down; };
```

下表显示了示例属性声明和返回的相应字符串`property_getAttributes`：

<table>
  <thead>
    <tr>
      <th style="text-align:left">Property declaration</th>
      <th style="text-align:left">Property description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>@property char charDefault;</code>
      </td>
      <td style="text-align:left"><code>Tc,VcharDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property double doubleDefault;</code>
      </td>
      <td style="text-align:left"><code>Td,VdoubleDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property enum FooManChu enumDefault;</code>
      </td>
      <td style="text-align:left"><code>Ti,VenumDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property float floatDefault;</code>
      </td>
      <td style="text-align:left"><code>Tf,VfloatDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property int intDefault;</code>
      </td>
      <td style="text-align:left"><code>Ti,VintDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property long longDefault;</code>
      </td>
      <td style="text-align:left"><code>Tl,VlongDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property short shortDefault;</code>
      </td>
      <td style="text-align:left"><code>Ts,VshortDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property signed signedDefault;</code>
      </td>
      <td style="text-align:left"><code>Ti,VsignedDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property struct YorkshireTeaStruct structDefault;</code>
      </td>
      <td style="text-align:left"><code>T{YorkshireTeaStruct=&quot;pot&quot;i&quot;lady&quot;c},VstructDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property YorkshireTeaStructType typedefDefault;</code>
      </td>
      <td style="text-align:left"><code>T{YorkshireTeaStruct=&quot;pot&quot;i&quot;lady&quot;c},VtypedefDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property union MoneyUnion unionDefault;</code>
      </td>
      <td style="text-align:left"><code>T(MoneyUnion=&quot;alone&quot;f&quot;down&quot;d),VunionDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property unsigned unsignedDefault;</code>
      </td>
      <td style="text-align:left"><code>TI,VunsignedDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property int (*functionPointerDefault)(char *);</code>
      </td>
      <td style="text-align:left"><code>T^?,VfunctionPointerDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>@property id idDefault;</code>
        </p>
        <p>Note: the compiler warns: <code>&quot;no &apos;assign&apos;, &apos;retain&apos;, or &apos;copy&apos; attribute is specified - &apos;assign&apos; is assumed&quot;</code>
        </p>
      </td>
      <td style="text-align:left"><code>T@,VidDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property int *intPointer;</code>
      </td>
      <td style="text-align:left"><code>T^i,VintPointer</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property void *voidPointerDefault;</code>
      </td>
      <td style="text-align:left"><code>T^v,VvoidPointerDefault</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>@property int intSynthEquals;</code>
        </p>
        <p>In the implementation block:</p>
        <p><code>@synthesize intSynthEquals=_intSynthEquals;</code>
        </p>
      </td>
      <td style="text-align:left"><code>Ti,V_intSynthEquals</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(getter=intGetFoo, setter=intSetFoo:) int intSetterGetter;</code>
      </td>
      <td style="text-align:left"><code>Ti,GintGetFoo,SintSetFoo:,VintSetterGetter</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(readonly) int intReadonly;</code>
      </td>
      <td style="text-align:left"><code>Ti,R,VintReadonly</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(getter=isIntReadOnlyGetter, readonly) int intReadonlyGetter;</code>
      </td>
      <td style="text-align:left"><code>Ti,R,GisIntReadOnlyGetter</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(readwrite) int intReadwrite;</code>
      </td>
      <td style="text-align:left"><code>Ti,VintReadwrite</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(assign) int intAssign;</code>
      </td>
      <td style="text-align:left"><code>Ti,VintAssign</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(retain) id idRetain;</code>
      </td>
      <td style="text-align:left"><code>T@,&amp;,VidRetain</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(copy) id idCopy;</code>
      </td>
      <td style="text-align:left"><code>T@,C,VidCopy</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(nonatomic) int intNonatomic;</code>
      </td>
      <td style="text-align:left"><code>Ti,VintNonatomic</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(nonatomic, readonly, copy) id idReadonlyCopyNonatomic;</code>
      </td>
      <td style="text-align:left"><code>T@,R,C,VidReadonlyCopyNonatomic</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>@property(nonatomic, readonly, retain) id idReadonlyRetainNonatomic;</code>
      </td>
      <td style="text-align:left"><code>T@,R,&amp;,VidReadonlyRetainNonatomic</code>
      </td>
    </tr>
  </tbody>
</table>