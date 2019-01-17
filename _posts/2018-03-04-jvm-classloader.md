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
&emsp;&emsp;grepcode.com:浏览源代码的网站，搜索sun.misc.Lacuncher，然后分析源代码。http://www.docjar.com/html/api/sun/misc/Launcher.java.html<br/>
&emsp;&emsp;首先看构造方法：首先创造扩展类加载器，再进入扩展类加载器的源代码，创建扩展类加载器，读取特定目录下的class文件。<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
Video 26<br/>
&emsp;&emsp;2.创建AppClassLoader加载应用，将扩展类加载器extcl作为参数传递，看AppClassLoader的构造方法，extcl作为父加载器。<br/>
&emsp;&emsp;3.为当前线程设置上下文类加载器，也就是AppClassLoader。<br/>
&emsp;&emsp;`getSystemClassLoader()`中调用了`initSystemClassLoader()`，`initSystemClassLoader()`中有一句：`new SystemClassLoaderAction(scl)`，它的作用是处理用户设置系统属性`java.system.class.loader`为自定义类加载器的的这种情况，即使用用户自定义类加载器作为系统类加载器。<br/>
&emsp;&emsp;`D:\IdeaProjects\jvm_study\out\production\classes>java -Djava.system.class.loader=com.xjw.jvm.classloader.MyTest16 com.xjw.jvm.classloader.MyTest23`显示设置系统属性的系统类加载器为MyTest16，然后运行MyTest23<br/>
```java
class SystemClassLoaderAction implements PrivilegedExceptionAction<ClassLoader> {
    private ClassLoader parent;

    SystemClassLoaderAction(ClassLoader parent) {
        this.parent = parent;
    }

    public ClassLoader run() throws Exception {
        String cls = System.getProperty("java.system.class.loader");
        if (cls == null) {
            return parent;
        }

        Constructor<?> ctor = Class.forName(cls, true, parent)
            .getDeclaredConstructor(new Class<?>[] { ClassLoader.class });
        ClassLoader sys = (ClassLoader) ctor.newInstance(
            new Object[] { parent });
        Thread.currentThread().setContextClassLoader(sys);
        return sys;
    }
}
```
&emsp;&emsp;`forName(String name, boolean initialize, ClassLoader loader)`，<br/>
&emsp;&emsp;阅读javadoc：返回和给定字符串名字对应的类或者接口相关联的Class对象，用给定的类加载器去加载。给定一个类或者接口的完全限定名（以相同的格式由`getName`返回），这个方法尝试去定位、加载、链接这个类或者接口。给定的类加载器用于加载类或者接口。如果参数`loader`是`null`的话，类就会由启动类加载器加载。只有当参数`initialize`是`true`，并且这个还没有被初始化的时候，这个类才会被初始化。<br/>
&emsp;&emsp;如果参数`name`表示一个原生类型或者`void`，将会尝试在为命名的包中定位由用户定义的class。因此，这个方法不能获得代表原生类型或者`void`的Class对象。<br/>
&emsp;&emsp;如果参数`name`表示一个数组类，数组类的组件类型会被加载，但是不会被初始化。<br/>
&emsp;&emsp;例如，在一个实例方法中，下面的表达式：`Class.forName("Foo")`，等价于`Class.forName("Foo", true, this.getClass().getClassLoader())`。<br/>
&emsp;&emsp;分析源代码：`run()`，首先获取`java.system.class.loader`系统属性的值字符串cls。如果cls等于null，说明用户没有将系统类加载器设置为自定义类加载器，直接返回parent，即系统默认的系统类加载器，也就是AppClassLoader。如果cls不等于null，`Class.forName(cls, true, parent)`表明，加载的是cls，即用户将系统类加载器设置为自定义类加载器，使用parent，即AppClassLoader类加载器加载，加载完成后进行初始化。<br/>
Video 27<br/>
&emsp;&emsp;线程上下文类加载器，双亲委托模型在某些情况下无法满足我们的需求。<br/>
&emsp;&emsp;当前类加载器（Current Classloader）<br/>
&emsp;&emsp;每个类都会使用自己的类加载器（即加载自身的类加载器）来去加载器其他类（指的是所依赖的类），例：如果ClassX引用了ClassY，那么ClassX的类加载器就会去加载ClassY（前提是ClassY尚未被加载）。<br/>
&emsp;&emsp;线程上下文类加载器（Context Classloader）<br/>
&emsp;&emsp;线程上下文类加载器是从JDK 1.2开始引入的，类Thread中的getContextClassLoader()与setContextClassLoader(ClassLoader cl)分别用来获取和设置上下文类加载器。<br/>
&emsp;&emsp;如果没有通过setContextClassLoader(ClassLoader cl)进行设置的话，线程将继承其父线程的上下文类加载器。Java应用运行时的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过改类加载器来加载类和资源。<br/>
&emsp;&emsp;线程上下文类加载器的重要性：<br/>
&emsp;&emsp;SPI（Service Provider Interface）<br/>
&emsp;&emsp;父ClassLoader可以使用当前线程Thread.currentThread().getContextClassLoader()所指定的ClassLoader加载类。这就改变了父ClassLoader不能使用子ClassLoader或是其他没有直接父子关系的ClassLoader加载的类的情况，即改变了双亲委派模型。<br/>
&emsp;&emsp;线程上下文类加载器就是当前线程的Current Classloader。<br/>
&emsp;&emsp;在双亲委托模型下，类加载是由下至上的，即下层的类加载器会委托上层进行加载。但是对于SPI来说，有些接口是Java核心库所提供的，而Java核心库是由启动类加载器来加载的，而这些接口的实现却来自于不同的jar包（厂商提供），Java的启动类加载器是不会加载其他来源的jar包，这样传统的双亲委托模型无法满足SPI的要求。而通过给当前线程设置上下文类加载器，就可以由设置的上下文类加载器来实现对于接口实现类的加载。<br/>
Video 28<br/>
&emsp;&emsp;在MyTest25.java中，打印出当前线程的上下文类加载器是AppClassLoader。原因在sun.misc.Launcher.java的构造方法中，在创建完成AppClassLoader后，立即将当前线程的上下文类加载器设置为AppClassLoader。<br/>
&emsp;&emsp;当高层提供了统一的接口让低层去实现，同时又要在高层加载（或实例化）低层的类时，就必须要通过线程上线问类加载器来帮助高层的ClassLoaded找到并加载该类。<br/>
&emsp;&emsp;如果我们没有对线程上下文类加载设置的话，那么当前线程的上下文类加载器就是系统类加载器。由于它是在运行期间被放置到了当前线程当中，不管在什么环境下（处于启动类），都可以通过Thread.currentThread().getContextClassLoader()来获取到AppClassLoader进行操作。<br/>
Video 29<br/>
&emsp;&emsp;`java.util.ServiceLoader`：服务加载器（服务提供者框架 service provider framework）<br/>
&emsp;&emsp;阅读javadoc：<br/>
&emsp;&emsp;这是一个简单的服务提供者加载设施。<br/>
&emsp;&emsp;服务是一个已知接口和抽象类的集合。服务提供者是服务的一个具体实现。提供者中的类实现了服务中的接口并且继承了服务中的类。服务提供者可以通过扩展的形式安装在Java平台，以jar文件的形式放置到常规的扩展目录下面。（MyTest26.java这个例子中就是把mysql的jar文件放到类路径下）提供者还可以通过将他们添加到应用的classpath获得，或者通过一些平台相关的方式。<br/>
&emsp;&emsp;出于加载的目的，服务由单个的类型表示，那就是单个的接口或抽象类。（具体的类是可以被使用的，但是不推荐使用抽象类。）给定服务的提供者包含一个或多个具体的类，用的是指定的提供者的数据和代码扩展服务类型。提供者类通常不是整个提供者本身，而是一个代理，它包含足够的信息来决定提供者是否能够满足特定的请求，以及根据需要的代码创建实际提供者。提供者类的细节是与服务高度相关的，没有单个的类或者接口能够统一他们，因此没有在这里定义这种类型。刺蛇是强制的惟一要求是提供者类必须具有零参数构造函数，以便在加载过程中实例化它们。<br/>
&emsp;&emsp;服务提供者是通过将提供者配置文件放置到资源目录META-INF/services下识别的。文件的名字是服务类型的完全限定二进制名字。这个文件包含了一个列表，提供者具体类的完全限定二进制名字这样的一个列表，每行放置一个。空格和tab键包围着每一个名字，和空行一样，可以忽略。注释字符是`#`；在每一行跟在第一个注释字符后面的所有字符都会被忽略掉。文件必须用`UTF-8`进行编码。<br/>
&emsp;&emsp;如果一个具体的提供者类在多个配置文件中都被命名了，或者在一个配置文件中命名了多次，那么重复的就被忽略了。命名了特定提供者的配置文件不需要与提供者本身位于相同的jar文件或其他分发单元中。提供者必须是可以访问的，从最初查询定位配置文件的类加载器。请注意，这并不一定是实际加载文件的类加载器。<br/>
&emsp;&emsp;提供者被定位和实例化，都是延时的，意思就是按需的。服务加载器维护了一个关于已经被加载了的提供者的缓存。每次对`iterator`方法的调用都会返回一个迭代器，这个迭代器会首先以实例化的顺序获取缓存里面的所有元素,然后延时定位和实例化剩余的提供者，然后按照顺序将每一个都添加到缓存里面。缓存可以通过`reload`方法清空。<br/>
&emsp;&emsp;使用注意：如果用于加载提供者的类加载器的类路径包含远程网络url，那么这些url将在搜索提供者配置文件的过程中被解除引用。<br/>
&emsp;&emsp;此活动是正常的，尽管它可能会在web服务器日志中创建令人费解的条目。但是，如果没有正确配置web服务器，则此活动可能导致提供者加载算法错误地失败。<br/>
&emsp;&emsp;当一个请求资源不存在的时候，web服服务器应该返回一个HTTP 404 (Not Found) 响应。<br/>
&emsp;&emsp;泛型S：被加载器所加载的服务类型<br/>
Video 30<br/>
&emsp;&emsp;MyTest26.java中第一行调用了ServiceLoader的load方法。跟进load方法，阅读javadoc:<br/>
&emsp;&emsp;为给定的服务类型创建一个新的服务类加载器，使用的是当前线程的上下文类加载器。<br/>
```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}
```
&emsp;&emsp;分析：这里使用当前线程上下文类加载器加载的原因是：MyTest26.java中第一行调用了ServiceLoader的load方法。MyTest26会由系统类加载器加载，ServiceLoader也会交给系统类加载器加载，根据类加载器的双亲委托机制，会一直向上追溯到启动类加载器，而ServiceLoader是JDK核心库下的class文件，启动类加载器可以加载ServiceLoader，那么load方法里面的类也都交由启动类加载器加载，显然，mysql驱动位于当前类路径下，启动类加载器是不能加载的。服务提供者（mysql）位于当前类路径下，而当前线程上下文类加载器即系统类加载器加载的就是classpath下的jar。<br/>
&emsp;&emsp;第二行调用了ServiceLoader的iterator方法。跟进iterator方法，阅读javadoc:<br/>
&emsp;&emsp;延迟加载可用的提供者，它是此加载器所加载服务的提供者。此方法返回的迭代器首先按实例化的顺序生成提供者缓存的所有元素。然后，它延迟加载并实例化任何剩余的提供者，依次将每个提供者添加到缓存中。<br/>
Video 31<br/>
```java
private static boolean isDriverAllowed(Driver driver, ClassLoader classLoader) {
    boolean result = false;
    if(driver != null) {
        Class<?> aClass = null;
        try {
            aClass =  Class.forName(driver.getClass().getName(), true, classLoader);
        } catch (Exception ex) {
            result = false;
        }

            result = ( aClass == driver.getClass() ) ? true : false;
    }

    return result;
}
```
&emsp;&emsp;分析：DriverManager中的这个方法特别重要，尤其是这一行`result = ( aClass == driver.getClass() ) ? true : false;` class对象比较的意义就涉及到命名空间的问题。参数 driver 之前被加载和接下来要使用的时候被加载器的类加载器应该是同一个。如果不是同一个类加载器加载的话，后面使用的时候一定会抛出ClassCastException。两个类即便名字是一摸一样的话，但是处于不同的命名空间中，相互是转化不了的。只有位于同一个命名空间下的class对象，它们之间才是可以相互转换的。因为之前的加载时 通过SPI（ServiceLoader）这样的机制进行的，对于开发者来说，可以轻松地通过Thread.currentThread.setContextClassLoader(cl)这样的方式将当前线程的上下文类加载器给改变了。改变了的话，就有可能会导致不是相同的类加载器进行加载的。<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>

at which point
在这个时刻点

privilege /ˈprɪvəlɪdʒ/
n.权限；特权；

general-purpose
adj. 多用途的；一般用途的
