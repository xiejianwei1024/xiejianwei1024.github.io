---
layout: post
title: jdk8 函数式接口
---

## 1.什么是函数式接口呢？
----------------------------------------

函数式接口是Java8新增加的内容。如果一个接口<font color="#FF0000">只有一个抽象方法</font>，那么该接口就是函数式接口。


## 2.FunctionalInterface 注解
----------------------------------------
java.lang.FunctionalInterface（该注解位于java.lang包下）

让我们看看 javadoc 是怎么描述的？

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}
```

@FunctionalInterface是一种信息化的注解类型，表示声明的接口是一个函数式接口(由Java语言规范定义)。
在概念上，函数式接口只有一个抽象方法。由于默认方法有一个实现，所以它们不是抽象的。**如果一个接口声明了一个抽象方法，该抽象方法覆盖了 java.lang.Object 的 public 方法，那么不会计入接口抽象方法的个数。
因为任何接口的实现类都将直接或间接地继承自java.lang.Object。**

请注意，可以使用<font color="#FF0000">lambda表达式，方法引用或构造方法引用创建函数式接口的实例</font>。

使用此注解注释类型，编译器如果不满足下面两条就会生成错误消息：
*   被注释的类型是接口类型，不是注解，枚举，类。

*   被注释的类型必须满足函数式接口的要求。

然而，无论接口声明中是否存在FunctionalInterface注解，编译器都会将符合函数式接口定义的任何接口视为函数式接口。

总结：
*   **如果我们在某个接口上声明了FunctionalInterface注解，那么编译器会按照函数式接口的定义来要求该接口**。

*   **如果某个接口只有一个抽象方法，但我们并没有给该接口声明FunctionalInterface注解，那么编译器依旧会将该接口看作是函数式接口**。

## 3.常用的函数式接口
----------------------------------------
Java 8 新增加的函数式接口位于 java.util.function 包下。

### 1). Function（java.util.function.Function）

让我们看看 javadoc 是怎么描述的？

```java
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
 
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
  
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

表示一个函数，接受一个参数，产生一个结果。  
这是一个函数式接口，函数式方法是 `apply(Object)`。  
T - 输入类型  
R - 结果类型

----------------------------------------
抽象方法：
```java
R apply(T t);
```
将此函数应用于给定的参数。它接受一个泛型 T 的对象，返回一个泛型 R 的对象。

----------------------------------------
默认方法：
```java
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
}
```
返回一个组合函数，首先对输入参数应用 before 函数，然后对（before 函数得到的）结果（作为参数）应用当前函数。如果任何一个函数的计算抛出了异常，取决于组合函数的调用者。  
V - before 函数 和 组合函数 的输入类型。  
before - 在应用当前函数之前，所应用的函数。  
返回：一个组合函数，首先应用 before 函数，然后应用当前函数。

----------------------------------------
默认方法：
```java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
}
```
返回一个组合函数，首先对输入参数应用 当前函数，然后对（当前函数得到的）结果（作为参数）应用 after 函数。如果任何一个函数的计算抛出了异常，取决于组合函数的调用者。  
V - after 函数 和组合函数的输出类型。  
after - 在应用当前函数之后，所应用的函数。  
返回：一个组合函数，首先应用当前函数，然后应用 after 函数。

----------------------------------------
静态方法：
```java
static <T> Function<T, T> identity() { return t -> t; }
```
返回一个函数，总是返回输入参数。（同一性）  
T - 函数的输入和输出对象的类型。  
返回：一个函数，总是返回输入参数。

----------------------------------------
### 2). BiFunction（java.util.function.BiFunction）

让我们看看 javadoc 是怎么描述的？

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {

    R apply(T t, U u);

    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}
```
表示一个函数，接受两个参数，产生一个结果。这是 Function 的两个参数的一种特化形式。  
这是一个函数式接口，函数式方法是 `apply(Object, Object)`。  
T - 函数的第一个参数类型  
U - 函数的第二个参数类型  
R - 函数的结果类型

----------------------------------------
抽象方法：
```java
R apply(T t, U u);
```
对输入参数应用当前函数。它接受一个泛型 T 和一个泛型 U 的对象，返回一个泛型 R 的对象。

----------------------------------------
默认方法：
```java
default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t, U u) -> after.apply(apply(t, u));
}
```
返回一个组合函数，首先对输入参数应用当前函数，然后对（当前函数得到的）结果（作为参数）应用 after 函数。如果任何一个函数的计算抛出了异常，取决于组合函数的调用者。  
V - after 函数和组合函数的输出类型。  
after - 在应用当前函数之后，所应用的函数。  
返回：一个组合函数，首先应用当前函数，然后应用 after 函数。

----------------------------------------
思考一个问题：为什么 BiFunction 接口中 没有 compose 方法？  
如果有 compose 方法，那么参数一定是 BiFunction 类型的，而 compose 方法会先对输入应用 BiFunction 函数，会返回一个结果，然后再对结果应用当前的 BiFunction 函数，那么矛盾产生了。<font color="#FF0000">当前的 BiFuntion 函数的输入需要两个参数，而即将作为参数的结果只有一个。因为 BiFunction 函数接受两个参数，只返回一个结果。</font>

----------------------------------------
### 3). Predicate（java.util.function.Predicate）

让我们看看 javadoc 是怎么描述的？

```java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

表示一个参数的谓词（布尔值函数）。

这是一个函数式接口，函数式方法是 `test(Object)`。

T - Predicate 的输入类型。

----------------------------------------
抽象方法：
```java
boolean test(T t);
```
在给定的参数上计算当前的 Predicate。

t - 输入参数。

返回：如果输入参数匹配 Predicate，返回 true，否则返回 false。

----------------------------------------
默认方法：
```java
default Predicate<T> and(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) && other.test(t);
}
```
返回一个组合的 Predicate，表示当前的 Predicate 和另外一个 Predicate 短路的逻辑与。当计算组合的 Predicate 时，如果当前的 Predicate 返回 false，那么 other Predicate 不会被计算。

在计算任意一个 Predicate 期间抛出的任何异常都取决于调用者；如果计算当前的 Predicate 抛出了异常，那么将不会计算 other Predicate。

other - 一个 Predicate 将要和当前的 Predicate 进行逻辑与。

返回：一个组合的 Predicate，表示当前的 Predicate 和 other Predicate的短路逻辑与。

抛出 NullPointerException：如果 other 是 null。

----------------------------------------
默认方法：
```java
default Predicate<T> negate() {
    return (t) -> !test(t);
}
```
返回一个 Predicate，表示当前的 Predicate的逻辑否定。

----------------------------------------
默认方法：
```java
default Predicate<T> or(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) || other.test(t);
}
```
返回一个 Predicate，表示当前的 Predicate 和 另外一个 Predicate 的短路逻辑或。当计算组合的 Predicate时，如果当前的 Predicate 是 true，那么 other Predicate 不会被计算。

在计算任意一个 Predicate期间抛出的任何异常都取决于调用者。如果计算当前的 Predicate 抛出了异常，那么 other Predicate 将不会被计算。

other - 一个 Predicate 将要和当前的 Predicate 进行 逻辑或。

返回：一个组合的 Predicate，表示当前的 Predicate 和 other Predicate 的短路逻辑或。

抛出 NullPointerException：如果 other 是 null。

----------------------------------------
静态方法：
```java
static <T> Predicate<T> isEqual(Object targetRef) {
    return (null == targetRef)
            ? Objects::isNull
            : object -> targetRef.equals(object);
}
```
返回一个组合的 Predicate，根据 Objects 的 equals(Object, Object) 测试两个参数是否相等。

T - Predicate 的参数类型。

targetRef - 用于比较相等的对象引用，可以是 null。

返回：一个 Predicate，根据 Objects 的 equals(Object, Object) 测试两个参数是否相等。

----------------------------------------
### 4). Supplier（java.util.function.Supplier）

让我们看看 javadoc 是怎么描述的？

```java
@FunctionalInterface
public interface Supplier<T> {

    T get();
}
```

代表结果的供应者。

没有要求每次调用 Supplier 的时候都必须返回一个新的或者不同的结果。（在多次调用 Supplier 时，也可以返回相同的结果）

这是一个函数式接口，函数式方法是 `get()`。

T - Supplier 供应的结果类型。

----------------------------------------
抽象方法：
```java
T get();
```

得到一个结果。