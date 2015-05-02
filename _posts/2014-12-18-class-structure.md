---
layout: post
title: Java Class 文件结构
categories: JVM
tags: 笔记 java
---

#### Class 类文件的结构

Class 文件是一组以8位字节位基础单元的二进制流，各个数据项目严格按照顺序紧凑地排列在 Class 文件之中，之间没有任何分隔符。当遇到需要占用8位字节以上空间的数据项，则会按照高位在前的方式分割成若干个8位字节进行存储。

无符号属性属于基本的数据类型，以 u1、u2、u4、u8 来分别代表1个字节、2个字节、4个字节和8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值。

整个 Class 文件本质上就是一张表，如下：

类型|名称|数量
:--|:--|:--
u4|magic|1
u2|minor_version|1
u2|major_version|1
u2|`constant_pool_count`|1
cp_info|constant_pool|`constant_pool_count ` - 1
u2|access_flags|1
u2|this_class|1
u2|super_class|1
u2|interfaces_count|1
u2|interfaces|interfaces_count
u2|fields_count|1
field_info|fileds|fields_count
u2|methods_count|1
method_info|methods|methods_count
u2|attribute_count|1
attribute_info|attributes|attribute_count

#### 魔数与 Class 文件的版本

每个 Class 文件的头4个字节成为魔数，他的唯一作用数由于确认整个文件是否为一个能被虚拟机接受的 Class 文件。第5-6字节是次版本号，第7-8字节是主版本号。

#### 常量池

在主版本之后是常量池入口，常量池是 Class 文件结构中与其他项目关联最多的数据类型。这个容量的计数是从1开始的。如果常量池的值是0x0016，即十进制22，就代表常量池中有21个常量，索引值是1-21。

Class 文件结构中只有常量池的容量计数是从1开始的，其他的都是从0开始的。

常量池中主要存放两大类常量：字面量和符号引用。

字面量如文本字符串、被声明为 final 的常量值等。

符号引用包括下面三类：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

常量池中的每一项常量都是一个表，共有11种结构不同的表，这11种表都有一个共同的特点，就是表开始的第一位是一个 u1 类型的标志位（tag，取值为1-12，缺少标志位2的数据类型），代表当前整个常量属于哪种常量类型，它的取值如下表：

类型|标志|描述
:--|:--|:--
`CONSTANT_Utf8_info`|1|utf-8缩略编码字符串
`CONSTANT_Integer_info`|3|整型字面量
`CONSTANT_Float_info`|4|浮点型字面量
`CONSTANT_Long_info`|5|长整型字面量
`CONSTANT_Double_info`|6|双精度浮点型字面量
`CONSTANT_Class_info`|7|类或接口的符号引用
`CONSTANT_String_info`|8|字符串类型字面量
`CONSTANT_Fieldref_info`|9|字段的符号引用
`CONSTANT_Methodref_info`|10|类中方法的符号引用
`CONSTANT_InterfaceMethodref_info`|11|接口中方法的符号引用
`CONSTANT_NameAndType_info`|12|字段或方法的部分符号引用

常量池当中的11种数类型的结构总表如下图：

![常量池1](http://renchx.com/public/images/constant1.png)
![常量池2](http://renchx.com/public/images/constant2.png)

#### 访问标志

常量池结束之后，接着的2个字节代表访问标志(access_flags)，这个标志用于识别一些类或接口层次的访问信息，包括：

- 这个 Class 是类还是接口
- 是否定义为 public 类型
- 是否定义位 abstract 类型
- 如果是类，是否生命位 final 等等

如下表：

标志名称|标志值|含义
:--|:--|:--
ACC_PUBLIC|0x0001|是否为 public 类型
ACC_FINAL|0x0010|是否被声明为 final，只有类可设置
ACC_SUPER|0x0020|是否允许使用 invokespecial 字节码指令，jdk 1.2 之后编译出来的类整个标志为真
ACC_INTERFACE|0x0200|标识这是个接口
ACC_ABSTRACT|0x0400|是否为 abstract 类型，对于接口或抽象类来说整个标志是真，其他为假
ACC_SYNTHETIC|0x1000|标识这个类并非由用户代码产生
ACC_ANNOTATION|0x2000|标识这是一个注解
ACC_ENUM|0x4000|标识这是一个枚举

每种访问信息都由一个十六进制的标志值表示，如果同时具有多种访问信息，则得到的标志值为这几种访问信息的标志值的逻辑或。

#### 类索引、父类索引与接口索引集合

- u2 `this_class` 表示类的常量池索引，指向常量池中 `CONSTANT_Class_info` 的常量

- u2 `super_class` 表示超类的索引，指向常量池中 `CONSTANT_Class_info` 的常量

- u2 interfaces_counts 表示接口的数量

- u2 interfaces[`interfaces_counts`]表示接口表，它里面每一项都指向常量池中 `CONSTANT_Class_info` 常量

Class 文件中由这三项数据来确定整类的继承关系。由于 Java 是单继承，所以父类索引只有一个，除了 java.lang.Object 外，所有 Java 类的父类索引都不是0。

#### 字段表集合

field_info 用于表述接口或类中声明的变量。字段包括了类级别变量和实例变量，但是不包括定义在方法内的变量。

字段 (field_info) 结构表如下：

类型|名称|数量
:--|:--|:--
u2|access_flags|1
u2|name_index（对常量池的引用，代表字段的简单名称）|1
u2|descriptor_index（对常量池的引用，代表字段的描述符）|1
u2|attributes_count|1
u2|attributes|attributes_count

字段表集合当中不会包含超类或者父类接口中继承而来的字段，但是有可能包含不存在的字段，例如内部类中为了保持对外部类的访问性，就会自动添加指向外部类实例的字段。

字段修饰符放在 `access_flags` 项目中，它与类中的 `access_flags` 项目是类似的都是一个 u2 的数据类型，具体标志位如下表：

标志名称|标志值|含义
:--|:--|:--
ACC_PUBLIC|0x0001|字段是否为 public
ACC_PRIVATE|0x0002|字段是否为 private
ACC_PROTECTED|0x0004|字段是否为 protected
ACC_STATIC|0x0008|字段是否为 static
ACC_FINAL|0x0010|字段是否为 final
ACC_VOLATILE|0x0040|字段是否为 volatile
ACC_TRANSIENT|0x0080|字段是否为 transient
ACC_SYNTHETIC|0x1000|字段是否由编译器自动产生的
ACC_ENUM|0x4000|字段是否为 enum

#### 简单名称、描述符、全限定名

例如：java.lang.Object 这个类，它的全限定名是 “java/lang/Object”，使用时会在后面加上“;”表示结束。

简单名称就是指没有类型和参数修饰的方法或字段名称，如果类中由个方法是这样 ```public void method() {...}``` 那么它的简单名称就是 method。

描述符是描述字段的数据类型、方法的参数列表和返回值。

描述符的含义可以看下表：

标识字符|含义
:--|:--
B|byte
C|char
D|double
F|float
I|int
J|long
S|short
Z|boolean
V|void
L|对象类型，如Ljava/lang/Object;

对于数组类型，每一维度将使用一个前置的 “[” 字符来表示，例如定义一个 `String[][]` 类型的二维数组，将被记录为： “[[Ljava/lang/String;” 。

描述符来描述方法时，按照先参数列表后返回值的顺序描述，参数列表按照参数的顺序放在一组小括号 `()` 之内。

如果方法是 void inc() 那么描述符是 ()V

如果方法是 String toString() 那么描述符是 ()Ljava/lang/String

如果方法是 int indexOf(char[] source,int offset,double dd) 那么描述符是 ([CID)I

#### 方法表集合

方法表集合与字段表集合差不多，如下表：

类型|名称|数量
:--|:--|:--
u2|access_flags|1
u2|name_index（对常量池的引用，代表方法的简单名称）|1
u2|descriptor_index（对常量池的引用，代表方法的描述符）|1
u2|attributes_count|1
u2|attributes|attributes_count

方法的访问标志表如下：

标志名称|标志值|含义
:--|:--|:--
ACC_PUBLIC|0x0001|方法是否为 public
ACC_PRIVATE|0x0002|方法是否为 private
ACC_PROTECTED|0x0004|方法是否为 protected
ACC_STATIC|0x0008|方法是否为 static
ACC_FINAL|0x0010|方法是否为 final
ACC_SYNCHRONIZED|0x0020|方法是否为 synchronized
ACC_BRIDGE|0x0040|方法是否由编译器产生的桥接方法
ACC_VRARGS|0x0080|方法是否为不定参数
ACC_NATIVE|0x0100|方法是为 native
ACC_ABSTRACT|0x0400|方法是为 abstract
ACC_STRICT|0x0800|方法是为 strictfp
ACC_SYNTHETIC|0x1000|方法是否由编译器自动产生的

#### 属性表集合

虚拟机规范预定义的属性表：

属性名称|使用位置|含义
:--|:--|:--
Code|方法表|Java 代码编译成的字节码
ConstantValue|字段表|final 关键字定义的常量
Deprecated|类、方法、字段表|被声明为 deprecated 的方法和字段
Exceptions|方法表|方法抛出的异常
InnerClasses|类文件|内部类列表
LineNumberTable|Code 属性|Java 源码的行号与字节码指令的对应关系
LocalVaribaleTable|Code 属性|方法的局部变量描述
SourceFile|类文件|源文件名称
Synthetic|类、方法表、字段表|标识方法或字段为编译器自动生成的

对于每个属性，它的名称都需要从常量池中引用一个 `CONSTANT_Utf8_info` 类型的常量来表示，而属性的结构是完全自定义的，只需要说明属性值所占用的位数长度即可。
