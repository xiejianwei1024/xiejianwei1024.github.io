---
layout: post
title: JVM类加载器学习第一篇
---

### 1 类的加载、连接与初始化  
#### 1.1 在Java的代码中，类型的加载、连接与初始化都是在程序运行期间完成的。 
*   加载：查找并加载类的二进制数据
*   连接  
    1）验证：确保被加载的类的正确性  
    2）准备：为类的静态变量分配内存，并将其初始化为默认值  
    3）解析：把类中的符号引用转换为直接引用
*   初始化：为类的静态变量赋予正确的初始值  

类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在内存中创建一个 java.lang.Class 对象（规范并未说明 Class 对象位于哪里，HotSpot 虚拟机将其放在了方法区中）来封装类在方法区内的数据结构。

#### 1.2 加载 .class 文件的方式  
*   从本地系统中直接加载
*   通过网络下载 .class 文件
*   从 zip, jar 等归档文件中加载 .class 文件
*   从专有数据库中提取 .class 文件
*   <font color="#FF0000">将 Java 源文件动态编译为 .class 文件</font>

#### 1.3 类加载器  
1.在如下几种情况下，Java虚拟机将结束生命周期。
*   执行了System.exit()
*   程序正常执行结束
*   程序在执行过程中遇到了异常或错误而异常终止
*   由于操作系统出现错误而导致Java虚拟机进程终止



#### 1.4 Java程序对类的使用方式分为两种：
*   主动使用
*   被动使用

所有的Java虚拟机实现必须在每个类或接口被“<font color="#FF0000">首次主动使用</font>”时才初始化他们。

#### 1.5 主动使用（七种）
*   创建类的实例
*   访问某个类或接口的静态变量，或者对该静态变量赋值 
*   调用类的静态方法
*   反射（Class.forName("java.lang.String")） 
*   初始化一个类的子类
*   Java虚拟机启动时被标明为启动类的类（Java Test） 
*   JDK1.7开始提供的动态语言支持：java.lang.invoke.MethodHandle 实例的结果 REF_getStatic，REF_putStatic，REF_invokeStatic 句柄对应的类没有初始化，则初始化

除了以上七种情况，其他使用类的方式都被看作是<font color="#FF0000">对类的被动使用，都不会导致类的初始化</font>。

#### 1.6 主动使用的第五种
初始化一个类的子类  
class Parent{}  
class Child extends Parent {}  
初始化 Child ，会主动使用 Parent

----------------------------------------
下面的程序演示主动使用：

```java
class MyParent1 {
    public static String str = "hello world";
    static {
        System.out.println("MyParent1 static block");
    }
}

class MyChild1 extends MyParent1 {
    public static String str2 = "welcome";
    static {
        System.out.println("MyChild1 static block");
    }
}

public class MyTest1 {
    public static void main(String[] args) {
        System.out.println(MyChild1.str);
    }
}
```
程序输出：  
MyParent1 static block  
hello world

现在程序改变一下，
```java
public class MyTest1 {
    public static void main(String[] args) {
        System.out.println(MyChild1.str2);
    }
}
```
程序输出：  
MyParent1 static block  
MyChild1 static block  
welcome

----------------------------------------

得出如下结论：  
<font color="#FF0000">
1.对于静态字段来说，只有直接定义该字段的类才会被初始化；<br/>
2.当初始化一个类时，要求其父类全部都初始化完毕了；
</font>

### 2.JVM 参数

-XX:+TraceClassLoading，用于追踪类的加载信息并打印出来

-XX:+<option\>，表示开启 option 选项  
-XX:-<option\>，表示关闭 option 选项  
-XX:<option\>=value，表示将 option 选项的值设置为 value


### 3. 常量池

下面的程序演示常量池：
```java
public class MyTest2 {
    public static void main(String[] args) {
        System.out.println(MyParent2.str);
    }
}

class MyParent2 {
    public static final String str = "hello world";
    static {
        System.out.println("MyParent2 static block");
    }
}
```

程序输出：  
hello world

----------------------------------------

得出如下结论：  
<font color="#FF0000">
1.常量在编译阶段会存入到调用该方法所在类的常量池中,本质上，调用类并没有直接引用到定义常量的类，因此并不会触发定义常量的类的初始化。<br/>
2.注意：这里指的是将常量存放在 MyTest2 的常量池中，之后 MyTest2 与 MyParent2 就没有任何关系了，甚至，我们可以将 MyParent2.class 文件删除。
</font>

### 4.助记符 
通过在终端反编译看到，反编译命令：
javap -c
G:\mytools\IdeaProjects\jvm_study\out\production\classes>javap -c com.shengsiyuan.classloader.MyTest2

 * ldc 表示将 int, float, String 类型的常量值从常量池中推送至栈顶
 * bitpush 表示将单字节（-128 ~ 127）的常量值推送至栈顶
 * sipush 表示将短整型（-32768 ~ 32767）的常量值推送至栈顶
 * iconst_1 表示将 int 类型 1 的常量值推送至栈顶（iconst_m1 ~ iconst_5）
 * getstatic [访问静态变量]
 * putstatic [赋值静态变量]
 * invokestatic [调用静态变量]
 * anewarray 表示创建一个引用类型（如类、接口、数组）的数组，并将其引用值压入栈顶
 * newarray 表示创建一个原始类型（如int、float、char）的数组，并将其引用值压入栈顶

 ### 5.运行期常量

下面的程序演示运行期常量：
```java
public class MyTest3 {
    public static void main(String[] args) {
        System.out.println(MyParent3.str);
    }
}

class MyParent3 {
    public static final String str = UUID.randomUUID().toString();
    static {
        System.out.println("MyParent3 static block");
    }
}
```

程序输出：  
MyParent3 static block  
81530983-df53-4922-a0c1-c1be640b39dd

----------------------------------------
得出如下结论：  
<font color="#FF0000">
1.当一个常量的值并非编译期间可以确定的，那么其值就不会放到调用类的常量池中，这时在程序运行时，会导致主动使用这个常量所在的类，显然会导致这个类被初始化。
</font>

### 6.数组实例类型
 ----------------------------------------

下面的程序演示运行期常量：
```java
public class MyTest4 {
    public static void main(String[] args) {
//        MyParent4 myParent4 = new MyParent4();
        MyParent4[] myParent4s = new MyParent4[1];
        System.out.println(myParent4s.getClass());
        MyParent4[][] myParent4s1 = new MyParent4[1][1];
        System.out.println(myParent4s1.getClass());

        System.out.println(myParent4s.getClass().getSuperclass());
        System.out.println(myParent4s1.getClass().getSuperclass());
        System.out.println("---");
        int[] ints = new int[1];
        System.out.println(ints.getClass());
        System.out.println(ints.getClass().getSuperclass());
        char[] chars = new char[1];
        System.out.println(chars.getClass());
        boolean[] booleans = new boolean[1];
        System.out.println(booleans.getClass());
    }
}
class MyParent4 {
    static {
        System.out.println("MyParent4 static block");
    }
}
```

程序输出：  
class [Lcom.shengsiyuan.classloader.MyParent4;<br/>
class [[Lcom.shengsiyuan.classloader.MyParent4;<br/>
class java.lang.Object<br/>
class java.lang.Object<br/>
---<br/>
class [I<br/>
class java.lang.Object<br/>
class [C<br/>
class [Z<br/>

----------------------------------------
得出如下结论：  
<font color="#FF0000">
1.对于数组实例来说，其类型是由 JVM 在运行期动态生成的，表示为 [Lcom.shengsiyuan.classloader.MyParent4 这种形式。<br/>
2.动态生成的类型，其父类型就是 Object。
</font>

----------------------------------------

Video 24<br/>
&emsp;&emsp;扩展类加载器和系统类加载器也是由启动类加载器所加载的。<br/>
&emsp;&emsp;System.out.println(Launcher.class.getClassLoader()); //null<br/>
&emsp;&emsp;分析：AppClassLoader 和 ExtClassLoader是Launcher的静态内部类，我们无法直接访问到；但是类加载器在加载类的时候有着这样的特性：类加载器加载Launcher的时候会加载里面的所有组件，当然也包括了AppClassLoader 和 ExtClassLoader。所有我们只要知道加载Launcher的是哪个类加载器，那么自然而然的，也就知道了加载AppClassLoader 和 ExtClassLoader是哪个类加载器。<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
Video 25<br/>
&emsp;&emsp;类加载器中一些重要的方法以及他们之间的一些联系。<br/>
&emsp;&emsp;第一个方法：`getSystemClassLoader()`<br/>
&emsp;&emsp;阅读javadoc：返回用于委托的系统类加载器。它是我们新创建的类加载器对象的默认委托双亲（也就是说调用这个静态方法所返回的ClassLoader实例是我们新建的自定义类加载器实例的委托双亲），通常地，它是用于启动应用的类加载器。<br/>
&emsp;&emsp;这个方法在运行时启动序列首先调用，在这个时刻点，它会创建系统类加载器，并且将该系统类加载器设置为调用`getSystemClassLoader()`方法的那个线程的上下文类加载器。<br/>
&emsp;&emsp;默认的系统类加载器时这个类的一个实现相关的实例。<br/>
&emsp;&emsp;当这个方法第一次被调用的时候，如果定义了系统属性`java.system.class.loader`，那么系统属性的值将作为一个类的名字，被当作系统类加载器返回（换句话说，系统属性`java.system.class.loader`所对应的值会作为返回的系统类加载器的名字）。使用默认的系统类加载器（AppClassLoader）加载这个类(系统属性`java.system.class.loader`的值肯定是一个类)，并且（我们自定义的类加载器）必须定义一个`public`的构造方法，接收单个`ClassLoader`类型的参数，这个参数用作自定义类加载器的委托双亲。接下来使用这个构造方法就会创建一个自定义类加载器的实例，这个构造方法接收一个默认的系统类加载器作为参数。新生成的类加载器就被定义成系统类加载器。也就是说用我们自定义的类加载器替换掉jdk自带的系统类加载器（AppClassLoader），作为系统中默认的类加载器。<br/>
&emsp;&emsp;openjdk.java.net<br/>
&emsp;&emsp;grepcode.com:浏览源代码的网站，搜索sun.misc.Lacuncher，然后分析源代码。<br/>
&emsp;&emsp;首先看构造方法：首先创造扩展类加载器，再进入扩展类加载器的源代码，创建扩展类加载器，读取特定目录下的class文件。<br/>
&emsp;&emsp;<br/>

at which point
在这个时刻点