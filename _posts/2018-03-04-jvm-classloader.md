---
layout: post
title: JVM类加载器
---

### 类的加载、连接与初始化  
1.在Java的代码中，类型的加载、连接与初始化都是在程序运行期间完成的。 
*   加载：查找并加载类的二进制数据
*   连接  
    1）验证：确保被加载的类的正确性  
    2）准备：为类的静态变量分配内存，并将其初始化为默认值  
    3）解析：把类中的符号引用转换为直接引用
*   初始化：为类的静态变量赋予正确的初始值  

类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在内存中创建一个 java.lang.Class 对象（规范并未说明 Class 对象位于哪里，HotSpot 虚拟机将其放在了方法区中）来封装类在方法区内的数据结构。

加载 .class 文件的方式  
*   从本地系统中直接加载
*   通过网络下载 .class 文件
*   从 zip, jar 等归档文件中加载 .class 文件
*   从专有数据库中提取 .class 文件
*   <font color="#FF0000">将 Java 源文件动态编译为 .class 文件</font>

类加载器
1.在如下几种情况下，Java虚拟机将结束生命周期。
*   执行了System.exit()
*   程序正常执行结束
*   程序在执行过程中遇到了异常或错误而异常终止
*   由于操作系统出现错误而导致Java虚拟机进程终止



Java程序对类的使用方式分为两种：
*   主动使用
*   被动使用

所有的Java虚拟机实现必须在每个类或接口被“<font color="#FF0000">首次主动使用</font>”时才初始化他们。

主动使用（七种）
*   创建类的实例
*   访问某个类或接口的静态变量，或者对该静态变量赋值 
*   调用类的静态方法
*   反射（Class.forName("java.lang.String")） 
*   初始化一个类的子类
*   Java虚拟机启动时被标明为启动类的类（Java Test） 
*   JDK1.7开始提供的动态语言支持：java.lang.invoke.MethodHandle 实例的结果 REF_getStatic，REF_putStatic，REF_invokeStatic 句柄对应的类没有初始化，则初始化

除了以上七种情况，其他使用类的方式都被看作是<font color="#FF0000">对类的被动使用，都不会导致类的初始化</font>。


助记符

getstatic[访问静态变量]  
putstatic[赋值静态变量]  
invokestatic[调用静态变量]  


初始化一个类的子类  
class Parent{}  
class Child extends Parent {}  
初始化 Child ，会主动使用 Parent