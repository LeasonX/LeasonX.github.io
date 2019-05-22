---
layout: post
title:  ".class文件解析"
date:   2019-04-28
excerpt: "简单的class文件分析"
tag:
- jvm
- class
- java
comments: true
---

# 介绍

《深入理解java虚拟机》看第二遍了,还是有好多问题和知识点疏漏和不懂,最近对着书复习了一点class文件的结构,所以写一篇博客来说记录下.

这次解析的源代码文件比较简单,使用的是jdk1.8.0_191.

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

贴代码块效果太差了,直接上截图吧
![class file code](../assets/img/2019-05-22-class-analysis/class-hex-code.png)

# 解析

class文件是一组以单字节(8bit)为单位的二进制流,一个字节占2个16进制的空间,class文件采用类似C语言的伪结构体,这种结构包括两种数据类型:无符号数和表.

无符号数属于基本数据类型,我们采用`u1`、`u2`、`u4`,`u8`分别表示一个字节、两个字节、四个字节和八个字节的无符号数.

表是有多个无符号数和其它数据项组合成的,下面会有例子更清楚.

整个class文件由下面的数据项组成

|        类型        |         名称        |          数量         |
|:------------------:|:-------------------:|:---------------------:|
|         u4         |        magic        |           1           |
|         u2         |    minor_version    |           1           |
|         u2         |    major_version    |           1           |
|         u2         | constant_pool_count |           1           |
| constant_pool_info |    constant_pool    | constant_pool_count-1 |
|         u2         |     access_flags    |           1           |
|         u2         |      this_class     |           1           |
|         u2         |     super_class     |           1           |
|         u2         |   interfaces_count  |           1           |
|         u2         |      interfaces     |    interfaces_count   |
|         u2         |     fields_count    |           1           |
|     filed_info     |        fileds       |      field_count      |
|         u2         |    methods_count    |           1           |
|     method_info    |       methods       |     methods_count     |
|         u2         |   attributes_count  |           1           |
|   attribute_info   |      attribute      |    attributes_count   |

上面注意的一点就是接口并不是一个表结构,而是一个u2大小的数据表示,指向常量池对应索引的数据信息,其它的则是info结尾的表结构数据

上面的信息表示class文件的各个字节表示的数据都是明确的,没有填充对齐的字节,顺序和长度都是固定的,这种考虑是因为java语言最初是面向智能家电和网络的.

## magic

根据上表可知class文件的前4个字节表示的是magic,翻译为魔数,对应图上的`CA FE BA BE`4个字节,主要是用来表示这个文件是虚拟机识别的class文件.约定俗成,可以google其来源.

## monior_version 、major_version

魔数后面的两个u2数据分别是`0000`和`0034`,分别对应的二进制是 0 和 52,分别对应的是编译器JDK的次、主版本号,每个大版本发布都会在主版本号上+1,而JDK1.0使用的版本号是45,从而推算出我使用的JDK版本号是JDK8

## constant_pool

紧接版本号后面的是常量池信息,主要存放一些字面量和符号引用,字面量主要意思类似于我们定义的基本类型的final值,包括字符串一些.符号引用就是用来唯一确定类、接口、字段、方法一些名称和描述符(公私有等一些修饰符)的.

前两个字节表示的是常量池中的常量的个数,由于是2个字节,所以最大值为25535个.
图中对应的个数为`1 * 16 ^ 1 + 3 * 16 ^ 0 = 19`个常亮,而所有常量池默认第一个常量用来表示`没有任何引用的常亮`的意思,比如用来表示匿名内部类的名称的信息,所以后面接着的18个`constant_pool_info`类型的数据就是常量池中的常量信息.

### constat_pool_info

constant_pool_info是一种表结构数据,class文件中一般存在14种这种类型的结构数据,如下表

|               类型               | tag |           描述           |
|:--------------------------------:|:---:|:------------------------:|
|        CONSTANT_Utf8_info        |  1  |     UTF-8编码的字符串    |
|       CONSTANT_Integer_info      |  3  |        整型字面量        |
|        CONSTANT_Float_info       |  4  |       浮点型字面量       |
|        CONSTANT_Long_info        |  5  |       长整型字面量       |
|       CONSTANT_Double_info       |  6  |    双精度浮点型字面量    |
|        CONSTANT_Class_info       |  7  |   类或者接口的符号引用   |
|       CONSTANT_String_info       |  8  |     字符串类型字面量     |
|      CONSTANT_Fieldref_info      |  9  |      字段的符号引用      |
|      CONSTANT_Methodref_info     |  10 |    类中方法的符号引用    |
| CONSTANT_InterfaceMethodref_info |  11 |   接口中方法的符号引用   |
|     CONSTANT_NameAndType_info    |  12 | 字段或方法的部分符号引用 |
|    CONSTANT_MethodHandle_info    |  15 |       表示方法句柄       |
|     CONSTANT_MethodType_info     |  16 |       表示方法类型       |
|    CONSTANT_InvokeDynamic_info   |  18 | 表示识一个动态方法调用点 |

每个类型的constant_pool_info都有不同的数据结构类型,而共同点都是前1个字节表示的都是tag,所以可以根据tag对应的类型所占用的数据结构一一解析出来整个常量池.

#### 常量解析

1.紧跟着`0013`后面的第一个字节是`0A`对应表中的`CONSTANT_Methodref_info`,而这个类型的表数据结构为


