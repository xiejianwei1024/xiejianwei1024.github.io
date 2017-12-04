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
List<String> words = Arrays.asList("Hello", "World", "HelloWorld");
List<Integer> wordLengths = words.stream().map(String::length).collect(Collectors.toList());
```

## 2. flatMap（流的扁平化）

```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```

返回<font color="#FF0000">一个新的流</font>，包含的结果会替换掉当前流中的每一个元素，用什么（）替换