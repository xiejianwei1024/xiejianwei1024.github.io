---
layout: post
title: jdk8-Spliterator
---

&emsp;&emsp;36_由一个最简单的使用流的示例剖析stream的底层源代码。<br/>

```java
public class StreamTest3 {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("hello", "world", "hello world");
        list.stream().forEach(System.out::println);
    }
}
```

&emsp;&emsp;debug之后，进入到Collection接口的stream()方法：<br/>

```java
/**
    * 返回一个串行流，用当前集合作为流的源。
    *
    * 当spliterator()方法不能返回(IMMUTABLE)不可变的、(CONCURRENT)并发的
    * 或(late-binding)延迟绑定的(spliterator)分割迭代器时，应该重写该方法。
    * (详细信息请参见spliterator()。)
    *
    * @implSpec
    * 默认的实现从集合的(Spliterator)分割迭代器创建了一个串行流。
    *
    * @return 当前集合中元素的串行流。
    * @since 1.8
    */
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}
```

&emsp;&emsp;继续跟进spliterator()方法，它也位于Collection接口中：<br/>

```java
/**
    * 创建了当前集合中元素的(Spliterator)分割迭代器。
    *
    * 实现应该记录(spliterator)分割迭代器报告的特征值。如果分割迭代器
    * 报告了Spliterator.SIZED并且当前集合不包含元素，则不需要报告这些特征值。
    *
    * 默认实现应该被子类重写，那么就能返回一个更高效的风格迭代器。
    * 为了保留对stream()和parallelStream()这两个方法的预期延迟行为，
    * 分割迭代器应该具有(IMMUTABLE)不可变的特征、或者(CONCURRENT)并发的特征、
    * 或者(late-binding)延迟绑定。如果这些都不实用，重写的类应该描述分割迭代器
    * 文档化的绑定和结构干扰策略，并且应该重写stream()和parallelStream()这两个方法,
    * 使用分割迭代器的Supplier创建流。如下所示：
    *
    *     Stream<E> s = StreamSupport.stream(() -> spliterator(), spliteratorCharacteristics)
    *
    * 这些要求确保了由stream()和parallelStream()这两个方法产生的流，将会
    * 在终止操作发起的时候，反映出集合的内容。
    *
    * @implSpec
    * 默认实现从当前集合的(Interator)迭代器创建了一个(late-binding)延迟绑定的分割迭代器。
    * 分割迭代器继承了集合迭代器的(fast-fail)快速失败的属性。
    *
    * 创建的(Spliterator)分割迭代器报告了Spliterator.SIZED固定代大小的。
    *
    * @implNote
    * 创建的(Spliterator)分割迭代器额外的报告了Spliterator.SUBSIZED。
    *
    * 如果分割迭代器没有覆盖任何元素，那么除了SIZED和SUBSIZED之外的其他特征值不能帮助客户端控制、专门化或简化计算。
    * 然而，这允许对空集合共享使用不可变的和空的分割迭代器(Spliterators.emptySpliterator())，
    * 并且允许客户端确定这样的分割迭代器是否不包含任何元素。
    *
    * @return 当前集合中元素的(Spliterator)分割迭代器。
    * @since 1.8
    */
@Override
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, 0);
}
```

&emsp;&emsp;跟进Spliterator，仔细理解：<br/>

```java
/**
 * 一个对象，用于遍历和分区源中的元素。Spliterator覆盖的元素的源
 * 可以是数组、集合、IO通道或生成器函数。 
 *
 * Spliterator可以一个一个单独地遍历元素(tryAdvance())，
 * 也可以按顺序以块的方式遍历元素(forEachRemaining())。
 *
 * Spliterator还可以将它的元素(使用trySplit)进行分区，
 * 成为另一个Spliterator，用于可能的并行操作。
 * 使用不能拆分或以高度不平衡或低效的方式拆分的Spliterator的操作，不太可能从并行性中获益。
 * 遍历和分割都会消耗元素；每一个分割迭代器只对单个块计算是有用的。
 *
 * Spliterator还报告了它的结构、源和元素的一组特征()，这些特征包括
 *  ORDERED, DISTINCT, SORTED, SIZED, NONNULL, IMMUTABLE, CONCURRENT, SUBSIZED。
 * 这些特征可能被Spliterator客户端用于控制、专门化或简化计算。
 * 例如，一个Collection的Spliterator会报告SIZED,
 * 一个Set的Spliterator会报告DISTINCT,一个SortedSet的Spliterator会报告SORTED。
 * 特征被报告为一个简单的统一的位操作。
 *
 * 一些特征还约束了方法行为;例如，如果是ORDERED的，遍历方法必须符合它们的文档顺序。
 * 将来可能会定义新的特性，所以实现者不应该给未列出的值赋予含义。
 *
 * 没有报告IMMUTABLE或者CONCURRENT的分割迭代器，应该由一个文档化的策略：
 * 当分割迭代器绑定到元素的源；并且在绑定后，对元素源的结构修改进行检测。
 * 
 * 一个延迟绑定的Spliterator是在第一次遍历、第一次拆分或第一次查询估计大小时绑定到元素源，
 * 而不是在分割迭代器创建的时候。一个不是延迟绑定的Spliterator是在分割迭代器创建的时候，
 * 或者是在分割迭代器中任意的方法首次调用的时候就绑定到元素源。
 * 在绑定之前对源所做的修改将在遍历Spliterator时反映出来。
 * 
 * 绑定Spliterator之后，如果检测到结构修改，应该尽最大努力抛出ConcurrentModificationException。
 * 这样做的spliterator称为快速失败。Spliterator的块遍历方法(forEachRemaining())
 * 可以在遍历所有元素之后优化遍历并检查结构干扰，而不是检查每个元素并立即失败。
 *
 * Spliterators可以通过estimateSize方法估计剩余元素的数量。理想情况下，
 * 正如在特征SIZED中所反映的那样，这个值与成功遍历中将遇到的元素数量完全对应。
 * 然而，即使在不完全知道的情况下，估计值对于在源上执行的操作仍然是有用的，
 * 比如帮助确定是进一步拆分还是按顺序遍历其余元素更好。
 *
 * 尽管spliterator在并行算法中有明显的实用价值，但它并不期望是线程安全的;
 * 相反，使用spliterator的并行算法实现应该确保spliterator一次只被一个线程使用。
 * 这通常很容易通过(serial thread-confinement)串行线程围栏来实现，
 * 而串行线程围栏这种模式通常是典型的并行算法的自然结果，这种算法通过递归解耦工作的。
 * 调用trySplit()的线程可能会将返回的Spliterator交给另一个线程，而另一个线程又可能
 * 遍历或进一步拆分该Spliterator。如果两个或多个线程在同一个spliterator上并发操作，
 * 则拆分和遍历的行为是未定义的。如果初始的线程将一个分割迭代器交给另一个线程去处理，
 * 最好在元素被tryAdvance()消费发生前切换，因为某些保证(比如estimateSize()的准确性，对分割迭代器SIZED的)
 * 仅仅在遍历开始之前是有效的。
 *
 * 37_分割迭代器与ForkJoin详解
 *
 * Spliterator的原生子类型特化提供了(OfInt)int, (OfLong)long, (OfDouble)double 值.
 * tryAdvance(Consumer)和forEachRemaining(Consumer)这两个方法子类型的默认实现
 * 将原生值包装成包装对象。这种装箱可能会破坏使用原始专门化所获得的性能优势。
 * 为了避免装箱，应该使用相应的基于原生方法。例如，应该优先使用Spliterator.OfInt.tryAdvance(IntConsumer)
 * 和 Spliterator.OfInt.forEachRemaining(IntConsumer)，而不是 Spliterator.OfInt.tryAdvance(Consumer)
 * 和Spliterator.OfInt.forEachRemaining(Consumer)。使用基于装箱的方法 tryAdvance() and forEachRemaining()
 * 遍历原生值不会影响装修后值的顺序。
 *
 * @apiNote
 * Spliterators和Iterators类型，用于遍历源中的元素。Spliterator的API被设计成
 * 支持串行遍历之外，还支持高效并行遍历，通过分解和单元素迭代。
 * 此外，通过Spliterator访问元素的协议被设计成对每个元素施加比Iterator更小的开销，
 * 并且避免了为hasNext()和next()使用单独的方法所涉及的固有竞争。
 *
 * 对于可变的源，如果在Spliterator绑定数据源和遍历结束期间，源被修改（元素添加，替换或者删除）的话，
 * 任意和不确定的行为可能会发生。例如，当使用流框架的时候，这样的修改将会产生任意和不确定的结果。
 *
 * 源的结构修改可按以下方法处理(大致按可取性下降的次序):
 * 
 * 源不能从结构上进行修改。
 * 例如，java.util.concurrent.CopyOnWriteArrayList的实例是一个不可变的源。
 * 从源创建的Spliterator报告IMMUTABLE的特性。
 *
 * 源管理并发修改。
 * 例如，java.util.concurrent.ConcurrentHashMap的key的set就是一个并发源。
 * 从源创建的Spliterator报告CONCURRENT的特性。
 * 
 * 可变的源提供了延迟绑定和快速失败的Spliterator。
 * 延迟绑定会缩小修改影响计算的窗口；快速失败在尽最大努力的基础上检测遍历开始后发生的结构修改，
 * 并抛出ConcurrentModificationException。例如，ArrayList和JDK中其他非并发的Collection，
 * 提供了延迟绑定，快速失败的spliterator。
 *
 * 可变的源提供了非延迟绑定，但是快速失败的Spliterator。
 * 由于潜在干扰的窗口较大，因此源增加了抛出ConcurrentModificationException的可能性。
 * 
 * 可变的源提供了延迟绑定，但是非快速失败的Spliterator。
 * 由于没有检测到修改，遍历后的源风险是任意的、不确定的行为。
 *
 * 可变的源提供了非延迟绑定，并且非快速失败的Spliterator。
 * 由于在构建之后可能会发生未检测到的修改，因此源增加了任意、不确定行为的风险。
 *
 * 例子。这里有一个类(除了数码之外，不是很有用)，它维护一个数组，其中实际数据保存在偶数位置，
 * 而不相关的标记数据保存在奇数位置。它的Spliterator忽略标记。
 *
 *
 * class TaggedArray<T> {
 *   private final Object[] elements; // immutable after construction
 *   TaggedArray(T[] data, Object[] tags) {
 *     int size = data.length;
 *     if (tags.length != size) throw new IllegalArgumentException();
 *     this.elements = new Object[2 * size];
 *     for (int i = 0, j = 0; i < size; ++i) {
 *       elements[j++] = data[i];
 *       elements[j++] = tags[i];
 *     }
 *   }
 *
 *   public Spliterator<T> spliterator() {
 *     return new TaggedArraySpliterator<>(elements, 0, elements.length);
 *   }
 *
 *   static class TaggedArraySpliterator<T> implements Spliterator<T> {
 *     private final Object[] array;
 *     private int origin; // current index, advanced on split or traversal
 *     private final int fence; // one past the greatest index
 *
 *     TaggedArraySpliterator(Object[] array, int origin, int fence) {
 *       this.array = array; this.origin = origin; this.fence = fence;
 *     }
 *
 *     public void forEachRemaining(Consumer<? super T> action) {
 *       for (; origin < fence; origin += 2)
 *         action.accept((T) array[origin]);
 *     }
 *
 *     public boolean tryAdvance(Consumer<? super T> action) {
 *       if (origin < fence) {
 *         action.accept((T) array[origin]);
 *         origin += 2; //真实数据在偶数位
 *         return true;
 *       }
 *       else // cannot advance
 *         return false;
 *     }
 *
 *     public Spliterator<T> trySplit() {
 *       int lo = origin; // divide range in half
 *       int mid = ((lo + fence) >>> 1) & ~1; // force midpoint to be even
 *       if (lo < mid) { // split out left half
 *         origin = mid; // reset this Spliterator's origin
 *         return new TaggedArraySpliterator<>(array, lo, mid);
 *       }
 *       else       // too small to split
 *         return null;
 *     }
 *
 *     public long estimateSize() {
 *       return (long)((fence - origin) / 2);
 *     }
 *
 *     public int characteristics() {
 *       return ORDERED | SIZED | IMMUTABLE | SUBSIZED;
 *     }
 *   }
 * }
 *
 * 例如一个并行计算框架，如java.util.stream包，将在并行计算中使用Spliterator，
 * 这里有一种实现相关并行forEach的方法，它演示了子任务分割的主要用法，
 * 直到估计的工作量足够小，可以按顺序执行为止。这里我们假设处理子任务的顺序无关紧要;
 * 不同的(分叉的)任务可以进一步拆分，并按未确定的顺序并发处理元素。
 * 本例使用java.util.concurrent.CountedCompleter;类似的用法也适用于其他并行任务结构。
 *
 *
 * static <T> void parEach(TaggedArray<T> a, Consumer<T> action) {
 *   Spliterator<T> s = a.spliterator();
 *   long targetBatchSize = s.estimateSize() / (ForkJoinPool.getCommonPoolParallelism() * 8);
 *   new ParEach(null, s, action, targetBatchSize).invoke();
 * }
 *
 * static class ParEach<T> extends CountedCompleter<Void> {
 *   final Spliterator<T> spliterator;
 *   final Consumer<T> action;
 *   final long targetBatchSize;
 *
 *   ParEach(ParEach<T> parent, Spliterator<T> spliterator,
 *           Consumer<T> action, long targetBatchSize) {
 *     super(parent);
 *     this.spliterator = spliterator; this.action = action;
 *     this.targetBatchSize = targetBatchSize;
 *   }
 *
 *   public void compute() {
 *     Spliterator<T> sub;
 *     while (spliterator.estimateSize() > targetBatchSize &&
 *            (sub = spliterator.trySplit()) != null) {
 *       addToPendingCount(1);
 *       new ParEach<>(this, sub, action, targetBatchSize).fork();
 *     }
 *     spliterator.forEachRemaining(action);
 *     propagateCompletion();
 *   }
 * }}</pre>
 *
 * @implNote
 * 如果布尔系统属性org.openjdk.java.util.stream.tripwireve被设置成true，
 * 当对原生子类型特化操作时，如果发生原生值装箱，然后会报告诊断警告。
 *
 * @param <T> 当前的Spliterator返回的元素类型。
 *
 * @see Collection
 * @since 1.8
 */
public interface Spliterator<T> {
    /**
     * 如果存在剩余的元素，那么对该元素执行给定的动作，返回true；否则返回false。
     * 如果当前的Spliterator是ORDERED有序的，那么将按相遇顺序对下一个元素执行操作。
     * 该动作引发的异常将传递给调用者。
     *
     * @param action 给定的动作
     * @return 如果在进入此方法时不存在其他元素，则为false，否则为true。
     * @throws NullPointerException 如果指定的action是null
     */
    boolean tryAdvance(Consumer<? super T> action);

    /**
     * 在当前线程中对剩余的元素执行给定的动作，以串行方式执行，知道所有的元素都被处理
     * 或者action动作抛出了异常。如果当前的Spliterator是ORDERED有序的，动作就会按照
     * 相遇的顺序处理。该动作引发的异常将传递给调用者。
     *
     * @implSpec
     * 默认实现反复调用tryAdvance，直到返回false。只要可能，就应该重写它。
     *
     * @param action 动作
     * @throws NullPointerException 如果指定的action是null
     */
    default void forEachRemaining(Consumer<? super T> action) {
        do { } while (tryAdvance(action));
    }

    /**
     * 如果当前的spliterator可以被分割，那么就会返回一个spliterator，它涵盖的元素If this spliterator can be partitioned, returns a Spliterator
     * 是从当前方法返回的，并且不会被当前的Spliterator覆盖掉。
     *
     * 如果当前的Spliterator是ORDERED有序的，那么返回的Spliterator必须涵盖元素的严格前缀。
     * (返回的Spliterator必须也是ORDERED的)
     *
     * 除非当前的Spliterator包含无穷多个元素，否则对trySplit()的重复调用最终必须返回null。
     * 非空返回：
     *
     * 分割之前estimateSize()报告的值必须大于等于分割之后estimateSize()报告的值,
     * 对于当前的Spliterator和返回的Spliterator。
     * 
     * 如果当前的Spliterator是SUBSIZED的，那么当前Spliterator的estimateSize()值在分割之前
     * 必须等于分割之后当前的加上返回的estimateSize()值总和。
     *
     * 当前的方法可能会因为任何原因返回null，包括空、遍历开始后无法分割、数据结构约束和效率考虑。
     *
     * @apiNote
     * 一个理想的trySplit方法有效地(不需要遍历)将其元素精确地分成两半，允许平衡的并行计算。
     * 许多偏离这一理想的做法仍然非常有效；例如，只近似地拆分一个近似平衡的树，
     * 或者对于叶子节点可能包含一个或两个元素的树，不能进一步拆分这些节点。
     * 然而，平衡上的较大偏差and/or效率过低的trySplit机制通常会导致较差的并行性能。
     *
     * @return 一个Spliterator，它涵盖了部分元素；或者返回null，如果当前的spliterator不能再分割。
     */
    Spliterator<T> trySplit();

    /**
     * 返回元素数量的估计值，是剩余遍历将遇到的元素。或者返回Long.MAX_VALUE,
     * (如果为无穷大、未知或计算成本太高)。
     *
     * 如果当前的Spliterator是SIZED的，并且没有被部分地遍历或分割，
     * 或者当前的Spliterator是SUBSIZED的，并且没有被部分地遍历，
     * 那么当前的估算值必须是元素的准确个数，元素是完整遍历所遇到的。
     * 否则，当前的估算值可能是任意的不准确的，但是必须但是必须随着trySplit调用的不同而减少。
     *
     * @apiNote
     * 即使是不精确的估计也常常是有用的，而且计算起来也很便宜。例如，
     * 例如，一个近似平衡的二叉树的子spliterator可能返回一个值，该值估计元素的数量是其父树的一半；
     * 如果根Spliterator没有维护精确的计数，它可以估计大小为与其最大深度对应的2的幂。
     *
     * @return 估算的值；或者Long.MAX_VALUE，如果为无穷大、未知或计算成本太高。
     */
    long estimateSize();

    /**
     * 便利的方法，它返回estimateSize()，如果当前的Spliterator是SIZED的，否则返回-1。
     * @implSpec
     * 默认实现返回estimateSize()的结果，如果Spliterator报告了SIZE特征值，否则返回-1。
     *
     * @return 如果知道，返回确定的size，否则返回-1。
     */
    default long getExactSizeIfKnown() {
        return (characteristics() & SIZED) == 0 ? -1L : estimateSize();
    }

    /**
     * 返回当前Spliterator及其元素的特征的集合set。结果被表示为ORed，值来自于
     * ORDERED, DISTINCT, SORTED, SIZED, NONNULL, IMMUTABLE, CONCURRENT,SUBSIZED。
     * 在给定的Spliterator上重复调用characteristics()的话，在调用trySplit之前或之间，
     * 应该总是返回相同的结果。
     *
     * 如果Spliterator报告了一组不一致的特征（从单个调用或跨多个调用返回的特征），
     * 则不能保证使用当前Spliterator进行任何计算。
     *
     * @apiNote 在分割之前，对于一个给定的spliterator的特征值可能和分割之后是不同的。
     * 具体示例，请参见特征值：SIZED，SUBSIZED，CONCURRENT。
     *
     * @return 特征的表示
     */
    int characteristics();

    /**
     * Returns {@code true} if this Spliterator's {@link
     * #characteristics} contain all of the given characteristics.
     *
     * @implSpec
     * The default implementation returns true if the corresponding bits
     * of the given characteristics are set.
     *
     * @param characteristics the characteristics to check for
     * @return {@code true} if all the specified characteristics are present,
     * else {@code false}
     */
    default boolean hasCharacteristics(int characteristics) {
        return (characteristics() & characteristics) == characteristics;
    }

    /**
     * If this Spliterator's source is {@link #SORTED} by a {@link Comparator},
     * returns that {@code Comparator}. If the source is {@code SORTED} in
     * {@linkplain Comparable natural order}, returns {@code null}.  Otherwise,
     * if the source is not {@code SORTED}, throws {@link IllegalStateException}.
     *
     * @implSpec
     * The default implementation always throws {@link IllegalStateException}.
     *
     * @return a Comparator, or {@code null} if the elements are sorted in the
     * natural order.
     * @throws IllegalStateException if the spliterator does not report
     *         a characteristic of {@code SORTED}.
     */
    default Comparator<? super T> getComparator() {
        throw new IllegalStateException();
    }

    /**
     * Characteristic value signifying that an encounter order is defined for
     * elements. If so, this Spliterator guarantees that method
     * {@link #trySplit} splits a strict prefix of elements, that method
     * {@link #tryAdvance} steps by one element in prefix order, and that
     * {@link #forEachRemaining} performs actions in encounter order.
     *
     * <p>A {@link Collection} has an encounter order if the corresponding
     * {@link Collection#iterator} documents an order. If so, the encounter
     * order is the same as the documented order. Otherwise, a collection does
     * not have an encounter order.
     *
     * @apiNote Encounter order is guaranteed to be ascending index order for
     * any {@link List}. But no order is guaranteed for hash-based collections
     * such as {@link HashSet}. Clients of a Spliterator that reports
     * {@code ORDERED} are expected to preserve ordering constraints in
     * non-commutative parallel computations.
     */
    public static final int ORDERED    = 0x00000010;

    /**
     * Characteristic value signifying that, for each pair of
     * encountered elements {@code x, y}, {@code !x.equals(y)}. This
     * applies for example, to a Spliterator based on a {@link Set}.
     */
    public static final int DISTINCT   = 0x00000001;

    /**
     * Characteristic value signifying that encounter order follows a defined
     * sort order. If so, method {@link #getComparator()} returns the associated
     * Comparator, or {@code null} if all elements are {@link Comparable} and
     * are sorted by their natural ordering.
     *
     * <p>A Spliterator that reports {@code SORTED} must also report
     * {@code ORDERED}.
     *
     * @apiNote The spliterators for {@code Collection} classes in the JDK that
     * implement {@link NavigableSet} or {@link SortedSet} report {@code SORTED}.
     */
    public static final int SORTED     = 0x00000004;

    /**
     * Characteristic value signifying that the value returned from
     * {@code estimateSize()} prior to traversal or splitting represents a
     * finite size that, in the absence of structural source modification,
     * represents an exact count of the number of elements that would be
     * encountered by a complete traversal.
     *
     * @apiNote Most Spliterators for Collections, that cover all elements of a
     * {@code Collection} report this characteristic. Sub-spliterators, such as
     * those for {@link HashSet}, that cover a sub-set of elements and
     * approximate their reported size do not.
     */
    public static final int SIZED      = 0x00000040;

    /**
     * Characteristic value signifying that the source guarantees that
     * encountered elements will not be {@code null}. (This applies,
     * for example, to most concurrent collections, queues, and maps.)
     */
    public static final int NONNULL    = 0x00000100;

    /**
     * Characteristic value signifying that the element source cannot be
     * structurally modified; that is, elements cannot be added, replaced, or
     * removed, so such changes cannot occur during traversal. A Spliterator
     * that does not report {@code IMMUTABLE} or {@code CONCURRENT} is expected
     * to have a documented policy (for example throwing
     * {@link ConcurrentModificationException}) concerning structural
     * interference detected during traversal.
     */
    public static final int IMMUTABLE  = 0x00000400;

    /**
     * Characteristic value signifying that the element source may be safely
     * concurrently modified (allowing additions, replacements, and/or removals)
     * by multiple threads without external synchronization. If so, the
     * Spliterator is expected to have a documented policy concerning the impact
     * of modifications during traversal.
     *
     * <p>A top-level Spliterator should not report both {@code CONCURRENT} and
     * {@code SIZED}, since the finite size, if known, may change if the source
     * is concurrently modified during traversal. Such a Spliterator is
     * inconsistent and no guarantees can be made about any computation using
     * that Spliterator. Sub-spliterators may report {@code SIZED} if the
     * sub-split size is known and additions or removals to the source are not
     * reflected when traversing.
     *
     * @apiNote Most concurrent collections maintain a consistency policy
     * guaranteeing accuracy with respect to elements present at the point of
     * Spliterator construction, but possibly not reflecting subsequent
     * additions or removals.
     */
    public static final int CONCURRENT = 0x00001000;

    /**
     * Characteristic value signifying that all Spliterators resulting from
     * {@code trySplit()} will be both {@link #SIZED} and {@link #SUBSIZED}.
     * (This means that all child Spliterators, whether direct or indirect, will
     * be {@code SIZED}.)
     *
     * <p>A Spliterator that does not report {@code SIZED} as required by
     * {@code SUBSIZED} is inconsistent and no guarantees can be made about any
     * computation using that Spliterator.
     *
     * @apiNote Some spliterators, such as the top-level spliterator for an
     * approximately balanced binary tree, will report {@code SIZED} but not
     * {@code SUBSIZED}, since it is common to know the size of the entire tree
     * but not the exact sizes of subtrees.
     */
    public static final int SUBSIZED = 0x00004000;

    /**
     * A Spliterator specialized for primitive values.
     *
     * @param <T> the type of elements returned by this Spliterator.  The
     * type must be a wrapper type for a primitive type, such as {@code Integer}
     * for the primitive {@code int} type.
     * @param <T_CONS> the type of primitive consumer.  The type must be a
     * primitive specialization of {@link java.util.function.Consumer} for
     * {@code T}, such as {@link java.util.function.IntConsumer} for
     * {@code Integer}.
     * @param <T_SPLITR> the type of primitive Spliterator.  The type must be
     * a primitive specialization of Spliterator for {@code T}, such as
     * {@link Spliterator.OfInt} for {@code Integer}.
     *
     * @see Spliterator.OfInt
     * @see Spliterator.OfLong
     * @see Spliterator.OfDouble
     * @since 1.8
     */
    public interface OfPrimitive<T, T_CONS, T_SPLITR extends Spliterator.OfPrimitive<T, T_CONS, T_SPLITR>>
            extends Spliterator<T> {
        @Override
        T_SPLITR trySplit();

        /**
         * If a remaining element exists, performs the given action on it,
         * returning {@code true}; else returns {@code false}.  If this
         * Spliterator is {@link #ORDERED} the action is performed on the
         * next element in encounter order.  Exceptions thrown by the
         * action are relayed to the caller.
         *
         * @param action The action
         * @return {@code false} if no remaining elements existed
         * upon entry to this method, else {@code true}.
         * @throws NullPointerException if the specified action is null
         */
        @SuppressWarnings("overloads")
        boolean tryAdvance(T_CONS action);

        /**
         * Performs the given action for each remaining element, sequentially in
         * the current thread, until all elements have been processed or the
         * action throws an exception.  If this Spliterator is {@link #ORDERED},
         * actions are performed in encounter order.  Exceptions thrown by the
         * action are relayed to the caller.
         *
         * @implSpec
         * The default implementation repeatedly invokes {@link #tryAdvance}
         * until it returns {@code false}.  It should be overridden whenever
         * possible.
         *
         * @param action The action
         * @throws NullPointerException if the specified action is null
         */
        @SuppressWarnings("overloads")
        default void forEachRemaining(T_CONS action) {
            do { } while (tryAdvance(action));
        }
    }

    /**
     * A Spliterator specialized for {@code int} values.
     * @since 1.8
     */
    public interface OfInt extends OfPrimitive<Integer, IntConsumer, OfInt> {

        @Override
        OfInt trySplit();

        @Override
        boolean tryAdvance(IntConsumer action);

        @Override
        default void forEachRemaining(IntConsumer action) {
            do { } while (tryAdvance(action));
        }

        /**
         * {@inheritDoc}
         * @implSpec
         * If the action is an instance of {@code IntConsumer} then it is cast
         * to {@code IntConsumer} and passed to
         * {@link #tryAdvance(java.util.function.IntConsumer)}; otherwise
         * the action is adapted to an instance of {@code IntConsumer}, by
         * boxing the argument of {@code IntConsumer}, and then passed to
         * {@link #tryAdvance(java.util.function.IntConsumer)}.
         */
        @Override
        default boolean tryAdvance(Consumer<? super Integer> action) {
            if (action instanceof IntConsumer) {
                return tryAdvance((IntConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfInt.tryAdvance((IntConsumer) action::accept)");
                return tryAdvance((IntConsumer) action::accept);
            }
        }

        /**
         * {@inheritDoc}
         * @implSpec
         * If the action is an instance of {@code IntConsumer} then it is cast
         * to {@code IntConsumer} and passed to
         * {@link #forEachRemaining(java.util.function.IntConsumer)}; otherwise
         * the action is adapted to an instance of {@code IntConsumer}, by
         * boxing the argument of {@code IntConsumer}, and then passed to
         * {@link #forEachRemaining(java.util.function.IntConsumer)}.
         */
        @Override
        default void forEachRemaining(Consumer<? super Integer> action) {
            if (action instanceof IntConsumer) {
                forEachRemaining((IntConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfInt.forEachRemaining((IntConsumer) action::accept)");
                forEachRemaining((IntConsumer) action::accept);
            }
        }
    }

    /**
     * A Spliterator specialized for {@code long} values.
     * @since 1.8
     */
    public interface OfLong extends OfPrimitive<Long, LongConsumer, OfLong> {

        @Override
        OfLong trySplit();

        @Override
        boolean tryAdvance(LongConsumer action);

        @Override
        default void forEachRemaining(LongConsumer action) {
            do { } while (tryAdvance(action));
        }

        /**
         * {@inheritDoc}
         * @implSpec
         * If the action is an instance of {@code LongConsumer} then it is cast
         * to {@code LongConsumer} and passed to
         * {@link #tryAdvance(java.util.function.LongConsumer)}; otherwise
         * the action is adapted to an instance of {@code LongConsumer}, by
         * boxing the argument of {@code LongConsumer}, and then passed to
         * {@link #tryAdvance(java.util.function.LongConsumer)}.
         */
        @Override
        default boolean tryAdvance(Consumer<? super Long> action) {
            if (action instanceof LongConsumer) {
                return tryAdvance((LongConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfLong.tryAdvance((LongConsumer) action::accept)");
                return tryAdvance((LongConsumer) action::accept);
            }
        }

        /**
         * {@inheritDoc}
         * @implSpec
         * If the action is an instance of {@code LongConsumer} then it is cast
         * to {@code LongConsumer} and passed to
         * {@link #forEachRemaining(java.util.function.LongConsumer)}; otherwise
         * the action is adapted to an instance of {@code LongConsumer}, by
         * boxing the argument of {@code LongConsumer}, and then passed to
         * {@link #forEachRemaining(java.util.function.LongConsumer)}.
         */
        @Override
        default void forEachRemaining(Consumer<? super Long> action) {
            if (action instanceof LongConsumer) {
                forEachRemaining((LongConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfLong.forEachRemaining((LongConsumer) action::accept)");
                forEachRemaining((LongConsumer) action::accept);
            }
        }
    }

    /**
     * A Spliterator specialized for {@code double} values.
     * @since 1.8
     */
    public interface OfDouble extends OfPrimitive<Double, DoubleConsumer, OfDouble> {

        @Override
        OfDouble trySplit();

        @Override
        boolean tryAdvance(DoubleConsumer action);

        @Override
        default void forEachRemaining(DoubleConsumer action) {
            do { } while (tryAdvance(action));
        }

        /**
         * {@inheritDoc}
         * @implSpec
         * If the action is an instance of {@code DoubleConsumer} then it is
         * cast to {@code DoubleConsumer} and passed to
         * {@link #tryAdvance(java.util.function.DoubleConsumer)}; otherwise
         * the action is adapted to an instance of {@code DoubleConsumer}, by
         * boxing the argument of {@code DoubleConsumer}, and then passed to
         * {@link #tryAdvance(java.util.function.DoubleConsumer)}.
         */
        @Override
        default boolean tryAdvance(Consumer<? super Double> action) {
            if (action instanceof DoubleConsumer) {
                return tryAdvance((DoubleConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfDouble.tryAdvance((DoubleConsumer) action::accept)");
                return tryAdvance((DoubleConsumer) action::accept);
            }
        }

        /**
         * {@inheritDoc}
         * @implSpec
         * If the action is an instance of {@code DoubleConsumer} then it is
         * cast to {@code DoubleConsumer} and passed to
         * {@link #forEachRemaining(java.util.function.DoubleConsumer)};
         * otherwise the action is adapted to an instance of
         * {@code DoubleConsumer}, by boxing the argument of
         * {@code DoubleConsumer}, and then passed to
         * {@link #forEachRemaining(java.util.function.DoubleConsumer)}.
         */
        @Override
        default void forEachRemaining(Consumer<? super Double> action) {
            if (action instanceof DoubleConsumer) {
                forEachRemaining((DoubleConsumer) action);
            }
            else {
                if (Tripwire.ENABLED)
                    Tripwire.trip(getClass(),
                                  "{0} calling Spliterator.OfDouble.forEachRemaining((DoubleConsumer) action::accept)");
                forEachRemaining((DoubleConsumer) action::accept);
            }
        }
    }
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
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
