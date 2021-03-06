---
layout: post
title: Java并发|Thread与Runnable
---
[Lesson: Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/)

## 进程和线程（Processes and Threads）
&emsp;&emsp;在并发编程中，有两个基本的执行单元:进程和线程。在Java编程语言中，并发编程主要关注线程。然而，进程也很重要。<br/>

&emsp;&emsp;计算机系统通常有许多活动进程和线程。即使在只有一个执行核心的系统中也是如此，因此在任何给定时刻只有一个线程实际执行。单个内核的处理时间通过一个称为时间切片的OS特性在进程和线程之间共享。<br/>

&emsp;&emsp;计算机系统拥有多个处理器或多个执行核心的处理器越来越普遍。这极大地增强了系统并发执行进程和线程的能力——但是即使在没有多个处理器或执行核心的简单系统上，并发也是可能的。<br/>

### 进程（Processes）
&emsp;&emsp;进程具有自包含的执行环境。进程通常具有一组完整的、私有的基本运行时资源;特别是，每个进程都有自己的内存空间。<br/>

&emsp;&emsp;进程通常被视为程序或应用程序的同义词。然而，用户所看到的单个应用程序实际上可能是一组协作进程。为了促进进程之间的通信，大多数操作系统都支持进程间通信(IPC)资源，比如管道和套接字。IPC不仅用于同一系统上的进程之间的通信，而且用于不同系统上的进程之间的通信。<br/>

&emsp;&emsp;Java虚拟机的大多数实现都作为单个进程运行。Java应用程序可以使用`ProcessBuilder`对象创建其他进程。多进程应用程序超出了本教程的范围。<br/>

### 线程（Threads）
&emsp;&emsp;线程有时被称为轻量级进程。进程和线程都提供了执行环境，但是创建新线程所需的资源比创建新进程少。<br/>

&emsp;&emsp;线程存在于一个进程中——每个进程至少有一个线程。线程共享进程的资源，包括内存和打开的文件。这有助于有效的沟通，但也有潜在的问题。<br/>

&emsp;&emsp;多线程执行是Java平台的一个基本特性。每个应用程序至少有一个或多个线程，如果将执行内存管理和信号处理等任务的“系统”线程计算在内的话。但是从应用程序程序员的角度来看，您只从一个线程开始，称为主线程。这个线程能够创建额外的线程，我们将在下一节中进行演示。<br/>

## 线程对象（Thread Objects）
&emsp;&emsp;每个线程都有一个与之相关的[Thread](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html)类实例。使用线程对象创建并发应用程序有两种基本策略。<br/>

*   要直接控制线程的创建和管理，只需在应用程序每次需要启动异步任务时实例化线程即可。<br/>
*   要从应用程序的其余部分抽象线程管理，请将应用程序的任务传递给执行器(executor)。<br/>

&emsp;&emsp;本节记录`Thread`对象的使用。[high-level concurrency objects](https://docs.oracle.com/javase/tutorial/essential/concurrency/highlevel.html)讨论了执行器(Executor)。<br/>

### 定义和启动线程
&emsp;&emsp;创建线程实例的应用程序必须提供将在该线程中运行的代码。有两种方法可以做到这一点:<br/>

*   提供一个`Runnable`对象。[Runnable](https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html)接口定义了一个方法`run`，用于包含在线程中执行的代码。`Runnable`对象被传递给线程构造函数，`HelloRunnable`例子:<br/>

```java
public class HelloRunnable implements Runnable {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new Thread(new HelloRunnable())).start();
    }

}
```
*   子类化`Thread`。`Thread`类本身实现`Runnable`，但是它的`run`方法什么也不做。应用程序可以子类化`Thread`，提供自己的`run`实现，如`HelloThread`示例:<br/>

```java
public class HelloThread extends Thread {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new HelloThread()).start();
    }

}
```

&emsp;&emsp;注意，这两个示例都调用`Thread.start`，以便启动新线程。<br/>

&emsp;&emsp;这些习惯用法你应该用哪一个?第一个习惯用法是使用`Runnable`对象，它更加通用，因为`Runnable`对象可以子类化`Thread`之外的类。第二个习惯用法在简单的应用程序中更容易使用，但受限于任务类必须是`Thread`的后代这一事实。本课重点介绍第一种方法，它将可运行任务与执行任务的`Thread`对象分离开来。这种方法不仅更加灵活，而且适用于稍后介绍的高级线程管理 API。<br/>

&emsp;&emsp;`Thread`类定义了许多对线程管理有用的方法。这些方法包括静态方法，它提供有关调用该方法的线程的信息，或影响该线程的状态。其他方法是从其他线程中调用的，这些线程涉及管理线程和`Thread`对象。我们将在下面几节中研究其中的一些方法。<br/>

## java.lang.Thread

### javadoc文档阅读：
&emsp;&emsp;thread是程序中执行的线程。Java虚拟机允许一个应用程序中有多个执行线程并发运行。<br/>

&emsp;&emsp;每个线程都有一个优先级。优先级高的线程比优先级低的线程优先执行。每个线程可以标记为守护进程，也可以不标记为守护进程。当在某个线程中运行的代码创建一个新线程对象时，新线程的优先级初始设置为创建线程的优先级，并且只有当创建线程是一个守护进程时，新线程才是守护进程。<br/>

&emsp;&emsp;当Java虚拟机启动时，通常有一个非守护进程线程(它通常调用一些指定类的main方法)。Java虚拟机继续执行线程，直到下列情况之一发生:<br/>

*   调用了`Runtime`类的`exit`方法，并且安全管理器允许执行退出操作。<br/>
*   不是守护进程线程的所有线程都已死亡，要么从对`run`方法的调用返回，要么抛出一个在`run`方法之外传播的异常。<br/>

&emsp;&emsp;有两种方法可以创建新的执行线程。第一种方法是将一个类声明为`Thread`类的子类。这个子类应该覆盖类`Thread`的`run`方法。然后可以分配并启动子类的实例。例如，计算大于指定值的质数的线程可以写成:<br/>

```java
class PrimeThread extends Thread {
    long minPrime;
    PrimeThread(long minPrime) {
        this.minPrime = minPrime;
    }

    public void run() {
        // compute primes larger than minPrime
        . . .
    }
}
```

&emsp;&emsp;下面的代码将创建一个线程并开始运行:<br/>

```java
PrimeThread p = new PrimeThread(143);
p.start();
```

&emsp;&emsp;创建线程的另一种方法是声明一个实现`Runnable`接口的类。然后该类实现run方法。然后可以分配该类的实例，在创建Thread时作为参数传递，并启动它。另一种风格的相同例子如下:<br/>

```java
class PrimeRun implements Runnable {
         long minPrime;
         PrimeRun(long minPrime) {
             this.minPrime = minPrime;
         }

         public void run() {
             // compute primes larger than minPrime
              . . .
         }
     }
```

&emsp;&emsp;下面的代码将创建一个线程并开始运行:<br/>

```java
PrimeRun p = new PrimeRun(143);
new Thread(p).start();
```

&emsp;&emsp;每个线程都有一个用于标识的名称。多个线程可能具有相同的名称。如果在创建线程时没有指定名称，则为其生成一个新名称。<br/>

&emsp;&emsp;除非另有说明，否则将null参数传递给该类中的构造函数或方法将引发NullPointerException。<br/>

## java.lang.Runnable

### javadoc文档阅读：
&emsp;&emsp;`Runnable`接口应该被实现。实现类的实例打算由一个线程来执行。实现类必须定义一个叫`run`的无参数方法。<br/>

&emsp;&emsp;此接口旨在为希望在活动时执行代码的对象提供公共协议。例如，`Runnable`是由类`Thread`实现的。活动仅仅意味着线程已经启动，还没有停止。<br/>

&emsp;&emsp;此外，`Runnable`提供了在不子类化`Thread`的情况下激活类的方法。实现`Runnable`的类可以通过实例化`Thread`实例并将自身作为目标传递进来，从而在不子类化`Thread`的情况下运行。在大多数情况下，如果只打算覆盖`run()`方法而不打算覆盖其他方法，则应该使用`Runnable`接口。<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>