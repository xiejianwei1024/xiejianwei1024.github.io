---
layout: post
title: jdk8-Default Methods
---

## 何为默认方法
集合框架的List接口中的sort方法是Java 8中全新的方法：

```java
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```

通过默认方法你可以指定接口方法的默认实现。换句话说，接口能提供方法的具体实现。
因此，实现接口的类 如果不显式地提供该方法的具体实现，就会自动继承默认的实现。
这种机制可以使你平滑地进行接口的优化和演进。

## 为何要引入默认方法

实现接口的类必须为接口中定义的每个方法提供一个实现，或者从父类中继承它的实现。但是，一旦类库的设计者需要更新接口，向其中加入新的方法，这种方式就会出现问题。现实情况是，现存的实体类往往不在接口设计者的控制范围之内，这些实体类为了适配新的接口约定也需要进行修改。

jdk8之前，要想给一个列表中的元素排序，比如，对inventory中的苹果按照重量进行排序，我们需要这样做：

```java
Collections.sort(inventory, new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
}); 
```

根据语义来说，我们根本没必要引入 Collections，明明是对库存中的苹果排序，在jdk8中，是这样做的：

```java
inventory.sort(comparing(Apple::getWeight));
```

inventory是一个列表，是List类型的，但是给List接口增加 sort() 方法，势必会导致下面的每一个实现类都提供对
sort()方法的实现。

进一步扩展，如果在原有的接口中增加多个方法呢，那么实现类的任务是繁重的。

所以就有了接口中的默认方法（default method）。

默认方法的引入就是为了以兼容的方式解决像 Java API这样的类库的演进问题的，

引入默认方法的目的：它让类可以自动地继承接口的一个默认实现。


## 默认方法的基本结构

```java
public interface MyInterface1 {
    default void myMethod() {
        System.out.println("MyInterace1");
    }
}
```
首先默认方法必须定义在接口中；
其次默认方法必须使用 default 修饰符修饰。

## 默认方法的使用模式

默认方法的两种用例：可选方法和行为的多继承。 

### 可选方法

类实现了接口，不过却刻意地将一些方法的实现留白。我们以 Iterator接口为例来说。Iterator接口定义了hasNext、next，还定义了remove方法。Java 8 之前，由于用户通常不会使用该方法，remove方法常被忽略。因此，实现Interator接口的类通常会为remove方法放置一个空的实现，这些都是些毫无用处的模板代码。
采用默认方法之后，你可以为这种类型的方法提供一个默认的实现，这样实体类就无需在自己的实现中显式地提供一个空方法。比如，在Java 8中，Iterator接口就为remove方法提供了一个默认实现，如下所示：
```java
interface Iterator<T> {
    boolean hasNext();
    T next();
    default void remove() {
        throw new UnsupportedOperationException();
    }
}
```
通过这种方式，你可以减少无效的模板代码。实现Iterator接口的每一个类都不需要再声 明一个空的remove方法了，因为它现在已经有一个默认的实现。


### 行为的多继承

这是一种让类从多个来源重用代码的能力。

```java
public class ArrayList<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable,
                Serializable, Iterable<E>, Collection<E> {

}
```

由于Java 8中接口方法可以包含实现，类可以从多个接口中继承它们的行为（即实现的代码）。 
具体示例参见《Java8 实战》第九章 9.3.2节。

## 解决冲突的规则

(1) 类中的方法优先级最高。类或父类中声明的方法的优先级高于任何声明为默认方法的优 先级。

(2) 如果无法依据第一条进行判断，那么子接口的优先级更高：函数签名相同时，优先选择拥有最具体实现的默认方法的接口，即如果B继承了A，那么B就比A更加具体。

(3) 最后，如果还是无法判断，继承了多个接口的类必须通过显式覆盖和调用期望的方法，显式地选择使用哪一个默认方法的实现。 

问题：如果一个类同时实现了两个接口，这两个接口恰巧又提供了同样的默认方法签名，这时会发生什么情况？类会选择使用哪一个方法？

示例1：
```java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B extends A {
    default void hello() {
        System.out.println("Hello from B");
    }
}

public class C implements B, A {
    public static void main(String[] args) {
        new C().hello();
    }
}
```

打印输出：Hello from B

解析：按照规则(2)，应该选择的是提供了最具体实现的默认方法的接口。由于B比A更具体，所以应该选择B的hello方法。所以程序打印输出“Hello from B”

示例2：
```java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B extends A {
    default void hello() {
        System.out.println("Hello from B");
    }
}

public class D implements A {
}

public class C extends D implements B, A {
    public static void main(String[] args) {
        new C().hello();
    }
}
```

打印输出：Hello from B

解析：依据规则(1)，类中声明的方法具有更高的优先级。D并未覆盖hello方法，可是它实现了接 口A。所以它就拥有了接口A的默认方法。规则(2)说如果类或者父类没有对应的方法，那么就应 该选择提供了最具体实现的接口中的方法。因此，编译器会在接口A和接口B的hello方法之间做 选择。由于B更加具体，所以程序会再次打印输出“Hello from B”

示例3：
```java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B extends A {
    default void hello() {
        System.out.println("Hello from B");
    }
}

public class D implements A {
    @Override
    public void hello() {
        System.out.println("Hello from D");
    }
}

public class C extends D implements B, A {
    public static void main(String[] args) {
        new C().hello();
    }
}
```

打印输出：Hello from D

解析：由于依据规则(1)，父类中声明的方法具有更高的优先级，所以程序会打印输出“Hello from D”

示例4：D的声明如下：
```java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B extends A {
    default void hello() {
        System.out.println("Hello from B");
    }
}

public abstract class D implements A {
    public abstract void hello();
}

public class C extends D implements B, A {

    @Override
    public void hello() {
         System.out.println("Hello from C");
    }

    public static void main(String[] args) {
        new C().hello();
    }
}
```

打印输出：Hello from C

解析：虽然在结构上，其他的地方已经声明了默认方法的实现，C还是必须提供自己的hello方法。

示例5：假设B 不再继承A
```java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B {
    default void hello() {
        System.out.println("Hello from B");
    }
}

public class C implements B, A {
    public static void main(String[] args) {
        new C().hello();
    }
}
```

解析：这时规则(2)就无法进行判断了，因为从编译器的角度看没有哪一个接口的实现更加具体，两个都差不多。A接口和B接口的hello方法都是有效的选项。所以，Java编译器这时就会抛出一个 编译错误，因为它无法判断哪一个方法更合适：“Error: class C inherits unrelated defaults for hello() from types B and A.” 

冲突的解决

只能显式地决定你希望在C中使用哪一个方法。你可以覆盖类C中的hello方法，Java 8中引入了一种新的语法X.super.m(…)，其中X是你希望调用的m方法所在的父接口。

举例来说，如果你希望C使用来自于B的默认方法，它的调用方式看起来就如下所示：
```java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B {
    default void hello() {
        System.out.println("Hello from B");
    }
}

public class C implements B, A {
    @Override
    public void hello() {
        B.super.hello();
    }

    public static void main(String[] args) {
        new C().hello();
    }
}
```

示例6： 菱形继承问题 
```java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B extends A {
}

public interface C extends A {
}

public class D implements B, C {
    public static void main(String[] args) {
        new D().hello();
    }
}
```

打印输出：Hello from A

解析：这种情况下类D中的默认方法到底继承自什么地方 ——源自B的默认方法， 还是源自C的默认方法？实际上只有一个方法声明可以选择。只有A声明了一个默认方法。由于这 个接口是D的父接口，代码会打印输出“Hello from A”。 
 

 如果B中也提供了一个默认的hello方法，并且函数签名跟A 中的方法也完全一致，这时会发生什么情况呢？根据规则(2)，编译器会选择提供了更具体实现的 接口中的方法。由于B比A更加具体，所以编译器会选择B中声明的默认方法。如果B和C都使用相 同的函数签名声明了hello方法，就会出现冲突，正如我们之前所介绍的，你需要显式地指定使 用哪个方法。 


 示例7：如果你在C接口中添加一个抽象的hello方法（这次添加的不是一个默认方法）

```java
public interface C extends A {
     void hello();
}
```

解析：这个新添加到C接口中的抽象方法hello比由接口A继承而来的hello方法拥有更高的优先级，
因为C接口更加具体。因此，类D现在需要为hello显式地添加实现，否则该程序无法通过编译。

总结：

*   首先，类或父类中显式声明的方法，其优先级高于所有的默认方法。 
*   如果用第一条无法判断，方法签名又没有区别，那么选择提供最具体实现的默认方法的 接口。 
*   最后，如果冲突依旧无法解决，你就只能在你的类中覆盖该默认方法，显式地指定在你 的类中使用哪一个接口中的方法。 