---
layout: post
title: "java.lang.Object 源代码阅读"
---

java.lang.Object类作为类继承层次中的根。Obejct是所有类的父类。所有对象，包括数组，
都实现了Object 类中的方法。

## 1. Object 类的 toString() 方法

```java
// Java code with syntax highlighting.
public String toString() {
	return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

javadoc 中是这样描述的：

返回对象的字符串表示形式。一般来说，toString方法返回一个“以文本方式表示”此对象的字符串。结果应该是一个简明扼要的内容，很容易让人阅读。建议所有子类覆盖此方法。

类Object的toString方法返回一个字符串，包含 对象从属于的类的名字，@符号，
对象的哈希码的无符号十六进制表示。换句话说，该方法返回一个等于下列值的字符串：