---
layout: post
title: jdk8 Collector(收集器接口)
---
1. collect : 收集器
2. Collector 作为 collect 方法的参数
3. 收集器接口是一个可变的汇聚操作，将输入元素累积到一个可变的结果容器中，
它会在所有输入元素处理完毕之后，将累积的结果转换为一个最终表示（这是一个可选操作）。它（汇聚操作）支持串行和并行两种方式执行。
4. Collectors 本身提供了关于 Collector 的常见汇聚实现，Collectors 本身是一个工厂。