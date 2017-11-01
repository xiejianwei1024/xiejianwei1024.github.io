---
layout: post
title: java.lang.String 源代码阅读
---

java.lang.String 表示字符串。

## 1. String 类的 equals() 方法

```java
public boolean equals(Object anObject) {
	//传进来的对象就是自身
    if (this == anObject) {
        return true;
    }
    //传进来的对象是否String类型
    if (anObject instanceof String) {
        String anotherString = (String) anObject;
        int n = value.length;
        //两个字符串的长度是否一样
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
            	//相同位置的两个字符是否一样
                if (v1[i] != v2[i])
                        return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

### 例子：

```java
@Test
public void equalsTest() {
	String str = "abc";
	String str2 = "abc";
	System.out.println(str.equals(str2));
}
```

运行结果：true

解析：判断当前字符串与传递进来的字符串内容是否一致。比较规则：两字符串的长度一样并且相同位置的字符一样。