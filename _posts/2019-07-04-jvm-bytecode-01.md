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

byte：3C 69 6E  69 74 3E。length后面数6个字节，换算成10进制，表示60 105 110 105 116 52，每个字节对应的字符串分别为< i n i t >，合起来就是`<init>`。长度为length的UTF-8编码的字符串。

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

第十个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode41.png)

length：00 0F
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode42.png)

byte：4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65。length后面数15个字节，换算成10进制，表示76 105 110 101 78 117 109 98 101 114 84 97 98 108 101，每个字节对应的字符串分别为L i n e N u m b e r T a b l e，合起来就是LineNumberTable。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode43.png)

以上是常量池中第十个元素的信息：01 00 0F 4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65，表示Utf8。

第11个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode44.png)

length：00 12
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode45.png)

byte：4C 6F 63 61 6C 56 61 72 69 61 62 6C 65 54 61 62 6C 65。length后面数18个字节，换算成10进制，表示76 111 99 97 108 86 97 114 105 97 98 108 101 84 97 98 108 101，每个字节对应的字符串分别为L o c a l V a r i a b l e T a b l e，合起来就是LocalVariableTable。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode46.png)

以上是常量池中第11个元素的信息：01 00 12 4C 6F 63 61 6C 56 61 72 69 61 62  6C 65 54 61 62 6C 65，表示Utf8。

第12个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode047.png)

length：00 04
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode048.png)

byte：74 68 69 73。length后面数4个字节，换算成10进制，表示116 104 105 115，每个字节对应的字符串分别为t h i s，合起来就是this。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode049.png)

以上是常量池中第12个元素的信息：01 00 04 74 68 69 73，表示Utf8。

第13个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode47.png)

length：00 1A
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode48.png)

byte：4C 63 6F 6D 2F 6A 76 6D 2F 62 79 74 65 63 6F 64 65 2F 4D 79 54 65 73 74 31 3B。length后面数26个字节，换算成10进制，表示76 99 111 109 47 106 118 109 47 98 121 116 101 99 111 100 101 47 77 121 84 101 115 116 49 59，每个字节对应的字符串分别为L c o m / j v m / b y t e c o d r / M y T e s t 1 ;，合起来就是Lcom/jvm/bytecode/MyTest1;。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode49.png)

以上是常量池中第13个元素的信息：01 00 1A 4C 63 6F 6D 2F 6A 76 6D 2F 62 79 74 65 63 6F 64 65 2F 4D 79 54 65 73 74 31 3B，表示Utf8。

第14个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode50.png)

length：00 04
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode51.png)

byte：67 65 74 41。length后面数4个字节，换算成10进制，表示103 101 116 65，每个字节对应的字符串分别为g e t A，合起来就是getA。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode52.png)

以上是常量池中第14个元素的信息：01 00 04 67 65 74 41，表示Utf8。

第15个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode53.png)

length：00 03
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode54.png)

byte：28 29 49。length后面数3个字节，换算成10进制，表示40 41 73，每个字节对应的字符串分别为( ) I，合起来就是()I。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode55.png)

以上是常量池中第15个元素的信息：01 00 03 28 29 49，表示Utf8。

第16个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode56.png)

length：00 04
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode57.png)

byte：73 65 74 41。length后面数4个字节，换算成10进制，表示115 101 116 65，每个字节对应的字符串分别为s e t A，合起来就是setA。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode58.png)

以上是常量池中第16个元素的信息：01 00 04 73 65 74 41，表示Utf8。

第17个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode59.png)

length：00 04
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode60.png)

byte：28 49 29 56。length后面数4个字节，换算成10进制，表示40 73 41 86，每个字节对应的字符串分别为( I ) V，合起来就是(I)V。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode61.png)

以上是常量池中第17个元素的信息：01 00 04 28 49 29 56，表示Utf8。

第18个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode62.png)

length：00 0A
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode63.png)

byte：53 6F 75 72 63 65 46 69 6C 65。length后面数10个字节，换算成10进制，表示83 111 117 114 99 101 70 73 108 101，每个字节对应的字符串分别为S o u r c e F i l e，合起来就是SourceFile。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode64.png)

以上是常量池中第18个元素的信息：01 00 0A 53 6F 75 72 63 65 46 69 6C 65，表示Utf8。

第19个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode65.png)

length：00 0C
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode66.png)

byte：4D 79 54 65 73 74 31 2E 6A 61 76 61。length后面数12个字节，换算成10进制，表示77 121 84 101 115 116 49 46 106 97 118 97，每个字节对应的字符串分别为M y T e s t 1 . j a v a，合起来就是MyTest1.java。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode67.png)

以上是常量池中第19个元素的信息：01 00 0C 4D 79 54 65 73 74 31 2E 6A 61 76 61，表示Utf8。

第20个元素的第一个数据是：0C。去表格中找u1类型，值为12的，NameAndType。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode68.png)

下图值为12的常量：NameAndType 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode69.png)

CONSTANT_NameAndType_Info：该常量包含三部分， tag,index,index。我们已经知道tag的值。 第一个index：00 07，指向该字段或方法名称常量项的索引。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode70.png)

第二个index：00 08。指向该字段或方法描述符常量项的索引。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode71.png)

以上是常量池中第20个元素的信息：0C 00 07 00 08，表示NameAndType。

第21个元素的第一个数据是：0C。去表格中找u1类型，值为12的，NameAndType。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode72.png)

下图值为12的常量：NameAndType 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode69.png)

CONSTANT_NameAndType_Info：该常量包含三部分， tag,index,index。我们已经知道tag的值。 第一个index：00 05，指向该字段或方法名称常量项的索引。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode73.png)

第二个index：00 06。指向该字段或方法描述符常量项的索引。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode74.png)

以上是常量池中第21个元素的信息：0C 00 05 00 06，表示NameAndType。

第22个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode75.png)

length：00 18
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode76.png)

byte：63 6F 6D 2F 6A 76 6D 2F 62 79 74 65 63 6F 64 65 2F 4D 79 54 65 73 74 31。length后面数24个字节，换算成10进制，表示99 111 109 47 106 118 109 47 98 121 116 101 99 111 100 101 47 77 121 84 101 115 116 49，每个字节对应的字符串分别为c o m / j v m / b y t e c o d e / M y T e st 1，合起来就是com/jvm/bytecode/MyTest1。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode77.png)

以上是常量池中第22个元素的信息：01 00 18 63 6F 6D 2F 6A 76 6D 2F 62 79 74 65 63 6F 64 65 2F 4D 79 54 65 73 74 31，表示Utf8。

第23个元素的第一个数据是：01。去表格中找u1类型，值为1的，Utf8。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode78.png)

length：00 10
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode79.png)

byte：6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 65 63 74 。length后面数16个字节，换算成10进制，表示106 97 118 97 47 108 97 110 103 47 79 98 106 101 99 116，每个字节对应的字符串分别为j a v a / l a n g / O b j e c t，合起来就是java/lang/Object。长度为length的UTF-8编码的字符串。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode80.png)

以上是常量池中第23个元素的信息：01 00 10 6A 61 76 61 2F 6C  61 6E 67 2F 4F 62 6A 65 63 74，表示Utf8。

以上是常量池的信息。

根据java字节码组成结构图，接下来的两个字节是U2类型的，access_flags(类的访问控制权限)。
访问标志信息包括该Class文件是类还是接口，是否被定义成public，是否是abstract，如果是类，是否被声明成final。通过上面的源代码，我们知道该文件是类并且是public。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode81.png)

对应到MyTest1.class中的是：00 21，是0x0020和0x0001的并集，表示ACC_PUBLIC, ACC_SUPER。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode82.png)

根据java字节码组成结构图，接下来的两个字节是U2类型的，this_class(类名)。<br/>
类索引：00 03。com/shengsiyuan/jvm/bytecode/MyTest1（This Class Name）
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode83.png)

根据java字节码组成结构图，接下来的两个字节是U2类型的，super_class(父类名)。<br/>
父类索引：00 04。java/lang/Object（Super Class Name）
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode84.png)

根据java字节码组成结构图，接下来的两个字节是U2类型的，interface_count(接口个数)。<br/>
接口索引：00 00。0（接口个数为0，说明当前类没有实现接口。所以接口名不会出现在字节码文件中。如果接口数大于等于1，那么接口名会出现在字节码文件中，并且占两个字节。）
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode85.png)

根据java字节码组成结构图，接下来的两个字节是U2类型的， (fields_count)域个数，
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode086.png)

根据java字节码组成结构图，接下来的是 (fields)域的表，它遵循如下的结构：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode86.png)

字段表的access_flags，字段访问标志：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode87.png)

字段表用于描述类和接口中声明的变量。这里的字段包含了类级别变量以及实例变量，但是不包括方法内部声明的局部变量。

access_flags，前两个字节00 02，去字段访问标志中找到对应的值：ACC_PRIVATE，字段是否private。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode88.png)

name_index，00 05，去常量池中找索引值为5的，表示a。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode89.png)

descriptor_index，00 06，去常量池中找索引值为6的，表示I。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode90.png)

attributes_count，00 00。attributes_count为0，attribute_info就不会出现在字节码文件中了。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode91.png)

以上是字段表的信息。

接下来的两个字节是方法的个数，00 03，表示有三个方法。
attributes_count，00 00。attributes_count为0，attribute_info就不会出现在字节码文件中了。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode92.png)

下面是方法表的结构：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode93.png)

下面是方法访问标志，access_flags的表：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode94.png)

access_flags，00 01，表示ACC_PUBLIC，方法是否为public。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode95.png)

name_index，00 07，去常量池中找索引值为7的，表示`<init>`。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode96.png)

descriptor_index，00 08，去常量池中找索引值为8的，表示()V。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode97.png)

attributes_count，00 01，表示该方法的属性结构数量是1。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode98.png)

attribute_info的结构如下：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode99.png)

attribute_name_index，00 09，去常量池中寻找索引值为9的，表示Code。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode100.png)

attribute_length：00 00 00 38，转换成10进制，表示56，那么数56个字节。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode101.png)

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode1011.png)

info[attribute_length]：又是一个结构体。<br/>
所有Code属性表如下，它包含了attribute_name_index和attribute_length：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode102.png)

结构体为：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode103.png)

先给出各个属性的意思：
*   attribute_length表示attribute所包含的字节数，不包含attribute_name_index和attribute_length字段。
*   max_stack表示这个方法运行的任何时刻所能达到的操作数栈的最大深度。
*   max_locals表示方法执行期间创建的局部变量的数目，包含用来表示传入的参数的局部变量。
*   code_length表示该方法所包含的字节码的字节数以及具体的指令码。具体字节码是该方法调用时，虚拟机所执行的字节码。
*   exception_table这里存放的是处理异常的信息。每个exception_table表项有start_pc，end_pc，handler_pc，catch_type组成。
*   start_pc和end_pc表示在Code数组中的从start_pc到end_pc处（包含start_pc，不包含end_pc）的指令抛出的异常会由这个表项来处理。
*   handler_pc表示处理异常的代码的开始处。
*   catch_type表示会被处理的异常类型，它指向常量池里的一个异常类。当catch_type为0时，表示处理所有的异常。

max_stack：00 02
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode104.png)

max_locals：00 01
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode105.png)

code_length：00 00 00 0A，转换成10进制表示10，后面继续数10个字节。这10个字节是具体字节码，即该方法被调用时，虚拟机所执行的字节码。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode106.png)

0x2A B7 00 01 2A 04 B5 00 02 B1：具体字节码，即该方法被调用时，虚拟机所执行的字节码。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode107.png)

那么如何确定这些执行的字节码是什么意思呢？

助记符与字节码文件中16进制字节相对应。

字节码查看工具<br/>
jclasslib<br/>
https://github.com/ingokege/jclasslib<br/>
IDEA中安装File->Settings,搜索jclasslib，重启IDEA，在view中查找jclasslib<br/>

https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.aload

助记符与16进制对应表。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode108.png)

aload_0 = 42 (0x2a) 从局部变量加载引用，压入栈顶，即准备执行方法。

invokespecial = 183 (0xb7) 调用父类相关的方法

invokespecial #1

`#1`是参数，对应字节码文件中0xB7后面的两个字节，即0x0001，去常量池中寻找索引为1的常量。那么整个命令的意思就是调用父类的构造方法。

aload_0 = 42 (0x2a) 又是将操作数压入栈顶

iconst_1 = 4 (0x4) 

putfield = 181 (0xb5) 

putfield #2

`#2`是参数，对应字节码文件中0xB5后面的两个字节，即0x0002，去常量池中寻找索引为2的常量。给成员变量赋值，值就是上面压入栈顶的1。

return = 177 (0xb1) 方法返回 void，构造方法没有返回值。

以上是Code属性表中code的信息。<br/>
接下来的两个字节0x 00 00，表示exception_table_length，，异常表长度为0，下面有关异常的就不用看了。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode109.png)

00 02，表示attributes_count，表示构造方法存在两个属性。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode110.png)

attribute_info属性信息又是一个结构体。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode111.png)

attribute_name_index：00 0A，换算成10进制是10，去常量池中找索引为10的，LineNumberTable。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode112.png)

attribute_length：00 00 00 0A，换算成10进制是10，属性长度为10，往后面数10个字节，0x00 02 00 00 00 03 00 04  00 04，这10个字节是两两对应的，前两个字节00 02表示有两对，为00 00 00 03 和 00 04 00 04。

第1对：00 00 00 03，表示行号为3，其中前两个字节表示偏移量是0，后两个字节表示行号是3。

第2对：00 04 00 04，表示行号为4，其中前两个字节表示偏移量是4，后两个字节表示行号是4。

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode113.png)

行号表的jclasslib查看：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode114.png)


以上是构造方法中第一个属性的信息。

下面看第二个属性的信息：

attribute_name_index：00 0B，换算成10进制是11，去常量池中找索引为11的，LocalVariableTable。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode115.png)

attribute_length：00 00 00 0C，换算成10进制是12，属性长度为12，往后面数12个字节，0x00 01 00 00 00 0A 00 0C 00 0D 00 00，这10个字节是两两对应的，前两个字节00 01表示局部变量的个数为1。再后面4个字节0x00 00 00 0A，表示开始位置是0，结束位置是10。0x00表示局部变量的索引是0。0x0C表示局部变量对应常量池中的位置是12，this，表示当前对象。0x000D，表示对局部变量的描述，对应常量池中的位置是13。最后两个字节0x0000，表示做校验检查的。

this,在Java中，每个方法都可以访问this，this表示对当前对象的引用。
在字节码中，如果一个方法是非静态的，是实例方法的话，this是作为方法的第一个参数隐式传递进来。即Java中的任意一个实例方法中，都至少有一个局部变量，this，表示对当前对象的引用。

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode116.png)

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode117.png)

局部变量表的jclasslib查看：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode118.png)


以上是第一个方法的信息，

下面是第二个方法的信息：

access_flags，00 01，表示ACC_PUBLIC，方法是否为public。 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode119.png)

name_index，00 0E，去常量池中找索引值为14的，表示`getA`。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode120.png)

descriptor_index，00 0F，去常量池中找索引值为15的，表示`()I`。 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode121.png)

attributes_count，00 01，表示该方法的属性结构数量是1。 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode122.png)

attribute_info的信息如下：

attribute_name_index，00 09，去常量池中寻找索引值为9的，表示Code。 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode123.png)

attribute_length：00 00 00 2F，转换成10进制，表示47，那么数47个字节。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode124.png)

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode125.png)

info[attribute_length]：又是一个结构体，即Code属性表信息，它包含了attribute_name_index和attribute_length。

max_stack：00 01
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode126.png)

max_locals：00 01 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode127.png)

code_length：00 00 00 05，表示5，后面继续数5个字节。这5个字节是具体字节码，即该方法被调用时，虚拟机所执行的字节码。 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode128.png)

2A B4 00 02 AC ：具体字节码，即该方法被调用时，虚拟机所执行的字节码。 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode129.png)

此时，需要借助jclasslib，观察助记符与16进制对应表。 

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode130.png)


aload_0 = 42 (0x2a) 从局部变量加载引用，压入栈顶，即准备执行方法。

getfield = 180 (0xb4)

getfield #2

`#2`是参数，对应字节码文件中0xB5后面的两个字节，即0x0002，去常量池中寻找索引为2的常量。

ireturn = 172 (0xac) 从方法中返回一个int。

以上是Code属性表中code的信息。

接下来的两个字节0x 00 00，表示exception_table_length，，异常表长度为0，下面有关异常的就不用看了。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode131.png)

00 02，表示attributes_count，表示构造方法存在两个属性。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode132.png)

attribute_info属性信息又是一个结构体。 

attribute_name_index：00 0A，换算成10进制是10，去常量池中找索引为10的，LineNumberTable。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode133.png)

attribute_length：00 00 00 06，属性长度为6，往后面数6个字节，即0x00 01 00 00 00 07，这6个字节是两两对应的，前两个字节0x0001表示有一对，为0x00 00 00 07：前两个字节0x0000表示偏移量为0，后两个字节0x0007表示行号为7。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode134.png)

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode135.png)

行号表的jclasslib查看： 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode136.png)

以上是构造方法中第一个属性的信息。

下面看第二个属性的信息：

attribute_name_index：00 0B，换算成10进制是11，去常量池中找索引为11的，LocalVariableTable。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode137.png)

attribute_length：00 00 00 0C，换算成10进制是12，属性长度为12，往后面数12个字节，0x00 01 00 00 00 0A 00 0C 00 0D 00 00，这10个字节是两两对应的，前两个字节00 01表示局部变量的个数为1。再后面4个字节0x00 00 00 0A，表示开始位置是0，结束位置是10。0x00表示局部变量的索引是0。0x0C表示局部变量对应常量池中的位置是12，this，表示当前对象。0x000D，表示对局部变量的描述，对应常量池中的位置是13。最后两个字节0x0000，表示做校验检查的。

this,在Java中，每个方法都可以访问this，this表示对当前对象的引用。 在字节码中，如果一个方法是非静态的，是实例方法的话，this是作为方法的第一个参数隐式传递进来。即Java中的任意一个实例方法中，都至少有一个局部变量，this，表示对当前对象的引用。

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode138.png)

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode139.png)

局部变量表的jclasslib查看： 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode140.png)

以上是第二个方法的信息，

下面是第三个方法的信息：

access_flags，00 01，表示ACC_PUBLIC，方法是否为public。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode141.png)

name_index，00 10，去常量池中找索引值为16的，表示`setA`。 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode142.png)

descriptor_index，00 11，去常量池中找索引值为17的，表示`(I)V`。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode143.png)

attributes_count，00 01，表示该方法的属性结构数量是1。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode144.png)

attribute_info的信息如下：

attribute_name_index，00 09，去常量池中寻找索引值为9的，表示Code。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode145.png)

attribute_length：00 00 00 3E，转换成10进制，表示52，那么数52个字节。 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode146.png)

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode147.png)

info[attribute_length]：又是一个结构体，即Code属性表信息，它包含了attribute_name_index和attribute_length。

max_stack：00 02 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode148.png)

max_locals：00 02 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode149.png)

code_length：00 00 00 06，表示6，后面继续数6个字节。这6个字节是具体字节码，即该方法被调用时，虚拟机所执行的字节码。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode150.png)

2A 1B B5 00 02 B1 ：具体字节码，即该方法被调用时，虚拟机所执行的字节码。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode151.png)

此时，需要借助jclasslib，观察助记符与16进制对应表。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode152.png)

aload_0 = 42 (0x2a) 从局部变量加载引用，压入栈顶，即准备执行方法。

iload_1 = 27 (0x1b) 从局部变量中加载int，第一个索引对应的int值。

putfield = 181 (0xb5) 设置成员变量的值。

putfield #2

`#2`是参数，对应字节码文件中0xB5后面的两个字节，即0x0002，去常量池中寻找索引为2的常量。给成员变量赋值，值就是上面的第一个索引对应的int值。

return = 177 (0xb1) 方法返回void。

以上是Code属性表中code的信息。

接下来的两个字节0x 00 00，表示exception_table_length，，异常表长度为0，下面有关异常的就不用看了。 

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode153.png)

00 02，表示attributes_count，表示构造方法存在两个属性。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode154.png)

attribute_info属性信息又是一个结构体。

attribute_name_index：00 0A，换算成10进制是10，去常量池中找索引为10的，LineNumberTable。
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode155.png)

attribute_length：00 00 00 0A，换算成10进制是10，属性长度为10，往后面数10个字节，0x00 02 00 00 00 0B 00 05 00 0C ，这10个字节是两两对应的，前两个字节00 02表示有两对，为00 00 00 0B 和 00 05 00 0C。 

第1对：00 00 00 0B，表示行号为11，其中前两个字节表示偏移量是0，后两个字节表示行号是11。 

第2对：00 05 00 0C，表示行号为4，其中前两个字节表示偏移量是5，后两个字节表示行号是12。 

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode156.png)

行号表的jclasslib查看：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode157.png)

以上是构造方法中第一个属性的信息。

下面看第二个属性的信息：

attribute_name_index：00 0B，换算成10进制是11，去常量池中找索引为11的，LocalVariableTable。 
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode158.png)

attribute_length：00 00 00 16，换算成10进制是22，属性长度为22，往后面数22个字节，即
0x00 02 00 00 00 06 00 0C 00 0D 00 00 00 00 00 06 00 05 00 06 00 01。这22个字节是两两对应的，前两个字节0x0002表示局部变量个数为2，再后面四个字节0x00 00 00 06，表示开始位置是0，结束位置是6。<br/>
0x00表示局部变量的索引是0。0x0C表示局部变量对应常量池中的位置是12，this，表示当前对象。0x00 0D，表示对局部变量的描述，对应常量池中的位置是13。最后两个字节0x00 00，表示做校验检查的。<br/>
再后面四个字节0x00 00 00 06，表示开始位置是0，结束位置是6。0x00表示局部变量的索引是0。0x05表示表示局部变量对应常量池中的位置是6，a。0x00 06表示，对局部变量的描述，对应常量池中的位置是6，表示int类型。最后两个字节0x00 01，表示做校验检查的。

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode159.png)

![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode160.png)

局部变量表的jclasslib查看：
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode161.png)

后面的字节属于当前字节码的属性，0x 00 01 00 12 00 00 00 02 00 13
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/jvm/bytecode162.png)

0x0001，表示属性的个数为1，<br/>
0x0012，表示属性名称的索引为18，对应常量池中SourceFile，<br/>
0x0002，表示属性名称的长度为2个字节，<br/>
0x0013，表示源文件真正的值13，对应常量池中MyTest1.java<br/>
