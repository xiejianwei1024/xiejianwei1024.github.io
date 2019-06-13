---
layout: post
title: jdk8-Spliterators
---

&emsp;&emsp;40_流源构造代码分析<br/>
&emsp;&emsp;笔者在上一篇文章中分析了Spliterator的源码，完成后，回到debug的那个方法中，它位于ArrayList in Arrays中：<br/>

```java
@Override
public Spliterator<E> spliterator() {
    return Spliterators.spliterator(a, Spliterator.ORDERED);
}
```

&emsp;&emsp;我们知道Collection是集合的顶层接口，Collection接口中定义了default spliterator()方法，有着丰富的javadoc说明：<br/>

```java
@Override
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, 0);
}
```

&emsp;&emsp;我们还是先从Collection接口中入手，继续跟进Spliterators的spliterator()：<br/>

```java
// Iterator-based spliterators

/**
    * 创建了一个Spliterator，使用给定集合的java.util.Collection.iterator()作为元素的源。
    * 并且报告报告了java.util.Collection.size()作为Spliterator的初始化大小。
    *
    * 来这个spliterator是延迟绑定的，继承了集合迭代器的快速失败的属性，并且实现了trySplit
    * 以允许有限的并行化。
    *
    * @param <T> 元素的类型
    * @param c 当前的集合
    * @param characteristics 当前分割迭代器的源或元素的特征。除非提供CONCURRENT特征值，
    *        否则就会报告SIZED 和 SUBSIZED。
    * @return 从迭代器返回的一个分割迭代器
    * @throws NullPointerException 如果给定的集合是null
    */
public static <T> Spliterator<T> spliterator(Collection<? extends T> c,
                                                int characteristics) {
    return new IteratorSpliterator<>(Objects.requireNonNull(c),
                                        characteristics);
}
```

&emsp;&emsp;继续跟进IteratorSpliterator，它是Spliterators的静态类：<br/>
```java
// Iterator-based Spliterators

/**
    * 这种分割迭代器使用给定的迭代器对元素进行操作。
    * 这种分割迭代器实现了trySplit以允许有限的并行化。
    */
static class IteratorSpliterator<T> implements Spliterator<T> {
    static final int BATCH_UNIT = 1 << 10;  // batch array size increment
    static final int MAX_BATCH = 1 << 25;  // max batch array size;
    private final Collection<? extends T> collection; // null OK
    private Iterator<? extends T> it;
    private final int characteristics;
    private long est;             // size estimate
    private int batch;            // batch size for splits

    /**
        * 创建一个分割迭代器，使用给定集合的java.util.Collection.iterator()用于遍历，
        * 并且报告了java.util.Collection.size()作为分割迭代器的初始化大小。
        *
        * @param c 集合
        * @param characteristics 当前分割迭代器的源或元素的属性值
        */
    public IteratorSpliterator(Collection<? extends T> collection, int characteristics) {
        this.collection = collection;
        this.it = null;
        this.characteristics = (characteristics & Spliterator.CONCURRENT) == 0
                                ? characteristics | Spliterator.SIZED | Spliterator.SUBSIZED
                                : characteristics;
    }

    /**
        * Creates a spliterator using the given iterator
        * for traversal, and reporting the given initial size
        * and characteristics.
        *
        * @param iterator the iterator for the source
        * @param size the number of elements in the source
        * @param characteristics properties of this spliterator's
        * source or elements.
        */
    public IteratorSpliterator(Iterator<? extends T> iterator, long size, int characteristics) {
        this.collection = null;
        this.it = iterator;
        this.est = size;
        this.characteristics = (characteristics & Spliterator.CONCURRENT) == 0
                                ? characteristics | Spliterator.SIZED | Spliterator.SUBSIZED
                                : characteristics;
    }

    /**
        * Creates a spliterator using the given iterator
        * for traversal, and reporting the given initial size
        * and characteristics.
        *
        * @param iterator the iterator for the source
        * @param characteristics properties of this spliterator's
        * source or elements.
        */
    public IteratorSpliterator(Iterator<? extends T> iterator, int characteristics) {
        this.collection = null;
        this.it = iterator;
        this.est = Long.MAX_VALUE;
        this.characteristics = characteristics & ~(Spliterator.SIZED | Spliterator.SUBSIZED);
    }

    @Override
    public Spliterator<T> trySplit() {
        /*
            * Split into arrays of arithmetically increasing batch
            * sizes.  This will only improve parallel performance if
            * per-element Consumer actions are more costly than
            * transferring them into an array.  The use of an
            * arithmetic progression in split sizes provides overhead
            * vs parallelism bounds that do not particularly favor or
            * penalize cases of lightweight vs heavyweight element
            * operations, across combinations of #elements vs #cores,
            * whether or not either are known.  We generate
            * O(sqrt(#elements)) splits, allowing O(sqrt(#cores))
            * potential speedup.
            */
        Iterator<? extends T> i;
        long s;
        if ((i = it) == null) {
            i = it = collection.iterator();
            s = est = (long) collection.size();
        }
        else
            s = est;
        if (s > 1 && i.hasNext()) {
            int n = batch + BATCH_UNIT;
            if (n > s)
                n = (int) s;
            if (n > MAX_BATCH)
                n = MAX_BATCH;
            Object[] a = new Object[n];
            int j = 0;
            do { a[j] = i.next(); } while (++j < n && i.hasNext());
            batch = j;
            if (est != Long.MAX_VALUE)
                est -= j;
            return new ArraySpliterator<>(a, 0, j, characteristics);
        }
        return null;
    }

    @Override
    public void forEachRemaining(Consumer<? super T> action) {
        if (action == null) throw new NullPointerException();
        Iterator<? extends T> i;
        if ((i = it) == null) {
            i = it = collection.iterator();
            est = (long)collection.size();
        }
        i.forEachRemaining(action);
    }

    @Override
    public boolean tryAdvance(Consumer<? super T> action) {
        if (action == null) throw new NullPointerException();
        if (it == null) {
            it = collection.iterator();
            est = (long) collection.size();
        }
        if (it.hasNext()) {
            action.accept(it.next());
            return true;
        }
        return false;
    }

    @Override
    public long estimateSize() {
        if (it == null) {
            it = collection.iterator();
            return est = (long)collection.size();
        }
        return est;
    }

    @Override
    public int characteristics() { return characteristics; }

    @Override
    public Comparator<? super T> getComparator() {
        if (hasCharacteristics(Spliterator.SORTED))
            return null;
        throw new IllegalStateException();
    }
}
```


&emsp;&emsp;返回Spliterator对象后，就要执行最外层的语句了：<br/>

```java
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}
```

&emsp;&emsp;跟进去stream()，它位于StreamSupport类下：<br/>

```java
/**
     * 创建一个新的串行或者并行的Stream从一个Spliterator。
     *
     * 只有在流管道的终端操作开始后，这个spliterator才会遍历、拆分或查询估计的大小。
     *
     * 强烈建议这个spliterator报告IMMUTABLE或者CONCURRENT特征值，或者是延迟绑定的。
     * 否则，应该使用stream(Supplier, int, boolean)来减少对源的潜在修改。
     *
     * @param <T> the type of stream elements
     * @param spliterator a {@code Spliterator} describing the stream elements
     * @param parallel if {@code true} then the returned stream is a parallel
     *        stream; if {@code false} the returned stream is a sequential
     *        stream.
     * @return a new sequential or parallel {@code Stream}
     */
public static <T> Stream<T> stream(Spliterator<T> spliterator, boolean parallel) {
    Objects.requireNonNull(spliterator);
    return new ReferencePipeline.Head<>(spliterator,
                                        StreamOpFlag.fromCharacteristics(spliterator),
                                        parallel);
}
```

&emsp;&emsp;这里，我们有必要阅读StreamSupport类的javadoc：<br/>

```java
/**
 * 底层辅助方法，用于创建和操作流。
 *
 * 这个类主要是为库作者提供数据结构的流视图;大多数面向最终用户的静态流方法都位于不同的Stream类中。
 *
 * @since 1.8
 */
public final class StreamSupport {...}
```

&emsp;&emsp;接下来，我们跟进ReferencePipeline：<br/>

```java
/**
 * 中间管道阶段或管道源阶段实现的基类，其元素类型为U。
 *
 * @param <P_IN> 上游源中的元素类型
 * @param <P_OUT> 在当前阶段生成的元素类型
 *
 * @since 1.8
 */
abstract class ReferencePipeline<P_IN, P_OUT>
        extends AbstractPipeline<P_IN, P_OUT, Stream<P_OUT>>
        implements Stream<P_OUT>  {

    /**
     * ReferencePipeline的源阶段。
     *
     * @param <E_IN> 上游源中的元素类型
     * @param <E_OUT> 在当前阶段生成的元素类型
     * @since 1.8
     */
    static class Head<E_IN, E_OUT> extends ReferencePipeline<E_IN, E_OUT> {}            

}
```

&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>