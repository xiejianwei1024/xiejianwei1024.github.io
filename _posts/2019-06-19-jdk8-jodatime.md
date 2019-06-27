---
layout: post
title: Joda-Time
---

了解 joda-time的项目背景：https://www.joda.org/joda-time/。


关于日期和时间

1. 格林威治标准时间。

2. UTC时间 如：2010-12-1T11:22:33.567Z

3. ISO8601 joda-time

joda-time 和 java8的time， 它们的类都是

不可变的，线程安全的，immutable and thread-safe.

现有的Date，SimpleDateFormat

都是可变的，非线程安全的

java 8 time，会使用就可以了。