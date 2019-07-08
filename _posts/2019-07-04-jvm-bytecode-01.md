---
layout: post
title: JVM java字节码文件结构
---

首先请Java虚拟机规范出场：
https://docs.oracle.com/javase/specs/jvms/se8/html/

MyTest1.java源文件如下：

```java
package com.jvm.bytecode;

public class MyTest1 {
    private int a = 1;

    public int getA() {
        return a;
    }

    public void setA(int a) {
        this.a = a;
    }
}
```

第一步：进入classes目录，用Java命令来查看MyTest1.class字节码文件。

D:\IdeaProjects\jvm_study\out\production\classes>javap -c com.jvm.bytecode.MyTest1

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode01.png)


D:\IdeaProjects\jvm_study\out\production\classes>javap -verbose com.jvm.bytecode.MyTest1

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode02.png)
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode03.png)
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode04.png)

1.使用 javap -verbose 命令分析一个字节码文件时，将会分析该字节码文件的魔数、版本号、常量池、类信息、类的构造方法、类中的方法信息、类变量与成员变量等信息。

2.魔数：所有的.class字节码文件的前4个字节都是魔数，魔数值为固定值：0xCAFEBABE。

3.魔数之后的4个字节为版本信息，前两个字节表示minor version（次版本号），后两个字节表示major version（主版本号）。这里的版本号为00 00 00 34，换算成十进制，表示次版本号为0，主版本号为52。所以，该文件的版本号为1.8.0，其中52对应jdk版本为1.8。可以通过java -version命令来验证这一点。

4.常量池（constant pool）：紧接着主版本后之后的就是常量池入口。一个Java类中定义的很多信息都是由常量池来维护和描述的，可以将常量池看作是class文件的资源仓库，比如说Java类中定义的方法与变量信息，都是存储在常量池中。常量池中主要存储两类常量：字面量与符号引用。字面量如文本字符串，Java中声明为final的常量值等，而符号引用如类和接口的全局限定名，字段的名称和描述符，方法的名称和描述符等。

5.常量池的总体结构：Java类所对应的常量池主要由常量池数量与常量池数组（常量表）这两部分共同构成。常量池数量紧跟在主版本号后面，占据两个字节；常量池数组则紧跟在常量池数量之后。常量池数组与一般的数组不同的是，常量池数组中不同的元素的类型、结构都是不同的，长度当然也不同；但是，每一种元素的第一个数据都是u1类型，该字节是个标志位，占据1个字节。JVM在解析常量池时，会根据这个u1类型来获取元素的具体类型。值得注意的是，常量池数组中元素的个数 = 常量数 – 1（0暂时不使用），目的的是满足某些常量池索引值的数据在特定情况下需要表达【不引用任何一个常量池】的含义；根本原因在于，索引为0也是一个常量（保留常量），只不过它不位于常量表中，这个常量就对应null值；所以常量的索引从1而非0开始。

6.在JVM规范中，每个变量/字段都有描述信息，描述信息的主要作用是描述字段的数据类型、方法的参数列表（包括数量、类型与顺序）与返回值。根据描述规则，基本数据类型和代表无返回值的void类型都用一个大写字母来表示，对象类型则使用字符L加对象的全限定名称来表示。为了压缩字节码文件的体积，对于基本数据类型，JVM都使用一个大写字母来表示，如下所示：B – byte,  C – char,  D – double,  F – float,  I – int,  J – long,  S – short,
Z – boolean,  V – void,  L – 对象类型，如Ljava/lang/String;

7.对于数组类型来说，每一个维度都是用一个前置的[来表示，如int[]被记录为[I，String[][]被记录为[[Ljava/lang/String;

8.用描述符描述方法时，按照先参数列表，后返回值的顺序来描述。参数列表按照参数的严格顺序放在一组()之内，如方法：String getReakNameAndNickName(int id, String name)的描述符为：(I, Ljava/lang/String;)Ljava/lang/String;

先来看一下MyTest1.class是以16进制存储的文件：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode05.png)

其实，class文件也是严格按照规定来编排的，它遵循着下面的结构：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode06.png)

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode10.png)

我们开始分析：

前4个字节为魔数：Magic Number，0xCA FE BA BE。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode07.png)


魔数之后的4个字节为版本信息：Version，0x00 00 00 34，前两个字节表示minor version（次版本号），后两个字节表示major version（主版本号）。换算成十进制，表示次版本号为0，主版本号为52。其中52对应jdk版本为1.8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode08.png)


紧接着主版本号之后的就是常量池入口：常量池主要由常量池数量与常量池数组（常量表）这两部分共同构成。常量池数量紧跟在主版本号后面，占据两个字节；常量池数组则紧跟在常量池数量之后。<br/>
常量池数量：0x00 18，换算成10进制，表示24。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode09.png)

常量表的分析要用到下面的表格：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode11.png)

常量池数组中不同的元素的类型、结构都是不同的，长度当然也不同；但是，每一种元素的第一个数据都是u1类型，该字节是个标志位，占据1个字节。

常量表的第一个元素的第一个数据是：0A，换算成10进制，表示10，去表格中找u1类型，值为10的，Methodref。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode12.png)

下图为值为10的常量：Methodref
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode13.png)

CONSTANT_Methodref_Info：该常量包含三部分，
tag,index,index。我们已经知道tag的值。
第一个index：00 04，指向声明方法的类描述符CONSTANT_Class_info的索引项。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode14.png)

第二个index：00 14，换算成10进制，表示20。指向名称及类型描述符CONSTANT_NameAndType_info的索引项。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode15.png)

以上是常量池中第一个元素的信息：0x0A 00 04 00 14，表示Methodref。

第二个元素的第一个数据是：09。去表格中找u1类型，值为9的，Fieldref。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode16.png)

下图值为9的常量：Fieldref
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode17.png)

CONSTANT_Fieldref_Info：该常量包含三部分，
tag,index,index。我们已经知道tag的值。
第一个index：00 03，指向声明字段的类或者接口描述符CONSTANT_Class_info的索引项。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode18.png)

第二个index：00 15，换算成10进制，表示21。指向字段描述符CONSTANT_NameAndType_info的索引项。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode19.png)

以上是常量池中第二个元素的信息：0x09 00 03 00 15 ，表示Fieldref。

第三个元素的第一个数据是：07。去表格中找u1类型，值为7的，Class。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode20.png)

下图值为7的常量：Class
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode21.png)

CONSTANT_Fieldref_Info：该常量包含两部分， tag,index。我们已经知道tag的值。 index：00 16，换算成10进制，表示22。指向全限定名常量项的索引。

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode22.png)

以上是常量池中第三个元素的信息：0x07 00 16 ，表示Class。

第四个元素的第一个数据是：07。去表格中找u1类型，值为7的，Class。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode23.png)

CONSTANT_Class_Info：该常量包含两部分， tag,index。我们已经知道tag的值。 index：00 17，换算成10进制，表示23。指向全限定名常量项的索引。

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode24.png)

以上是常量池中第四个元素的信息：0x07 00 17 ，表示Class。

第五个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode25.png)

下图值为7的常量：Utf8
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode26.png)

CONSTANT_Utf8_Info：该常量包含三部分，tag，length，bytes。我们已经知道tag的值。<br/>
length：00 01。UTF-8编码的字符串长度。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode27.png)

byte：61。换算成10进制，表示97，对应字符串为a。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode28.png)

以上是常量池中第五个元素的信息：01 00 01 61 ，表示Utf8。

第六个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode29.png)

length：00 01
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode30.png)

byte：49。换算成10进制，表示73，对应字符串为I。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode31.png)

以上是常量池中第六个元素的信息：01 00 01 49 ，表示Utf8。

第七个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode32.png)

length：00 06
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode33.png)

byte：3C 69 6E  69 74 3E。length后面数6个字节，换算成10进制，表示60 105 110 105 116 52，每个字节对应的字符串分别为< i n i t >，合起来就是<init>。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode34.png)

以上是常量池中第七个元素的信息：01 00 06 3C 69 6E  69 74 3E，表示Utf8。

第八个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode35.png)

length：00 03
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode36.png)

byte：28 29 56。length后面数3个字节，换算成10进制，表示40 41 86，每个字节对应的字符串分别为( ) V，合起来就是()V。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode37.png)

以上是常量池中第八个元素的信息：01 00 03 28 29 56，表示Utf8。

第九个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode38.png)

length：00 04
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode39.png)

byte：43 6F 64 65。length后面数4个字节，换算成10进制，表示67 111 100 101，每个字节对应的字符串分别为C o d e，合起来就是Code。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode40.png)

以上是常量池中第九个元素的信息：01 00 04 43 6F 64 65，表示Utf8。

第九个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode41.png)

length：00 0F
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode42.png)

byte：4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65。length后面数15个字节，换算成10进制，表示76 105 110 101 78 117 109 98 101 114 84 97 98 108 101，每个字节对应的字符串分别为L i n e N u m b e r T a b l e，合起来就是LineNumberTable。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode43.png)

以上是常量池中第十个元素的信息：01 00 0F 4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65，表示Utf8。

