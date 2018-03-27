---
layout: post
title: JVM类加载器学习第二篇
---

### 1 类加载的过程
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/classload1.png)

### 2 类加载器的父亲委托机制
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/classloader2.png)

在父亲委托机制中，各个加载器按照父子关系形成了树形结构，除了根类加载器之外，其余的类加载器都有且只有一个父加载器。

双亲委托机制过程描述：

用户自定义类加载器 loader1  想要去加载 用户自己编写的 Sample 类，
并不是由 loader1 直接加载 Sample，相反的是， loader1 将加载的任务交给父亲即系统类加载器来完成，
系统类加载器将加载的任务交给父亲即扩展类加载器来完成，扩展类加载器将加载的任务交给父亲即根类加载器来完成，
由于根类加载器处于加载器的最顶端，它没有父加载器，那么根类加载器尝试着加载Smaple类，
由于根类加载器加载的是指定目录下的jar，那么它将无法加载Sample类，也就是说根类加载器加载失败了，
根类加载器将加载的任务返回给扩展类加载器，此时，扩展类加载器将尝试加载Sample类，
由于扩展类加载器加载的也是指定目录下的jar，那么它将无法加载Sample类，也就是说扩展类加载器加载依然失败了，
扩展类加载器将加载的任务返回给应用类加载器，此时，应用类加载器将尝试加载Sample类，
由于应用类加载器加载的是CLASSPATH下的class文件，而我们编写的Sample类一般都位于当前类路径下，所以，
应用类加载器加载Sample类成功了，将流程返回给loader1，宣告加载Sample类成功。

一般的，一个类加载器在加载 class 的过程中，会将加载的任务交委托它的父亲类加载器，以此类推，直到根类加载器，
因为根类加载器上面没有父类加载器了，然后根类加载器尝试加载 class，如果根类加载器加载不成功，会返回给扩展类加载器，
由扩展类加载器尝试加载 class，从上到下一直返回到最开始的那个类加载器。
在这个过程中，只要有一个层次的类加载器成功加载 class，那么就宣告 指定的类加载成功了，流程会返回到最开始的类加载器。

### 3 类加载器架构图
![classloader](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/classloader3.png)