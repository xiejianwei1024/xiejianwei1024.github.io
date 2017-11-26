---
layout: post
title: ValueBased(Oracle 官网翻译)
---

原文：[https://docs.oracle.com/javase/8/docs/api/java/lang/doc-files/ValueBased.html](https://docs.oracle.com/javase/8/docs/api/java/lang/doc-files/ValueBased.html)

## 基于值的类
一些类，如java.util.Optional和 java.time.LocalDateTime，是基于值的。基于值的类的<font color="#FF0000">实例</font>满足以下条件：
*   是最终的和不可变的（尽管可能包含对可变对象的引用）。  
*   具有对 equals、hashCode 和 toString 的实现，这些方法的实现仅仅是通过<font color="#FF0000">实例本身的状态</font>来计算的，并不会通过其他对象或变量的状态来计算。  
*   不要使用身份敏感的操作，如实例之间的引用相等(==)，实例的身份哈希码，实例的内部锁的同步。  
*   判断相等性的条件仅仅基于 equals() ，而不是基于引用相等(==)。  
*   没有可访问的构造方法，而是通过工厂方法实例化，这些工厂方法不保证返回实例的相等性(调用两次，生成的实例不一定是相等的)。  
*   在相等的情况下是可以自由替代的，这意味着在任何计算或方法调用中根据equals()交换任何两个相等的x和y应该不会产生可见的行为变化。  

如果程序尝试区分两个基于值的类的引用之间的相等性（无论直接通过引用相等还是间接地通过对同步，身份散列，序列化或任何其他身份敏感机制），则可能会产生不可预知的结果。在基于值的类的实例上使用这种身份敏感操作可能有不可预知的影响，应该避免。