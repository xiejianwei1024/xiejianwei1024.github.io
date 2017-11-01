---
layout: post
title: "java.lang.Object 源代码阅读"
---

java.lang.Object类作为类继承层次中的根。Obejct是所有类的父类。所有对象，包括数组，
都实现了Object 类中的方法。

## 1. Object 类的 toString() 方法

```java
public String toString() {
	return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

javadoc 中是这样描述的：

返回对象的字符串表示形式。一般来说，toString()方法返回一个“以文本方式表示”此对象的字符串。结果应该是一个简明扼要的内容，很容易让人阅读。建议所有子类覆盖此方法。

类Object的toString方法返回一个字符串，包含 对象从属于的类的名字，@符号，
对象的哈希码的无符号十六进制表示。换句话说，该方法返回一个等于下列值的字符串：

```java
getClass().getName() + '@' + Integer.toHexString(hashCode())
```

### 例子：

```java
@Test
public void toStringTest() {
	Object object = new Object();
	System.out.println(object.toString());
}
```

运行结果：java.lang.Object@3654919e

解析：返回对象的一个字符串表示：类的全名+“@”+16进制哈希码。

## 2. Object 类的 equals() 方法

```java
public boolean equals(Object obj) {
	return (this == obj);
}
```

javadoc 中是这样描述的：

表示 其他的对象是否和当前的对象相等。

equals方法在非空引用上实现了等价关系：
*   自反性：对于任何的非空引用 x，x.equals(x) 应该返回 true。  

*   对称性：对于任何的非空引用 x 和 y，x.equals(y) 为 true，那么 y.equals(x) 也为 true。

*   传递性：对于任何的非空引用 x, y 和 z, x.equals(y) 为 true，并且 y.equals(z) 为 true，那么 x.equals(z) 也应该为 true。

*   一致性：对于任何的非空引用 x 和 y，x.equals(y)的多次调用始终返回true或始终返回false(第一次调用为 true，那么 x.equals(y) 的第二次、第三次、第n次调用，也应该为 true)，前提是在比较之间没有修改 x 和 y。

*   对于任何的非空引用 x，x.equals(null) 返回 false。


### 例子：

```java
@Test
public void equalsTest() {
	Object object = new Object();
	Object object2 = new Object();
	System.out.println(object.equals(object2));
}
```

运行结果：false

解析：判断调用 equals() 方法的引用与传递进来参数的引用是否一致，即这两个引用是否指向同一个对象。它等价于 == 。