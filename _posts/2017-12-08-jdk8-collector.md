---
layout: post
title: jdk8 Collector(收集器接口)
---
### 1 Collector 收集器
#### 1.1 Collector 作为 `collect()` 方法的参数
        收集器接口是一个可变的汇聚操作，将输入元素累积到一个可变的结果容器中，它会在所有输入元素处理完毕之后，将累积的结果转换为一个最终表示（这是一个可选操作）。它（汇聚操作）支持串行和并行两种方式执行。  
Collectors 本身提供了关于 Collector 的常见汇聚实现，Collectors 本身是一个工厂。  
可变的汇聚操作包括：  
1）将元素累积到一个 Collection（集合）中；  
2）使用 StringBuilder 连接字符串；  
3）计算元素的汇总信息，如 sum, min, max, or average；  
4）计算“数据透视表”（pivot table）的汇总信息，如“根据卖方（分组）得到的交易数量最大值”；  

----------------------------------------

5. 收集器由四个函数指定，它们一起工作，将条目累积到一个可变的结果容器中，并且可以对结果执行一个最终转换（这是一个可选操作）。  
这四个函数分别是：  
1）生成器：创建一个新的结果容器（`supplier()`）  
2）累加器：将一个数据元素合并到一个结果容器中（`accumulator()`）  
3）合并器：将两个结果容器合并成为一个（`combiner()`）  
4）完成器：将累积的中间结果执行一个最终转换，这是一个可选操作（`finisher()`）  
6. 收集器还有一个返回 Set<Characteristics > 的 函数 `characteristics()`，比如 `Collector.Characteristics.CONCURRENT`，它提供的暗示可以在汇聚实现中使用，以便提供更好的性能。

----------------------------------------

7. 收集器的汇聚操作的串行实现，使用 supplier 函数创建一个结果容器，对每个输入元素都执行一次 accumulator 函数。  
并行实现，将输入分区，为每一个分区创建一个结果容器，将每一个分区的内容累积到对应的子结果容器中，然后使用 combiner 函数将这些子结果容器合并到一个结果容器中。
8. 为确保串行与并行结果的相等性，Collector 函数需要满足两个条件：identity（同一性） 和 associativity（结合性）。
1）同一性约束指的是对于任意的部分累积结果来说，将它与一个空的结果容器合并的话，必须生成相等的结果。也就是说，有一个部分累积结果 a，它是并行的某一条线上的累加器和合并器调用，a 等价于 combiner.apply(a, supplier.get())。   
例子：(List<String> list1, List<String> list2) -> { list1.addAll(list2); return list1;}
说明：list1和list2分别是 combiner 函数的两个参数  
对比：list1 相当于 a，list2 相当于supplier.get()（新创建的一个空的结果容器）。
2）结合性约束指的是分割计算必须生成相同的结果。（单线程就是顺序执行生成一个结果；多线程会分成多份去并行执行，然后将生成的结果合并起来生成一个最终的结果，这两个结果必须是相同的。）也就是说，对于任何的输入元素 t1 和 t2,结果 r1 和 r2 在下面的计算中必须是等价的：

```java
A a1 = supplier.get();
accumulator.accept(a1, t1);
accumulator.accept(a1, t2);
R r1 = finisher.apply(a1);  // result without splitting

A a2 = supplier.get();
accumulator.accept(a2, t1);
A a3 = supplier.get();
accumulator.accept(a3, t2);
R r2 = finisher.apply(combiner.apply(a2, a3));  // result with splitting
```  
第一段代码的结果是没有使用分割操作的，换句话说是单线程的；第二段代码的结果使用了分割操作，是多线程的。  
`accumulator.accept(a1, t1);` 接受两个参数 a1 和 t1。  
第一个参数：每一次累加之后的中间结果；  
第二个参数：流中要处理（待累加）的下一个元素；  
9. 那些基于 `Collector` 实现汇聚操作的库，比如 `Stream.collect(Collector)`，必须遵循以下的约定：  
1）泛型 A 必须是中间结果类型。
2) 要么将中间结果继续传递下去，要么返回，否则不要对中间结果做任何的操作。（函数式编程的技巧，否则并行情况下，结果会错乱。）
3）如果将 结果 传递给了 `combiner()` 或者 `finisher()` ，相同的对象也没有返回，说明结果已经使用完了。
4）一旦将 结果 传递给了 `combiner()` 或者 `finisher()` ，就不会再传递给 `accumulator()`，因为 `Collector` 执行的顺序是固定的，说明已经执行完累加器函数了。
5）非并发的收集器，结果必须是线程限制的（thread-confined）。并发的时候，保证线程的独立性，针对每个线程分区，累加完成以后再合并。
6）并发的收集器，可以有选择地（不强制）实现并发的汇聚操作。并发的汇聚操作指的是累加器被多个线程并发的调用，使用相同的可并发修改的结果容器（对同一个结果容器操作）。  
这两种情况下应该应用并发的汇聚操作：
第一种情况：收集器 具有 `Characteristics.UNORDERED`  特性。
第二种情况：数据源是 unordered 没有顺序的。


