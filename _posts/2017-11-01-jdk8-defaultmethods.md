---
date: 2017-05-29 09:14
layout: post
title: jdk8-Default Methods(Oracle 官网翻译)
---

摘自https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html

## 默认方法（Default Methods）

Default methods enable you to add new functionality to the interfaces of your libraries and ensure binary compatibility with code written for older versions of those interfaces.

默认方法允许您向现有库的接口中添加新的功能，并能确保与采用旧版本接口编写的代码的二进制兼容性。

看下面的接口：
```java
package defaultmethods;

import java.time.LocalDateTime;

public interface TimeClient {
	void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
                               int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
}
```

下面的类 SimpleTimeClient 实现了 TimeClient 接口：
```java
package defaultmethods;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;

public class SimpleTimeClient implements TimeClient {

	private LocalDateTime dateAndTime;

	public SimpleTimeClient() {
		dateAndTime = LocalDateTime.now();
	}

	public void setTime(int hour, int minute, int second) {
		LocalDate currentDate = LocalDate.from(dateAndTime);
		LocalTime timeToSet = LocalTime.of(hour, minute, second);
		dateAndTime = LocalDateTime.of(currentDate, timeToSet);
	}

	public void setDate(int day, int month, int year) {
		LocalDate dateToSet = LocalDate.of(day, month, year);
		LocalTime currentTime = LocalTime.from(dateAndTime);
		dateAndTime = LocalDateTime.of(dateToSet, currentTime);
	}

	public void setDateAndTime(int day, int month, int year, int hour, int minute, int second) {
		LocalDate dateToSet = LocalDate.of(day, month, year);
		LocalTime timeToSet = LocalTime.of(hour, minute, second);
		dateAndTime = LocalDateTime.of(dateToSet, timeToSet);
	}

	public LocalDateTime getLocalDateTime() {
		return dateAndTime;
	}

	public String toString() {
		return dateAndTime.toString();
	}

	public static void main(String... args) {
		TimeClient myTimeClient = new SimpleTimeClient();
		System.out.println(myTimeClient.toString());
	}

}

```

假设您要向 TimeClient 接口中添加新的功能，例如通过 ZonedDateTime 对象指定时区的功能（它类似于 LocalDateTime 对象，除了它存储时区信息）：
```java
package defaultmethods;

import java.time.LocalDateTime;
import java.time.ZonedDateTime;

public interface TimeClient {
	void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
                               int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
    //新功能
    ZonedDateTime getZonedDateTime(String zoneString);
}

```

随着对 TimeClient 接口的修改，您还必须修改 SimpleTimeClient 类并实现 getZonedDateTime 方法。
然而，您可以改为定义一个默认的实现，而不是将 getZonedDateTime 作为抽象方法（abstract）。（请记住，抽象方法是一个声明为没有实现的方法。）
```java
package defaultmethods;

import java.time.DateTimeException;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;

public interface TimeClient {
	void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
                               int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
    
    static ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }
        
    default ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }
}

```

在接口中,您可以在方法签名的开头使用 default 关键字指定一个默认方法。在接口中的所有方法声明，包括默认方法，都是 public，这样你就可以省略了 public 修饰符。

有了这个接口，您不必修改类 SimpleTimeClient ，并且这个类（以及实现接口 TimeClient 的任何类），已经定义了 getZonedDateTime 方法。下面的例子，TestSimpleTimeClient，从 SimpleTimeClient 的一个实例调用 getZonedDateTime 方法：
```java
package defaultmethods;

public class TestSimpleTimeClient {
	public static void main(String... args) {
		TimeClient myTimeClient = new SimpleTimeClient();
		System.out.println("Current time: " + myTimeClient.toString());
		System.out.println("Time in California: " + myTimeClient.getZonedDateTime("Blah blah").toString());
	}
}
```
## 接口默认方法的继承（Extending Interfaces That Contain Default Methods）
当您扩展了包含默认方法的接口时，您可以执行以下三种操作：  

1.不覆写默认方法，直接从父接口中获取方法的默认实现。（继承）  

2.覆写默认方法并将它重新声明为抽象方法，这样新接口的子类必须再次覆写并实现这个抽象方法。  

3.覆写默认方法，这跟类与类之间的覆写规则相类似。  


假设您将如下扩展接口 TimeClient：
```java
package defaultmethods;

public interface AbstractZoneTimeClient extends TimeClient { }

```
实现接口 AnotherTimeClient 的任何类将具有默认方法 TimeClient.getZonedDateTime 指定的实现。（AnotherTimeClient 接口已经继承了默认方法，它的实现类不必覆写默认方法。）

假设您将如下扩展接口 TimeClient：
```java
package defaultmethods;

import java.time.ZonedDateTime;

public interface AbstractZoneTimeClient extends TimeClient {
    public ZonedDateTime getZonedDateTime(String zoneString);
}

```
实现接口 AbstractZoneTimeClient 的任何类都必须实现 getZonedDateTime 方法；这种方法是一种像接口中所有其他非默认（和非静态）方法的抽象方法。

假设您将如下扩展接口 TimeClient：
```java
package defaultmethods;

import java.time.DateTimeException;
import java.time.ZoneId;
import java.time.ZonedDateTime;

public interface HandleInvalidTimeZoneClient extends TimeClient {
	default public ZonedDateTime getZonedDateTime(String zoneString) {
        try {
            return ZonedDateTime.of(getLocalDateTime(),ZoneId.of(zoneString)); 
        } catch (DateTimeException e) {
            System.err.println("Invalid zone ID: " + zoneString +
                "; using the default time zone instead.");
            return ZonedDateTime.of(getLocalDateTime(),ZoneId.systemDefault());
        }
    }
}

```

实现接口 HandleInvalidTimeZoneClient的 任何类将使用此接口指定的 getZonedDateTime 的实现，而不是由接口 TimeClient 指定的。

## 静态方法（Static Methods）
除了默认方法之外，还可以在接口中定义静态方法。（静态方法是与定义它的类相关联的方法，而不是与任何对象相关联的方法。类的每个实例都共享其静态方法。）这使您更容易在库中组织辅助方法;您可以在同一接口中保留特定于接口的静态方法，而不是在单独的类中。以下示例定义了一个静态方法，该方法检索与时区标识符相对应的 ZoneId 对象；如果没有与给定标识符相对应的 ZoneId 对象，则使用系统默认时区。（因此，您可以简化 getZonedDateTime 方法。）
```java
public interface TimeClient {
    // ...
    static public ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }

    default public ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }    
}
```

像类中的静态方法一样，在接口中定义静态方法需要在方法签名的开头使用 static 关键字。接口中的所有方法声明，包括静态方法，都是 public 的，所以你可以省略 public 修饰符。