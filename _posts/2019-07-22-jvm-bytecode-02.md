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

首先，用反编译命令查看MyTest.class文件。

第一步：进入目录：D:\IdeaProjects\jvm_study\out\production\classes>
第二步：输入命令：javap -verbose -p com.shengsiyuan.jvm.bytecode.MyTest2

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

接下来是MyTest2.class以16进制存储的文件结构： 

```java
CA FE BA BE 00 00 00 34  00 49 0A 00 0E 00 2F 08
00 30 09 00 05 00 31 09  00 05 00 32 07 00 33 0A
00 05 00 2F 0A 00 05 00  34 0A 00 35 00 36 09 00
05 00 37 09 00 38 00 39  08 00 3A 0A 00 3B 00 3C
08 00 3D 07 00 3E 01 00  03 73 74 72 01 00 12 4C
6A 61 76 61 2F 6C 61 6E  67 2F 53 74 72 69 6E 67
3B 01 00 01 78 01 00 01  49 01 00 02 69 6E 01 00
13 4C 6A 61 76 61 2F 6C  61 6E 67 2F 49 6E 74 65
67 65 72 3B 01 00 06 3C  69 6E 69 74 3E 01 00 03
28 29 56 01 00 04 43 6F  64 65 01 00 0F 4C 69 6E
65 4E 75 6D 62 65 72 54  61 62 6C 65 01 00 12 4C
6F 63 61 6C 56 61 72 69  61 62 6C 65 54 61 62 6C
65 01 00 04 74 68 69 73  01 00 1A 4C 63 6F 6D 2F
6A 76 6D 2F 62 79 74 65  63 6F 64 65 2F 4D 79 54
65 73 74 32 3B 01 00 04  28 49 29 56 01 00 01 69
01 00 04 6D 61 69 6E 01  00 16 28 5B 4C 6A 61 76
61 2F 6C 61 6E 67 2F 53  74 72 69 6E 67 3B 29 56
01 00 04 61 72 67 73 01  00 13 5B 4C 6A 61 76 61
2F 6C 61 6E 67 2F 53 74  72 69 6E 67 3B 01 00 07
6D 79 54 65 73 74 32 01  00 04 73 65 74 58 01 00
04 74 65 73 74 01 00 15  28 4C 6A 61 76 61 2F 6C
61 6E 67 2F 53 74 72 69  6E 67 3B 29 56 01 00 0D
53 74 61 63 6B 4D 61 70  54 61 62 6C 65 07 00 33
07 00 3F 07 00 3E 07 00  40 01 00 05 74 65 73 74
32 01 00 08 3C 63 6C 69  6E 69 74 3E 01 00 0A 53
6F 75 72 63 65 46 69 6C  65 01 00 0C 4D 79 54 65
73 74 32 2E 6A 61 76 61  0C 00 15 00 16 01 00 07
57 65 6C 63 6F 6D 65 0C  00 0F 00 10 0C 00 11 00
12 01 00 18 63 6F 6D 2F  6A 76 6D 2F 62 79 74 65
63 6F 64 65 2F 4D 79 54  65 73 74 32 0C 00 23 00
1C 07 00 41 0C 00 42 00  43 0C 00 13 00 14 07 00
44 0C 00 45 00 46 01 00  0B 68 65 6C 6C 6F 20 77
6F 72 6C 64 07 00 47 0C  00 48 00 25 01 00 03 61
62 63 01 00 10 6A 61 76  61 2F 6C 61 6E 67 2F 4F
62 6A 65 63 74 01 00 10  6A 61 76 61 2F 6C 61 6E
67 2F 53 74 72 69 6E 67  01 00 13 6A 61 76 61 2F
6C 61 6E 67 2F 54 68 72  6F 77 61 62 6C 65 01 00
11 6A 61 76 61 2F 6C 61  6E 67 2F 49 6E 74 65 67
65 72 01 00 07 76 61 6C  75 65 4F 66 01 00 16 28
49 29 4C 6A 61 76 61 2F  6C 61 6E 67 2F 49 6E 74
65 67 65 72 3B 01 00 10  6A 61 76 61 2F 6C 61 6E
67 2F 53 79 73 74 65 6D  01 00 03 6F 75 74 01 00
15 4C 6A 61 76 61 2F 69  6F 2F 50 72 69 6E 74 53
74 72 65 61 6D 3B 01 00  13 6A 61 76 61 2F 69 6F
2F 50 72 69 6E 74 53 74  72 65 61 6D 01 00 07 70
72 69 6E 74 6C 6E 00 21  00 05 00 0E 00 00 00 03
00 00 00 0F 00 10 00 00  00 02 00 11 00 12 00 00
00 09 00 13 00 14 00 00  00 07 00 01 00 15 00 16
00 01 00 17 00 00 00 46  00 02 00 01 00 00 00 10
2A B7 00 01 2A 12 02 B5  00 03 2A 08 B5 00 04 B1
00 00 00 02 00 18 00 00  00 12 00 04 00 00 00 0F
00 04 00 05 00 0A 00 07  00 0F 00 11 00 19 00 00
00 0C 00 01 00 00 00 10  00 1A 00 1B 00 00 00 01
00 15 00 1C 00 01 00 17  00 00 00 50 00 02 00 02
00 00 00 10 2A B7 00 01  2A 12 02 B5 00 03 2A 08
B5 00 04 B1 00 00 00 02  00 18 00 00 00 12 00 04
00 00 00 13 00 04 00 05  00 0A 00 07 00 0F 00 15
00 19 00 00 00 16 00 02  00 00 00 10 00 1A 00 1B
00 00 00 00 00 10 00 1D  00 12 00 01 00 09 00 1E
00 1F 00 01 00 17 00 00  00 57 00 02 00 02 00 00
00 17 BB 00 05 59 B7 00  06 4C 2B 10 08 B7 00 07
10 14 B8 00 08 B3 00 09  B1 00 00 00 02 00 18 00
00 00 12 00 04 00 00 00  18 00 08 00 1A 00 0E 00
1C 00 16 00 1D 00 19 00  00 00 16 00 02 00 00 00
17 00 20 00 21 00 00 00  08 00 0F 00 22 00 1B 00
01 00 22 00 23 00 1C 00  01 00 17 00 00 00 3E 00
02 00 02 00 00 00 06 2A  1B B5 00 04 B1 00 00 00
02 00 18 00 00 00 0A 00  02 00 00 00 20 00 05 00
21 00 19 00 00 00 16 00  02 00 00 00 06 00 1A 00
1B 00 00 00 00 00 06 00  11 00 12 00 01 00 02 00
24 00 25 00 01 00 17 00  00 00 85 00 02 00 04 00
00 00 17 2B 59 4D C2 B2  00 0A 12 0B B6 00 0C 2C
C3 A7 00 08 4E 2C C3 2D  BF B1 00 02 00 04 00 0E
00 11 00 00 00 11 00 14  00 11 00 00 00 03 00 18
00 00 00 12 00 04 00 00  00 24 00 04 00 25 00 0C
00 26 00 16 00 27 00 19  00 00 00 16 00 02 00 00
00 17 00 1A 00 1B 00 00  00 00 00 17 00 0F 00 10
00 01 00 26 00 00 00 18  00 02 FF 00 11 00 03 07
00 27 07 00 28 07 00 29  00 01 07 00 2A FA 00 04
00 2A 00 2B 00 16 00 01  00 17 00 00 00 19 00 00
00 00 00 00 00 01 B1 00  00 00 01 00 18 00 00 00
06 00 01 00 00 00 29 00  08 00 2C 00 16 00 01 00
17 00 00 00 31 00 02 00  00 00 00 00 11 10 0A B8
00 08 B3 00 09 B2 00 0A  12 0D B6 00 0C B1 00 00
00 01 00 18 00 00 00 0E  00 03 00 00 00 09 00 08
00 0C 00 10 00 0D 00 01  00 2D 00 00 00 02 00 2E
```



















