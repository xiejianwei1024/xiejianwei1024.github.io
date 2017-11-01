---
layout: post
title: java.util.ArrayList 源代码阅读
---

ArrayList是List类的一个典型的实现，是基于数组实现的List类，因此，ArrayList封装了一个动态的、可变长度的Object[]数组。ArrayList是通过initialCapacity参数来设置数组长度的，当向ArrayList添加的数据超出了ArrayList的长度之后，initialCapacity会自动增加。  

***  

## 私有属性

ArrayList定义了两个私有属性： 

```java
//elementData存储ArrayList内的元素，size表示它包含的元素的数量。
private transient Object[] elementData;
private int size;
```

其中有一个关键字：transient：Java的serialization提供了一种持久化对象实例的机制。当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。  

```java
import java.io.Serializable;

public class UserInfo implements Serializable {
	private static final long serialVersionUID = 996890129747019948L;
	private String name;
	private transient String psw;

	public UserInfo(String name, String psw) {
		this.name = name;
		this.psw = psw;
	}

	public String toString() {
		return "name=" + name + ", psw=" + psw;
	}
}

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class TestTransient {  
     public static void main(String[] args) {  
         UserInfo userInfo = new UserInfo("张三", "123456");  
         System.out.println(userInfo);  
         try {  
             // 序列化，被设置为transient的属性没有被序列化  
             ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream(  
                     "UserInfo.out"));  
             o.writeObject(userInfo);  
             o.close();  
         } catch (Exception e) {  
             // TODO: handle exception  
             e.printStackTrace();  
         }  
         try {  
             // 重新读取内容  
             ObjectInputStream in = new ObjectInputStream(new FileInputStream(  
                     "UserInfo.out"));  
             UserInfo readUserInfo = (UserInfo) in.readObject();  
             //读取后psw的内容为null  
             System.out.println(readUserInfo.toString());  
         } catch (Exception e) {  
             // TODO: handle exception  
             e.printStackTrace();  
         }  
     }  
 }
```

运行结果：  
name=张三, psw=123456  

name=张三, psw=null

## 构造方法

ArrayList提供了三种方式的构造器。

```java
//ArrayList带容量大小的构造函数。
public ArrayList(int initialCapacity) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+initialCapacity);
    this.elementData = new Object[initialCapacity];
}
//ArrayList无参数构造参数,这时不会分配容量。
public ArrayList() {
    super();
    this.elementData = EMPTY_ELEMENTDATA;
}
//创建一个包含collection的ArrayList
public ArrayList(Collection<? extends E> c) {
	//调用toArray()方法把collection转换成数组
    elementData = c.toArray();
    //把数组的长度赋值给ArrayList的size属性
    size = elementData.length;
    // c.toArray might (incorrectly) not return Object[] (see 6260652)
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```

在这有一个地方需要注意下，就是在JDK1.6中无参数的构造方法是这么写的：

```java
//ArrayList无参构造函数。默认容量是10。
public ArrayList() {
    this(10);
}
```

## ArrayList的动态扩容（核心）

当ArrayList进行add操作的时候，如果添加的元素超出了数组的长度，怎么办？

```java
//在列表的结尾附加指定的元素
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

add方法会去调用下面的方法，根据传入的最小需要容量minCapacity来和数组的容量长度对比，若minCapactity大于或等于数组容量，则需要进行扩容。

```java
//确保列表的容量足够
private void ensureCapacityInternal(int minCapacity)
	//向一个空的列表中添加元素，最小容量为默认容量 10
    if (elementData == EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
//确保列表的容量
private void ensureExplicitCapacity(int minCapacity) {
	//修改次数+1
    modCount++;

    // overflow-conscious code
    //是否扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

扩容的时候会去调用grow()方法来进行动态扩容，在grow中采用了位运算，我们知道位运算的速度远远快于整除运算：

```java
//最大数组容量
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
//这才是动态扩展的精髓，看到这个方法，ArrayList瞬间被打回原形
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    //首先得到数组的旧容量，然后进行oldCapacity + (oldCapacity >> 1)，将oldCapacity 右移一位，其效果相当于oldCapacity /2，整句的结果就是设置新数组的容量扩展为原来数组的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //再判断一下新数组的容量够不够，够了就直接使用这个长度创建新数组， 
    //不够就将数组长度设置为需要的长度
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //判断有没超过最大限制，如果超出限制则调用hugeCapacity
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    //将原来数组的值copy新数组中去， ArrayList的引用指向新数组
    //这儿会新创建数组，如果数据量很大，重复的创建的数组，那么还是会影响效率，
    //因此鼓励在合适的时候通过构造方法指定默认的capaticy大小
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

有一点需要注意的是，容量拓展，是创建一个新的数组，然后将旧数组上的数组copy到新数组，这是一个很大的消耗，所以在我们使用ArrayList时，最好能预计数据的大小，在第一次创建时就申请够内存。  

***  

看一下JDK1.6的动态扩容的实现原理：

```java
public void ensureCapacity(int minCapacity) {
    modCount++;
    int oldCapacity = elementData.length;
    if (minCapacity > oldCapacity) {
        Object oldData[] = elementData;
        int newCapacity = (oldCapacity * 3)/2 + 1;
        if (newCapacity < minCapacity)
            newCapacity = minCapacity;
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
```

从代码上，我们可以看出区别： 
第一：在容量进行扩展的时候，其实例如整除运算将容量扩展为原来的1.5倍加1，而jdk1.7是利用位运算，从效率上，jdk1.7就要快于jdk1.6。 

第二：在算出newCapacity时，其没有和ArrayList所定义的
MAX_ARRAY_SIZE作比较，为什么没有进行比较呢，原因是jdk1.6没有定义这个MAX_ARRAY_SIZE
最大容量，也就是说，其没有最大容量限制的，但是jdk1.7做了一个改进，进行了容量限制。