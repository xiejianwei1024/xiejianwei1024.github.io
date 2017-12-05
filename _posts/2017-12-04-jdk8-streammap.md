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

----------------------------------------

## 2. flatMap（流的扁平化）

```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```

返回<font color="#FF0000">一个新的流</font>，新流中包含的结果会替换掉当前流中的每一个元素，用（映射流中的内容）替换，这个映射流是通过（对每一个元素应用由我们自己所提供的映射函数）产生的。每个映射流(在其内容放入此流之后)都会关闭。（如果映射流为空，则使用空流）。这是一个中间操作。

注意：  
flatMap（）操作的作用是对流的元素应用一对多的转换，然后将生成的元素打平成一个新的流。换句话说，flatMap（）让你把一个流中的每个值都换成另一个流，然后把所有的流连接起来（即打平）成为一个流。

R - 新的流中元素的类型。  

mapper - 是一个 non-interfering, stateless function， 对每个元素应用，每个元素生成一连串的新值（一个流中有多个值）。  这里也体现了一对多的概念。

返回一个新流。  

----------------------------------------

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

这段程序是有问题的。传递给 map 的 lambda 表达式 为每个单词返回了一个 String[]（String列表）。map返回的流实际上是 `Stream<String[]>` 类型的。而我们想要的是用`Stream<String>`来表示一个字符流。

----------------------------------------

**尝试使用 map 和 Arrays.stream()**  
我们需要一个字符流，而不是数组流。Arrays.stream() 可以接受一个数组并产生一个流。

```java
List<String> words = Arrays.asList("Hello", "World");
//第二个版本
words.stream()
        .map(word -> word.split(""))
        .map(Arrays::stream)
        .distinct()
        .collect(Collectors.toList());
```

这个段程序仍然是有问题的。我们现在得到的是一个字符流的列表（`Stream<String>`）。

----------------------------------------

**使用 flatMap**  

```java
List<String> words = Arrays.asList("Hello", "World");
//第三个版本
words.stream()
        .map(word -> word.split(""))
        .flatMap(Arrays::stream)
        .distinct()
        .collect(Collectors.toList());
```

使用 flatMap, 各个数组并不是分别映射成一个流，而是映射成流的内容。所有使用 `map(Arrays::stream)` 时生成的单个流都被合并起来，即扁平化为一个流。

----------------------------------------

### 2.2 举例

描述：给定两个单词列表["Hi", "Hello", "你好"] 和 ["zhangsan", "lisi", "wangwu", "zhaoliu"]，返回列表["Hi zhangsan", "Hi lisi", "Hi wangwu", "Hi zhaoliu", "Hello zhangsan", "Hello lisi", "Hello wangwu", "Hello zhaoliu", "你好 zhangsan", "你好 lisi", "你好 wangwu", "你好 zhaoliu"]。

```java
List<String> l1 = Arrays.asList("Hi", "Hello", "你好");
List<String> l2 = Arrays.asList("zhangsan", "lisi", "wangwu", "zhaoliu");
List<String> result = l1.stream()
                        .flatMap(item1 -> l2.stream()
                                            .map(item2 -> item1 + " " + item2))
                        .collect(Collectors.toList());
```

`flatMap()` 接受的 lambda 表达式 返回的是 Stream<String>类型的，首先将字符串 列表 l2 转化成为（`Stream<String>`）类型的流 ，然后将流中的元素应用函数（的实例即第二个lambda表达式`item2 -> item1 + " " + item2`），每个元素都生成新（`Stream<String>`）类型的流。  

自定义流的表示：\|item1, item2\|  

第一个流：\|"Hi zhangsan", "Hi lisi", "Hi wangwu", "Hi zhaoliu"\|，  

第二个流：\|"Hello zhangsan", "Hello lisi", "Hello wangwu", "Hello zhaoliu"\|，  

第三个流：\|"你好 zhangsan", "你好 lisi", "你好 wangwu", "你好 zhaoliu"\|  

`flatMap()` 将这三个流连接在一起，生成（打平即扁平化为）一个新的流。新流中的元素即三个流中的元素被打平到一个流中了。

新的流：\|"Hi zhangsan", "Hi lisi", "Hi wangwu", "Hi zhaoliu", "Hello zhangsan", "Hello lisi", "Hello wangwu", "Hello zhaoliu", "你好 zhangsan", "你好 lisi", "你好 wangwu", "你好 zhaoliu"\|