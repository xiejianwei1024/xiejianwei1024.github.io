---
layout: post
title: [Java Design Patterns]（creationalpattern）Builder
---
# Builder

## Purpose
*   Separate the construction of a complex object from its representation so that the same construction process can create different representations.

## Structure
![Builder_Model1](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/design-patterns/Builder_Model1.gif)

*   <span style="font-weight:bold;">Builder</span> : specifies an abstract interface for creating parts of a Product object.
*   <span style="font-weight:bold;">ConcreteBuilder</span> : Constructs and assembles parts of the product by implementing the Builder interface. Defines and keeps track of the representation it creates. Provides an interface for retrieving the product (e.g., GetASCIIText, GetTextWidget).
*   <span style="font-weight:bold;">Director</span> : Constructs an object using the Builder interface.
*   <span style="font-weight:bold;">Product</span> : Represents the complex object under construction. ConcreteBuilder builds the product's internal representation and defines the process by which it's assembled

## Interaction
![Builder_Model1](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/design-patterns/Build_Model_Seq1.gif)

*   The client creates the Director object and configures it with the desired Builder object.
*   Director notifies the builder whenever a part of the product should be built.
*   Builder handles requests from the director and adds parts to the product.
*   The client retrieves the product from the builder.

## Applications
*   The algorithm for creating a complex object should be independent of the parts that make up the object and how they're assembled.
*   The construction process must allow different representations for the object that's constructed.

## Consequences
*   <span style="font-weight:bold;">It lets you vary a product's internal representation.</span> The Builder object provides the director with an abstract interface for constructing the product. The interface lets the builder hide the representation and internal structure of the product. It also hides how the product gets assembled. Because the product is constructed through an abstract interface, all you have to do to change the product's internal representation is define a new kind of builder.
*   <span style="font-weight:bold;">It isolates code for construction and representation.</span> I The Builder pattern improves modularity by encapsulating the way a complex object is constructed and represented. Clients needn't know anything about the classes that define the product's internal structure; such classes don't appear in Builder's interface.
*   <span style="font-weight:bold;">It gives you finer control over the construction process.</span> Unlike creational patterns that construct products in one shot, the Builder pattern constructs the product step by step under the director's control. Only when the product is finished does the director retrieve it from the builder. Hence the Builder interface reflects the process of constructing the product more than other creational patterns. This gives you finer control over the construction process and consequently the internal structure of the resulting product.

# 建造者模式

## JDK库中的建造者模式-java.lang.StringBuilder
StringBuilder 的继承实现关系如下所示：

![StringBuilder](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/design-patterns/StringBuilder.png)

&emsp;&emsp;`Appendable` 接口如下：<br/>

```java
public interface Appendable {
    Appendable append(CharSequence csq) throws IOException;
    Appendable append(CharSequence csq, int start, int end) throws IOException;
    Appendable append(char c) throws IOException;
}
```
&emsp;&emsp;`StringBuilder` 的父类 `AbstractStringBuilder` 实现了 Appendable 接口。<br/>
```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    @Override
    public AbstractStringBuilder append(char c) {
        ensureCapacityInternal(count + 1);
        value[count++] = c;
        return this;
    }
    //省略
}
```

&emsp;&emsp;`StringBuilder` 中的 `append` 方法使用了建造者模式，不过装配方法只有一个，并不算复杂，`append` 方法返回的是 `StringBuilder` 自身。<br/>
```java
public final class StringBuilder extends AbstractStringBuilder implements java.io.Serializable, CharSequence {
    @Override
    public StringBuilder append(char c) {
        super.append(c);
        return this;
    }
    //省略
}
```
&emsp;&emsp;我们可以看出，<br/>`Appendable` 为抽象建造者，定义了建造方法；<br/>`AbstractStringBuilder`为具体建造者，它实现了appendable接口的append(Character c)方法；<br/>
`StringBuilder` 为指挥者角色，由于`StringBuilder`继承了`AbstractStringBuilder`，这里`StringBuilder`通过`super`来作为具体建造者的引用。<br/><br/>