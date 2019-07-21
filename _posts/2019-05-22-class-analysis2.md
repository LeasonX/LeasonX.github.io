---
layout: post
title:  ".class文件解析2--其他属性解析"
date:   2019-06-17
excerpt: "简单的class文件分析"
tag:
- jvm
- class
- java
comments: true
---

# 介绍

接着上节的解析结果,我们继续解析剩下的一些属性.

> .java文件源码

```java
package io.leason.demo;

public class ClazzStudy {
    private int m;

    public int addM() {
        return m + 1;
    }
}
```

> 编译之后的.class文件(javac编译器)

剩下的字节码我已经圈出来了

![class file code](../assets/img/2019-05-22-class-analysis/class-hex-code1.jpg)

# 解析

## 1. 访问标志

常量池后面紧接着是该class文件类或接口的访问标志信息(两个字节),表示该类或者接口是否是public,final,abstract等,具体的含义如下

|    标志名称    | 标志值 | 含义                                                                                                                                                            |
|----------------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ACC_PUBLIC     | 0X0001 | 是否为public                                                                                                                                                    |
| ACC_FINAL      | 0X0010 | 是否有final修饰                                                                                                                                                 |
| ACC_SUPER      | 0X0020 | 是否允许使用invokespecial字节码指令的新语意, invokespecial指令的语意在JDK1.0.2发审过改变, 为了区别这条指令使用哪种语意,JDK1.0.2之后编译 出来的class文件该值为真 |
| ACC_INTERFACE  | 0X0200 | 是否是个接口                                                                                                                                                    |
| ACC_ABSTRACT   | 0X0400 | 是否是抽象的,接口也是抽象的                                                                                                                                     |
| ACC_SYNTHETIC  | 0X1000 | 表示不是用户代码产生的类                                                                                                                                        |
| ACC_ANNOTATION | 0X2000 | 表示是个注解                                                                                                                                                    |
| ACC_ENUM       | 0X4000 | 表示是个枚举类                                                                                                                                                  |

`0X0021 = 0X0001 | 0X0020`,所以表示这个类是public的,因为用的是JDK1.8所以ACC_SUPER标志也为真,其它的接口标志什么的都为false.

## 2. 类索引、父类索引、接口索引集合

类索引占用u2结构空间,由于java是单继承的,所以父类索引也是用u2结构表示,而接口可以实现多个,所以用u2类型数据的集合(length:u2,value:length * u2).

可以解析出来的是类索引为0x0003,父类索引为0x0004,接口个数为0.

根据上节常量池的解析结果可知0x0003表示的是全限定类名为`io/leason/demo/ClazzStudy`的类索引,也能够理解.

父类对应的是全限定类名为`java/lang/Object`的类索引,跟我们所认知的所有类默认都是直接或间接继承于Object的是对应的.

## 3. 字段表集合

字段表表示声明在类或接口中的变量.修饰符一般都有访问修饰符如public、是否static、是否final、是否volatile、是否可序列化(transient)、字段名称等.有些修饰符可以用boolean值来表示,适合用标志位来表示.而名称一些属性则使用前面的常量池索引来表示.

字段表结构如下

|      类型      | 名称              | 数量            |
|----------------|-------------------|-----------------|
| u2             | access_flags      | 1               |
| u2             | name_index        | 1               |
| u2             | descriptor_index | 1               |
| u2             | attribute_count   | 1               |
| attribute_info | attributes        | attribute_count |

access_flags标志位和含义如下

|    标志名称   | 标志值 | 含义               |
|---------------|--------|--------------------|
| ACC_PUBLIC    | 0X0001 | 是否public         |
| ACC_PRIVATE   | 0X0002 | 是否private        |
| ACC_PROTECTED | 0X0004 | 是否protected      |
| ACC_STATIC    | 0X0008 | 是否静态           |
| ACC_FINAL     | 0X0010 | 是否final          |
| ACC_VOLATILE  | 0X0040 | 是否volatile       |
| ACC_TRANSIENT | 0X0080 | 是否可序列化       |
| ACC_SYNTHETIC | 0X1000 | 是否编译器自动生成 |
| ACC_ENUM      | 0X4000 | 是否枚举类型       |

开头的`00 01`表示这个类只有一个字段.

根据图中可以看出access_flags为`00 02`,所以表示这是个public修饰的字段,不为static、final等.

name_index为`00 05`,对应常量池的字符串为`m`,`descriptor_index`为`I`,那么`I`表示什么呢.

`I`表示字段的类型.一般如下表

| 标志字符 | 含义                           |
|----------|--------------------------------|
| B        | byte                           |
| C        | char                           |
| D        | double                         |
| F        | float                          |
| I        | int                            |
| J        | long                           |
| S        | short                          |
| Z        | boolean                        |
| V        | void                           |
| L        | 对象类型,例如Ljava/lang/String |

所以表明是个int类型的字段,后面的`attribute_count` = 0,这个属性一般用于字段的额外属性,例如static final类型的数据,就会有ConstantValue属性,指向常量池的部分信息.

所以上述表示的是`private int m;`这段代码.

### descriptor_index

我们来说下这个`descriptor_index`,翻译过来就是描述符索引,描述符适用于字段和方法,表示字段数据类型、方法的参数列表和返回值信息.
一般的字段类型描述符就如上表表示的一样,数组类型的字段则用`[`来表示,二维数组用两个`[`表示,比如`String[] a`则对应的描述符为`[Ljava/lang/String`.

用来描述方法时,按照 `(参数列表)返回值` 形式来表示, 例如`char[] toCharArray(String a)`的描述符为`(Ljava/lang/String)[C`.

## 4.方法表集合

方法表跟字段表表示相似，也分为访问标志、名称索引等一些信息，具体结构如下:

|      类型      | 名称             | 数量             |
|----------------|------------------|------------------|
| u2             | access_flags     | 1                |
| u2             | name_index       | 1                |
| u2             | descriptor_index | 1                |
| u2             | attributes_count | 1                |
| attribute_info | attributes       | attributes_count |

对着图片上class文件解析得到: 这个方法总共有`0x0002`(2)个方法

---

第一个方法 `access_flag` = `0x0001`，`name_index` = `0x0007`，`descriptor_index` = `0x0008`，`attribute_count` = `0x0001`，接着后面1个attribute_info单位的是属性表集合。

方法的访问access_flags对应的表示意义如下

|     标志名称     | 标志值 | 含义                             |
|------------------|--------|----------------------------------|
| ACC_PUBLIC       | 0X0001 | 方法是否为public                 |
| ACC_PRIVATE      | 0X0002 | 方法是否为private                |
| ACC_PROTECTED    | 0X0004 | 方法是否为protected              |
| ACC_STATIC       | 0X0008 | 方法是否为static                 |
| ACC_FINAL        | 0X0010 | 方法是否为final                  |
| ACC_SYNCHRONIZED | 0X0020 | 方法是否为synchornized           |
| ACC_BRIDGE       | 0X0040 | 方法是否是由编译器产生的桥接方法 |
| ACC_VARARGS      | 0X0080 | 方法是否接受不定参数             |
| ACC_NATIVE       | 0X0100 | 方法是否为native                 |
| ACC_ABSTRACT     | 0X0400 | 方法是否为abstract               |
| ACC_STRICTFP     | 0X0800 | 方法是否为strictfp               |
| ACC_SYNTHETIC    | 0X1000 | 方法是否是由编译器自动产生的     |

所以表示这个方法是public的，名称为`<init>`，方法描述符为`()V`，其实`<init>`表示这个方法是默认无参构造器的意思，与我们所学的java基础知识是符合的，默认为public，所有构造器都是不指定返回值的。后面的属性表不为空，所以接下来我们先分析下属性表是什么结构的，有哪些表示。

## 5.属性表集合

属性表的结构如下

| 类型 | 名称                 | 数量             |
|-----|----------------------|------------------|
| u2   | attribute_name_index | 1                |
| u4   | attribute_length     | 1                |
| u1   | info                 | attribute_length |

所以上述构造器的属性表信息为: `attribute_name_index` = `0X0009`(常量池中的`Code`字符串表示)，`attribute_length` = `0X0000001D`(29)，后面29个字节都表示info。

属性表结构很多，如下所示




### 5.1 Code属性

Code属性表示的是方法里面的代码经过编译之后，最终以字节码指令的形式存在Code属性中。而Code属性标的结构如下

|      类型      | 名称                   | 数量                   |
|--------------|------------------------|------------------------|
| u2             | attribute_name_index   | 1                      |
| u4             | attribute_length       | 1                      |
| u2             | max_stack              | 1                      |
| u2             | max_locals             | 1                      |
| u4             | code_length            | 1                      |
| u1             | code                   | code_length            |
| u2             | exception_table_length | 1                      |
| exception_info | exception_table        | exception_table_length |
| u2             | attributes_count       | 1                      |
| attribute_info | attribute              | attributes_count       |

其中前六个字节就是上面的前六个字节，后面从`max_stack`到`attribute`结束才是真正的主题内容，根据解析出来的是29个字节，我们等下可以验证下。

接着解析得出表中字段分别对应的值为

- `max_stack` = `0X0001` = 1
- `max_locals` = `0X0001` = 2
- `code_length` = `0X00000005` = 5
- `code` = `0X2A`、`0XB7`、`0X00`、`0X01`、`0XB1`
- `exception_table_length` = `0X0000` = 0
- `attribute_count`  = `0X0001`
- `attribute`: 
  - `attribute_name_index` = `0X000A` = 10 = `LineNumberTable`
  - `attribute_length` = `0X00000006` = 6
  - `line_number_table_length` = `0X0001` = 1
  - `line_number_table`:
    - `start_pc` = `0X0000` = 0
    - `line_number` = `0X0003` = 3

上述表示的是构造器方法最大栈深度为1，本地变量表长度为2，字节码指令长度为5，异常表长度为0(没有异常抛出)，有另外的属性表为LineNumberTable，这个属性表示java源码行号和字节码行号的对应关系，用来抛出异常时，显示出错的行号，还可以用来debug代码时设置断点用的。
其中的`start_pc`表示字节码行号，`line_number`表示的是java源码的行号。

code字节码分别在字节码指令表中表示的是
- `0X2A` = `aload_0` 将0个slot中为reference类型的本地变量推送到操作数栈顶
- `0XB7` =`invokespecial` 以栈顶reference类型的本地变量最为方法的接收者，调用此对象的实力构造器方法、private方法或者父类方法，后面的u2类型参数说明调用那个具体方法，是指向常量池的索引表示
- `0X0001`为invokespecial的参数，查常量池表为`<init>`方法的符号引用
- `0XB1` = `return` 方法返回，返回值为void

上面代码从`max_stack`到`line_number`正好为29字节，对应上面的`attribute_length`


---

第二个方法
