---
layout: post
title: java.util.LinkedList 源代码阅读
---

## 前言
（1）LinkedList的内部实现是双向链表，继承了AbstractSequentialList，实现了List, Deque, Cloneable, Java.io.Serializable接口，因此LinkdeList本身支持就支持双端队列操作。LinkedList**允许所有元素（包括 null）**。除了实现 List 接口外，LinkedList 类还为在列表的开头及结尾 get、remove 和 insert 元素提供了统一的命名方法。这些操作允许将链接列表用作堆栈、队列或双端队列。

## 源码
LinkedList内部通过Node来抽象一个节点，结点包括值和前向指针和后向指针。Node的定义是在LinkedList内部，作为静态内部类存在。

```java
private static class Node<E> {
    E item; //结点的值
    Node<E> next; //结点的后向指针
    Node<E> prev; //结点的前向指针
    //构造函数中已完成Node成员的赋值
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element; //结点的值赋值为element
        this.next = next; //后向指针赋值
        this.prev = prev; //前向指针赋值
    }
}
```

未完待续...