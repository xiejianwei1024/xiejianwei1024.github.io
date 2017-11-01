---
layout: post
title: java位移运算
---

## java中有三种移位运算符

\<\<      :     左移运算符，运算规则：左移n位相当于乘以2的n次方。num << 1,相当于num乘以2。  

\>\>      :     右移运算符，运算规则：右移n位相当于除以2的n次方。这里是取商哈，余数就不要了。              num >> 1,相当于num除以2。  

\>\>\>    :     无符号右移，运算规则：按二进制形式把所有的数字向右移动对应位数，低位移出(舍弃)，高位的空位补零。对于正数来说和带符号右移相同，对于负数来说不同。 其他结构和>>相似。忽略符号位，空位都以0补齐。

### 举例

```java
@Test
public void testBitwise() {
	int number = 10;
	int b = number << 1;
	System.out.print(b+":");
	int c = number >> 1;
	System.out.print(c);
}
```

运行结果：  20:5  