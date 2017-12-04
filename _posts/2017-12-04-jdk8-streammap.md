---
layout: post
title: jdk8 Stream map(流的映射)
---

## 1. map（对流中每一个元素应用函数）

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

返回<font color="#FF0000">一个新的流</font>，新流中包含的是对当前流中的元素应用给定的函数后，产生的结果。
这是一个中间操作。

R - 新的流中元素的类型。  

mapper - 应用到每个元素的 non-interfering, stateless function。  

返回一个新流。  

----------------------------------------

### 1.1 举例

描述：给定一个单词列表，返回另一个单词列表，显示每个单词中有几个字母。

```java
List<String> l1 = Arrays.asList("Hello", "World", "HelloWorld");
List<Integer> l2 = l1.stream()
                     .map(String::length)
                     .collect(Collectors.toList());
```

## 2. flatMap（流的扁平化）

```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```

返回<font color="#FF0000">一个新的流</font>，包含的结果会替换掉当前流中的每一个元素，用（映射后的流中的内容）替换，这个映射后的流通过（对每一个元素应用所提供的映射函数）产生的


### 2.1 举例

描述：对于一张单词表，如何返回一张列表，列出里面各不相同的字符呢？例如：给定单词列表["Hello", "World"]，返回列表["H","e","l", "o","W","r","d"]。

```java
List<String> words = Arrays.asList("Hello", "World");
//第一个版本
words.stream()
        .map(word -> word.split(""))
        .distinct()
        .collect(Collectors.toList());
```
这段程序是有问题的。传递给 map 的 lambda 为每个单词返回了一个 String[]（String列表）。map返回的流实际上是 Stream<String[]> 类型的。而我们想要的是用 <font color="#FF0000">Stream<String>来表示一个字符流</font>。
