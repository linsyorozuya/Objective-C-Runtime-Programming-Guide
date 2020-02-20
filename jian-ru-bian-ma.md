# 键入编码

为了协助运行时系统，编译器进行编码 返回值 和参数类型对于字符串中的每个方法，并将该字符串与方法选择器相关联。它使用的编码方案在其他上下文中也很有用，因此可以使用`@encode()`编译器指令公开。给定类型规范时，`@encode()`返回编码该类型的字符串。类型可以是基本类型，例如`int`，指针，标记结构或联合，或类名 - 事实上，任何类型都可以用作C `sizeof()`运算符的参数。

```text
char *buf1 = @encode(int **);
char *buf2 = @encode(struct key);
char *buf3 = @encode(Rectangle);
```

下表列出了类型代码。请注意，它们中的许多重叠与编码对象时用于存档或分发的代码重叠。但是，此处列出的代码在编写编码器时无法使用，并且在编写非编译器时可能需要使用的代码`@encode()`。（有关[`NSCoder`](https://developer.apple.com/library/archive/documentation/LegacyTechnologies/WebObjects/WebObjects_3.5/Reference/Frameworks/ObjC/Foundation/Classes/NSCoder/Description.html#//apple_ref/occ/cl/NSCoder)编码对象以进行存档或分发的详细信息，请参阅Foundation Framework参考中的类规范。）

**Table 6-1**  Objective-C type encodings

| Code | Meaning |
| :--- | :--- |


| `c` | A `char` |
| :--- | :--- |


| `i` | An `int` |
| :--- | :--- |


| `s` | A `short` |
| :--- | :--- |


<table>
  <thead>
    <tr>
      <th style="text-align:left"><code>l</code>
      </th>
      <th style="text-align:left">
        <p>A <code>long</code>
        </p>
        <p><code>l</code> is treated as a 32-bit quantity on 64-bit programs.</p>
      </th>
    </tr>
  </thead>
  <tbody></tbody>
</table>| `q` | A `long long` |
| :--- | :--- |


| `C` | An `unsigned char` |
| :--- | :--- |


| `I` | An `unsigned int` |
| :--- | :--- |


| `S` | An `unsigned short` |
| :--- | :--- |


| `L` | An `unsigned long` |
| :--- | :--- |


| `Q` | An `unsigned long long` |
| :--- | :--- |


| `f` | A `float` |
| :--- | :--- |


| `d` | A `double` |
| :--- | :--- |


| `B` | A C++ `bool` or a C99 `_Bool` |
| :--- | :--- |


| `v` | A `void` |
| :--- | :--- |


| `*` | A character string \(`char *`\) |
| :--- | :--- |


| `@` | An object \(whether statically typed or typed `id`\) |
| :--- | :--- |


| `#` | A class object \(`Class`\) |
| :--- | :--- |


| `:` | A method selector \(`SEL`\) |
| :--- | :--- |


| \[_array type_\] | An array |
| :--- | :--- |


| {_name=type..._} | A structure |
| :--- | :--- |


| \(_name_=_type..._\) | A union |
| :--- | :--- |


| `b`num | A bit field of _num_ bits |
| :--- | :--- |


| `^`type | A pointer to _type_ |
| :--- | :--- |


| `?` | An unknown type \(among other things, this code is used for function pointers\) |
| :--- | :--- |


**重要说明：**  Objective-C不支持该`long double`类型。`@encode(long double)`返回`d`，与编码`double`相同。

数组的类型代码用方括号括起来; 数组中元素的数量是在数组类型之前的开括号之后立即指定的。例如，一个包含12个指针的数组`float`将被编码为：

```text
[12^f]
```

结构在括号内指定，括号内的联合指定。首先列出结构标记，然后是等号，并按顺序列出结构字段的代码。例如，结构

```text
typedef struct example {
    id   anObject;
    char *aString;
    int  anInt;
} Example;
```

将编码如下：

```text
{example=@*i}
```

无论是否传递定义的类型name（`Example`）或结构标记（`example`），都会产生相同的编码结果`@encode()`。结构指针的编码带有关于结构字段的相同数量的信息：

```text
^{example=@*i}
```

但是，另一个间接级别会删除内部类型规范：

```text
^^{example}
```

对象被视为结构。例如，传递`NSObject`类名以`@encode()`产生此编码：

```text
{NSObject=#}
```

本`NSObject`类仅声明了一个实例变量，`isa`类类。

请注意，虽然`@encode()`指令不返回它们，但运行时系统使用表6-2中列出的附加编码来表示类型限定符，当它们用于声明协议中的方法时。

**Table 6-2**  Objective-C method encodings

| Code | Meaning |
| :--- | :--- |


| `r` | `const` |
| :--- | :--- |


| `n` | `in` |
| :--- | :--- |


| `N` | `inout` |
| :--- | :--- |


| `o` | `out` |
| :--- | :--- |


| `O` | `bycopy` |
| :--- | :--- |


| `R` | `byref` |
| :--- | :--- |


| `V` | `onewa` |
| :--- | :--- |


