---
layout: post
title: Java Design Patterns（creationalpattern）Builder
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
## 意图
&emsp;&emsp;将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。<br/>

&emsp;&emsp;建造者模式是一种对象创建型模式。建造者模式一步一步创建一个复杂的对象，它允许用户只通过指定复杂对象的类型和内容就可以构建它们，用户不需要知道内部的具体构建细节。<br/>

## 角色
*   <span style="font-weight:bold;">抽象建造者（Builder）</span>：为创建一个Product对象的各个部件指定抽象接口。<br/>说明：在该接口中一般声明两类方法，一类方法是buildPartX()，它们用于创建复杂对象的各个部件；另一类方法是getResult()，它们用于返回复杂对象。Builder既可以是抽象类，也可以是接口。

*   <span style="font-weight:bold;">具体建造者（ConcreteBuilder）</span>：实现Builder接口，构造和装配产品的各个部件。定义并明确它所创建的表示。提供一个返回这个产品的接口。

*   <span style="font-weight:bold;">指挥者（Director）</span>：构建一个使用Builder接口的对象。<br/>
说明：它负责安排复杂对象的建造次序，指挥者与抽象建造者之间存在关联关系，可以在其construct()建造方法中调用建造者对象的部件构造与装配方法，完成复杂对象的建造。客户端一般只需要与指挥者进行交互，在客户端确定具体建造者的类型，并实例化具体建造者对象（也可以通过配置文件和反射机制），然后通过指挥者类的构造函数或者Setter方法将该对象传入指挥者类中。

*   <span style="font-weight:bold;">产品角色（Product）</span>：被构建的复杂对象，具体建造者创建该产品的内部表示并定义它的装配过程。包含定义组成部件的类，包括将这些部件装配成最终产品的接口。<br/><br/>
<span style="font-weight:bold;">复杂对象</span>：是指那些包含多个成员属性的对象，这些成员属性也称为部件或零件，如汽车包括方向盘、发动机、轮胎等部件，电子邮件包括发件人、收件人、主题、内容、附件等部件。

## 适用性
*   当创建复杂对象的算法应该独立于该对象的组成部分以及它们的装配方式时。

*   当构造过程必须允许被构造的对象有不同的表示时。

&emsp;&emsp;上面两点太难理解了，反正笔者看一次懵逼一次。所以有了下面的适用场景。<br/>

## 适用场景（为了更好的理解适用性）
*   需要生成的产品对象有复杂的内部结构，这些产品对象通常包含多个成员属性。

*   需要生成的产品对象的属性相互依赖，需要指定其生成顺序。

*   对象的创建过程独立于创建该对象的类。在建造者模式中通过引入了指挥者类，将创建过程封装在指挥者类中，而不在建造者类和客户类中。

*   隔离复杂对象的创建和使用，并使得相同的创建过程可以创建不同的产品。

## 优缺点
建造者模式的<span style="font-weight:bold;">主要优点</span>如下：

*   在建造者模式中，客户端不必知道产品内部组成的细节，将产品本身与产品的创建过程解耦，使得相同的创建过程可以创建不同的产品对象。

*   每一个具体建造者都相对独立，而与其他的具体建造者无关，因此可以很方便地替换具体建造者或增加新的具体建造者，用户使用不同的具体建造者即可得到不同的产品对象。由于指挥者类针对抽象建造者编程，增加新的具体建造者无须修改原有类库的代码，系统扩展方便，符合 "开闭原则"。

*   可以更加精细地控制产品的创建过程。将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰，也更方便使用程序来控制创建过程。

建造者模式的<span style="font-weight:bold;">主要缺点</span>如下：

*   建造者模式所创建的产品一般具有较多的共同点，其组成部分相似，如果产品之间的差异性很大，例如很多组成部分都不相同，不适合使用建造者模式，因此其使用范围受到一定的限制。

*   如果产品的内部变化复杂，可能会导致需要定义很多具体建造者类来实现这种变化，导致系统变得很庞大，增加系统的理解难度和运行成本。

<span style="color:red;">以上就是建造者模式的理论部分。一定要理解好理论部分，然后结合代码示例，加深对理论的理解。也就是我们常说的理论结合实际。</span>

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
`StringBuilder` 为指挥者角色，由于`StringBuilder`继承了`AbstractStringBuilder`，这里`StringBuilder`通过`super`来作为具体建造者的引用。由于`append` 方法返回的是 `StringBuilder` 自身，所以 `StringBuilder` 还充当了产品角色。我们平常所用的`sb.append().append()`这种链式调用，最终的产品就是`StringBuilder`对象。<br/><br/>
## JDK库中的建造者模式-java.lang.StringBuffer
StringBuffer 的继承实现关系如下所示：

![StringBuilder](https://raw.githubusercontent.com/xiejianwei1024/markdownphotos/master/design-patterns/StringBuffer.png)

```java
public final class StringBuffer extends AbstractStringBuilder implements java.io.Serializable, CharSequence {
    @Override
    public synchronized StringBuffer append(char c) {
        toStringCache = null;
        super.append(c);
        return this;
    }
    //省略
}
```
&emsp;&emsp;`StringBuffer` 中的 `append` 加了 `synchronized` 关键字，所以 `StringBuffer` 是线程安全的，而 `StringBuilder` 是非线程安全的。<br/><br/>

&emsp;&emsp;`StringBuffer` 中的建造者模式与 `StringBuilder` 是一致的。<br/><br/>