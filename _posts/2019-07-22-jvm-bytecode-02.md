---
layout: post
title: JVM java字节码文件结构分析示例2
---

MyTest2.java源文件如下：

```java
package com.jvm.bytecode;

public class MyTest2 {

    String str = "Welcome";

    private int x = 5;

    public static Integer in = 10;

    static {
        System.out.println("abc");
    }

    public MyTest2() {

    }

    public MyTest2(int i) {

    }

    public static void main(String[] args) {
        MyTest2 myTest2 = new MyTest2();

        myTest2.setX(8);

        in = 20;
    }

    private synchronized void setX(int x) {
        this.x = x;
    }

    private void test(String str) {
        synchronized (str) {
            System.out.println("hello world");
        }
    }

    private synchronized static void test2() { }
}
```

D:\IdeaProjects\jvm_study\out\production\classes>
javap -verbose -p com.shengsiyuan.jvm.bytecode.MyTest2

```java
D:\IdeaProjects\jvm_study\out\production\classes>javap -verbose -p com.jvm.bytecode.MyTest2
Classfile /D:/IdeaProjects/jvm_study/out/production/classes/com/jvm/bytecode/MyTest2.class
  Last modified 2019-7-22; size 1376 bytes
  MD5 checksum a9342f5d15b233ebdd4ff648010f7859
  Compiled from "MyTest2.java"
public class com.jvm.bytecode.MyTest2
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #14.#47        // java/lang/Object."<init>":()V
   #2 = String             #48            // Welcome
   #3 = Fieldref           #5.#49         // com/jvm/bytecode/MyTest2.str:Ljava/lang/String;
   #4 = Fieldref           #5.#50         // com/jvm/bytecode/MyTest2.x:I
   #5 = Class              #51            // com/jvm/bytecode/MyTest2
   #6 = Methodref          #5.#47         // com/jvm/bytecode/MyTest2."<init>":()V
   #7 = Methodref          #5.#52         // com/jvm/bytecode/MyTest2.setX:(I)V
   #8 = Methodref          #53.#54        // java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
   #9 = Fieldref           #5.#55         // com/jvm/bytecode/MyTest2.in:Ljava/lang/Integer;
  #10 = Fieldref           #56.#57        // java/lang/System.out:Ljava/io/PrintStream;
  #11 = String             #58            // hello world
  #12 = Methodref          #59.#60        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #13 = String             #61            // abc
  #14 = Class              #62            // java/lang/Object
  #15 = Utf8               str
  #16 = Utf8               Ljava/lang/String;
  #17 = Utf8               x
  #18 = Utf8               I
  #19 = Utf8               in
  #20 = Utf8               Ljava/lang/Integer;
  #21 = Utf8               <init>
  #22 = Utf8               ()V
  #23 = Utf8               Code
  #24 = Utf8               LineNumberTable
  #25 = Utf8               LocalVariableTable
  #26 = Utf8               this
  #27 = Utf8               Lcom/jvm/bytecode/MyTest2;
  #28 = Utf8               (I)V
  #29 = Utf8               i
  #30 = Utf8               main
  #31 = Utf8               ([Ljava/lang/String;)V
  #32 = Utf8               args
  #33 = Utf8               [Ljava/lang/String;
  #34 = Utf8               myTest2
  #35 = Utf8               setX
  #36 = Utf8               test
  #37 = Utf8               (Ljava/lang/String;)V
  #38 = Utf8               StackMapTable
  #39 = Class              #51            // com/jvm/bytecode/MyTest2
  #40 = Class              #63            // java/lang/String
  #41 = Class              #62            // java/lang/Object
  #42 = Class              #64            // java/lang/Throwable
  #43 = Utf8               test2
  #44 = Utf8               <clinit>
  #45 = Utf8               SourceFile
  #46 = Utf8               MyTest2.java
  #47 = NameAndType        #21:#22        // "<init>":()V
  #48 = Utf8               Welcome
  #49 = NameAndType        #15:#16        // str:Ljava/lang/String;
  #50 = NameAndType        #17:#18        // x:I
  #51 = Utf8               com/jvm/bytecode/MyTest2
  #52 = NameAndType        #35:#28        // setX:(I)V
  #53 = Class              #65            // java/lang/Integer
  #54 = NameAndType        #66:#67        // valueOf:(I)Ljava/lang/Integer;
  #55 = NameAndType        #19:#20        // in:Ljava/lang/Integer;
  #56 = Class              #68            // java/lang/System
  #57 = NameAndType        #69:#70        // out:Ljava/io/PrintStream;
  #58 = Utf8               hello world
  #59 = Class              #71            // java/io/PrintStream
  #60 = NameAndType        #72:#37        // println:(Ljava/lang/String;)V
  #61 = Utf8               abc
  #62 = Utf8               java/lang/Object
  #63 = Utf8               java/lang/String
  #64 = Utf8               java/lang/Throwable
  #65 = Utf8               java/lang/Integer
  #66 = Utf8               valueOf
  #67 = Utf8               (I)Ljava/lang/Integer;
  #68 = Utf8               java/lang/System
  #69 = Utf8               out
  #70 = Utf8               Ljava/io/PrintStream;
  #71 = Utf8               java/io/PrintStream
  #72 = Utf8               println
{
  java.lang.String str;
    descriptor: Ljava/lang/String;
    flags:

  private int x;
    descriptor: I
    flags: ACC_PRIVATE

  public static java.lang.Integer in;
    descriptor: Ljava/lang/Integer;
    flags: ACC_PUBLIC, ACC_STATIC

  public com.jvm.bytecode.MyTest2();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String Welcome
         7: putfield      #3                  // Field str:Ljava/lang/String;
        10: aload_0
        11: iconst_5
        12: putfield      #4                  // Field x:I
        15: return
      LineNumberTable:
        line 15: 0
        line 5: 4
        line 7: 10
        line 17: 15
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      16     0  this   Lcom/jvm/bytecode/MyTest2;

  public com.jvm.bytecode.MyTest2(int);
    descriptor: (I)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String Welcome
         7: putfield      #3                  // Field str:Ljava/lang/String;
        10: aload_0
        11: iconst_5
        12: putfield      #4                  // Field x:I
        15: return
      LineNumberTable:
        line 19: 0
        line 5: 4
        line 7: 10
        line 21: 15
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      16     0  this   Lcom/jvm/bytecode/MyTest2;
            0      16     1     i   I

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #5                  // class com/jvm/bytecode/MyTest2
         3: dup
         4: invokespecial #6                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: bipush        8
        11: invokespecial #7                  // Method setX:(I)V
        14: bipush        20
        16: invokestatic  #8                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        19: putstatic     #9                  // Field in:Ljava/lang/Integer;
        22: return
      LineNumberTable:
        line 24: 0
        line 26: 8
        line 28: 14
        line 29: 22
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      23     0  args   [Ljava/lang/String;
            8      15     1 myTest2   Lcom/jvm/bytecode/MyTest2;

  private synchronized void setX(int);
    descriptor: (I)V
    flags: ACC_PRIVATE, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: iload_1
         2: putfield      #4                  // Field x:I
         5: return
      LineNumberTable:
        line 32: 0
        line 33: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lcom/jvm/bytecode/MyTest2;
            0       6     1     x   I

  private void test(java.lang.String);
    descriptor: (Ljava/lang/String;)V
    flags: ACC_PRIVATE
    Code:
      stack=2, locals=4, args_size=2
         0: aload_1
         1: dup
         2: astore_2
         3: monitorenter
         4: getstatic     #10                 // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #11                 // String hello world
         9: invokevirtual #12                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_2
        13: monitorexit
        14: goto          22
        17: astore_3
        18: aload_2
        19: monitorexit
        20: aload_3
        21: athrow
        22: return
      Exception table:
         from    to  target type
             4    14    17   any
            17    20    17   any
      LineNumberTable:
        line 36: 0
        line 37: 4
        line 38: 12
        line 39: 22
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      23     0  this   Lcom/jvm/bytecode/MyTest2;
            0      23     1   str   Ljava/lang/String;
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 17
          locals = [ class com/jvm/bytecode/MyTest2, class java/lang/String, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4

  private static synchronized void test2();
    descriptor: ()V
    flags: ACC_PRIVATE, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=0, locals=0, args_size=0
         0: return
      LineNumberTable:
        line 41: 0

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=2, locals=0, args_size=0
         0: bipush        10
         2: invokestatic  #8                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
         5: putstatic     #9                  // Field in:Ljava/lang/Integer;
         8: getstatic     #10                 // Field java/lang/System.out:Ljava/io/PrintStream;
        11: ldc           #13                 // String abc
        13: invokevirtual #12                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        16: return
      LineNumberTable:
        line 9: 0
        line 12: 8
        line 13: 16
}
SourceFile: "MyTest2.java"
```





















