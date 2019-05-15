---
layout: post
title: jdk8-Optional
---
## java.util.Optional 类 javadoc 解读

```java

package java.util;

import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

/**
 * 一个容器对象，可能包含也可能不包含非空（non-null）值。如果值是存在的，那么isPresent()返回 true，
 * 并且get ()返回这个值。
 *
 * 还提供了其他方法,这些方法依赖于包含值的存在与否，比如orElse()(如果值不存在，则返回默认值)
 * 和ifPresent()(如果值存在，则执行代码块)。  
 *
 * 这是一个value-based class（基于值的类）；在Optional的实例上使用标识敏感操作(包括引用相等(==)、
 * 标识哈希码或同步)可能会产生不可预测的结果，应该避免。
 *
 * @since 1.8
 */
public final class Optional<T> {
    /**
     * empty()的通用实例
     */
    private static final Optional<?> EMPTY = new Optional<>();

    /**
     * 如果non-null, the value; 如果是null, 表示没有值存在。
     */
    private final T value;

    /**
     * 构造一个空的实例。
     *
     * @implNote 通常每个VM只应该存在一个空实例EMPTY。
     */
    private Optional() {
        this.value = null;
    }

    /**
     * 返回一个空的Optional实例。 这个Optional没有值。
     *
     * @apiNote 虽然这样做很有诱惑力，但是用==比较Option.empty()返回的实例，来避免测试对象是否为空。
     * 不能保证它是单例的。相反,使用isPresent ()。
     *
     * @param <T> （non-existent）不存在的值的类型
     * @return 一个空的 Optional
     */
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    /**
     * 构造一个有值的实例。
     *
     * @param value 非空（non-null）值
     * @throws NullPointerException 如果 value 是 null，则抛出 NullPointerException 
     */
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }

    /**
     * 返回一个Optional，它具有指定的非空（non-null）值。
     *
     * @param <T> value的类型
     * @param value value 是存在的, 必须是非空的（non-null）
     * @return 一个有值的Optional
     * @throws NullPointerException 如果 value 是null，则抛出NullPointerException
     */
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

     /**
     * 如果给定的值是非空的（non-null），返回一个有值的Optional，否则返回一个空的Optional。
     *
     * @param <T>  value的类型
     * @param value 可能为空的（possibly-null）值
     * @return 如果给定的值是非空的（non-null），返回一个有值的Optional，否则返回一个空的Optional。
     */
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }

     /**
     * 如果Optional中的值是存在的，返回这个值，否则抛出NoSuchElementException。
     *
     * @return这个 Optional所持有的非空（non-null）值。
     * @throws 如果没有值存在，则抛出NoSuchElementException
     *
     * @see Optional#isPresent()
     */
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }

    /**
     * 如果值是存在的，返回true，否则返回false。
     *
     * 如果值是存在的，返回true，否则返回false。
     */
    public boolean isPresent() {
        return value != null;
    }

    /**
     * 如果值是存在的，那么对这个值调用指定的Consumer函数，否则什么也不做。
     *
     * @param consumer如果值是存在的话，则执行 consumer块。
     * @throws NullPointerException 如果值是存在的，并且consumer是null，则抛出NullPointerException。
     */
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }

    /**
     * If a value is present, and the value matches the given predicate,
     * return an {@code Optional} describing the value, otherwise return an
     * empty {@code Optional}.
     *
     * @param predicate a predicate to apply to the value, if present
     * @return an {@code Optional} describing the value of this {@code Optional}
     * if a value is present and the value matches the given predicate,
     * otherwise an empty {@code Optional}
     * @throws NullPointerException if the predicate is null
     */
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }

    /**
     * If a value is present, apply the provided mapping function to it,
     * and if the result is non-null, return an {@code Optional} describing the
     * result.  Otherwise return an empty {@code Optional}.
     *
     * @apiNote This method supports post-processing on optional values, without
     * the need to explicitly check for a return status.  For example, the
     * following code traverses a stream of file names, selects one that has
     * not yet been processed, and then opens that file, returning an
     * {@code Optional<FileInputStream>}:
     *
     * <pre>{@code
     *     Optional<FileInputStream> fis =
     *         names.stream().filter(name -> !isProcessedYet(name))
     *                       .findFirst()
     *                       .map(name -> new FileInputStream(name));
     * }</pre>
     *
     * Here, {@code findFirst} returns an {@code Optional<String>}, and then
     * {@code map} returns an {@code Optional<FileInputStream>} for the desired
     * file if one exists.
     *
     * @param <U> The type of the result of the mapping function
     * @param mapper a mapping function to apply to the value, if present
     * @return an {@code Optional} describing the result of applying a mapping
     * function to the value of this {@code Optional}, if a value is present,
     * otherwise an empty {@code Optional}
     * @throws NullPointerException if the mapping function is null
     */
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }

    /**
     * If a value is present, apply the provided {@code Optional}-bearing
     * mapping function to it, return that result, otherwise return an empty
     * {@code Optional}.  This method is similar to {@link #map(Function)},
     * but the provided mapper is one whose result is already an {@code Optional},
     * and if invoked, {@code flatMap} does not wrap it with an additional
     * {@code Optional}.
     *
     * @param <U> The type parameter to the {@code Optional} returned by
     * @param mapper a mapping function to apply to the value, if present
     *           the mapping function
     * @return the result of applying an {@code Optional}-bearing mapping
     * function to the value of this {@code Optional}, if a value is present,
     * otherwise an empty {@code Optional}
     * @throws NullPointerException if the mapping function is null or returns
     * a null result
     */
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }

    /**
     * 如果 value 值存在，那么返回 value，否则返回 ohter。
     *
     * @param other 如果没有值（value）存在，这个other 被返回，它可能是 null。
     * @return 如果 value 值存在，那么返回 value，否则返回 ohter。
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }

    /**
     * 如果 value 值存在，那么返回 value，否则调用 other，返回调用结果。
     *
     * @param other 如果没有值存在，Supplier函数的结果被返回
     * @return 如果 value 值存在，那么返回 value，否则返回调用other.get()的结果。
     * @throws NullPointerException 如果value不存在，并且 other 是 null。
     */
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }

    /**
     * Return the contained value, if present, otherwise throw an exception
     * to be created by the provided supplier.
     *
     * @apiNote A method reference to the exception constructor with an empty
     * argument list can be used as the supplier. For example,
     * {@code IllegalStateException::new}
     *
     * @param <X> Type of the exception to be thrown
     * @param exceptionSupplier The supplier which will return the exception to
     * be thrown
     * @return the present value
     * @throws X if there is no value present
     * @throws NullPointerException if no value is present and
     * {@code exceptionSupplier} is null
     */
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }

    /**
     * Indicates whether some other object is "equal to" this Optional. The
     * other object is considered equal if:
     * <ul>
     * <li>it is also an {@code Optional} and;
     * <li>both instances have no value present or;
     * <li>the present values are "equal to" each other via {@code equals()}.
     * </ul>
     *
     * @param obj an object to be tested for equality
     * @return {code true} if the other object is "equal to" this object
     * otherwise {@code false}
     */
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Optional)) {
            return false;
        }

        Optional<?> other = (Optional<?>) obj;
        return Objects.equals(value, other.value);
    }

    /**
     * Returns the hash code value of the present value, if any, or 0 (zero) if
     * no value is present.
     *
     * @return hash code value of the present value or 0 if no value is present
     */
    @Override
    public int hashCode() {
        return Objects.hashCode(value);
    }

    /**
     * Returns a non-empty string representation of this Optional suitable for
     * debugging. The exact presentation format is unspecified and may vary
     * between implementations and versions.
     *
     * @implSpec If a value is present the result must include its string
     * representation in the result. Empty and present Optionals must be
     * unambiguously differentiable.
     *
     * @return the string representation of this instance
     */
    @Override
    public String toString() {
        return value != null
            ? String.format("Optional[%s]", value)
            : "Optional.empty";
    }
}

```
## 为何要引入 Optional？
jdk8 引入 Optional 类的目的是为了解决NPE（NullPointerException）。<br/>

## 如何创建 Optional 实例？
创建Optional实例不能直接new，因为其构造方法都是私有的；我们可以通过API提供的静态工厂方法：
*   empty():Optional<T>
*   of(T) :Optional<T>
*   ofNullable(T) :Optional<T>

我们需要知道这三个静态工厂方法的意义，在javadoc中有详细的描述，这里简单描述下：
*   empty()：创建一个空的Optional对象，即Optional包装的value是null。
*   of(T)：创建一个非空值的Optional对象，即Optional包装的value是non-null的，是存在的。
*   ofNullable(T)：如果参数是null，那么创建一个空的Optional对象；如果参数是non-null的，那么创建一个非空值的Optional对象。

如何使用它们呢？

假如，我们从数据库中查找数据，这时的返回值有可能是空的，也有可能不是空的，那么我们应该使用第三个静态工厂方法：ofNullable(T)。<br/>
如果，你确信，值是空的，那么使用empty()；<br/>
如果，你确信，值是非空的，那么使用of(T)；<br/>

## 具体示例
### 示例1：OptionalTest
第一个例子：OptionalTest.java，主要介绍了Optional中API简单使用。<br/>
```java
import java.util.Optional;

public class OptionalTest {
    public static void main(String[] args) {
//        Optional<String> optional = Optional.of("hello");
        Optional<String> optional = Optional.empty();

//        //面向对象的编程风格，不推荐使用
//        if(optional.isPresent()) {
//            System.out.println(optional.get());
//        }
        //推荐的Optional使用方式，函数式风格
        optional.ifPresent(item -> System.out.println(item));
        System.out.println("---");
        System.out.println(optional.orElse("world"));
        System.out.println("---");
        System.out.println(optional.orElseGet(() -> "nihao"));
    }
}
```

### 示例2：OptionalTest2
第二个例子：实际开发中经常使用。该实例是一对多的关系，一个公司有多个员工。
Employee.java 和 Company.java 和 OptionalTest2.java

Employee.java 如下：<br/>
```java
public class Employee {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

Company.java 如下：<br/>
```java
import java.util.List;

public class Company {
    private String name;

    private List<Employee> employees;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Employee> getEmployees() {
        return employees;
    }

    public void setEmployees(List<Employee> employees) {
        this.employees = employees;
    }
}
```
OptionalTest2.java 如下：<br/>
```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.Optional;

public class OptionalTest2 {
    public static void main(String[] args) {
        Employee employee1 = new Employee();
        employee1.setName("zhangsan");
        Employee employee2 = new Employee();
        employee2.setName("lisi");

        Company company = new Company();
        company.setName("company1");

        List<Employee> employees = Arrays.asList(employee1, employee2);
        company.setEmployees(employees);

        // 取出Company中的Employee列表，如果有，那么返回该Employee列表；
        // 如果没有，那么返回一个空的Employee列表。
        // 最佳实践
        Optional<Company> optional = Optional.ofNullable(company);
        List<Employee> result = optional.map(theCompany -> theCompany.getEmployees())
                                        .orElse(Collections.emptyList());
        result.forEach(employee -> System.out.println(employee.getName()));

    }

    // IDEA 警告
    private Optional<Company> c;

    // IDEA 警告
    public void test(Optional optional) {}
    
}
```
## Optional 用作返回类型，否则 IDEA 警告
如果在OptionalTest2.java中定义Optional类型的成员变量或者定义的方法参数类型是Optional的,如下：<br/>

```java
private Optional<Company> c;

public void test(Optional optional) {}
```

IDEA会给出警告：

'Optional' used as type for parameter 'optional' less... (Ctrl+F1)

Reports any uses of java.util.Optional<T>, java.util.OptionalDouble, java.util.OptionalInt, java.util.OptionalLong or com.google.common.base.Optional as the type for a field or a parameter. Optional was designed to provide a limited mechanism for library method return types where there needed to be a clear way to represent "no result". Using a field with type java.util.Optional is also problematic if the class needs to be Serializable, which java.util.Optional is not.

大意：对成员变量或者方法参数使用这些类型做出了报告。这些类型包括java.util.Optional<T>, java.util.OptionalDouble, java.util.OptionalInt, java.util.OptionalLong or com.google.common.base.Optional。
Optional的设计目的是为库方法返回类型提供一种有限的机制，在这种情况下需要一种明确的方法来表示“no result”。
如果一个类需要被序列化，那么用java.util.Optional作为成员变量的类型也是有问题的，因为java.util.Optional没有被序列化。

反观，String类，明确实现了java.io.Serializable接口：
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {}
```

所以，Optional是用来解决空指针异常才被引入的，表示“no result”，常用作返回值类型。
不应该将Optional声明为成员变量，不应该将Optional用作方法的参数。

