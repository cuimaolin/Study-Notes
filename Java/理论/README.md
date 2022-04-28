### Java static 关键字

在一个`class`中定义的字段，我们称之为实例字段。实例字段的特点是，每个实例都有独立的字段，各个实例的同名字段互不影响。

还有一种字段，是用`static`修饰的字段，称为静态字段:`static field`

实例字段在每个实例中都有自己的一个独立空间，但是静态字段只有一个共享空间，所有实例都会共享该字段。举个例子：

```java
class Person {
    public String name;
    public int age;
    // 定义静态字段number:
    public static int number;
}
```

我们来看看下面的代码

```java
public class Main {
    public static void main(String[] args) {
        Person ming = new Person("Xiao Ming", 12);
        Person hong = new Person("Xiao Hong", 15);
        ming.number = 88;
        System.out.println(hong.number);
        hong.number = 99;
        System.out.println(ming.number);
    }
}

class Person {
    public String name;
    public int age;

    public static int number;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

对于静态字段，无论修改哪个实例的静态字段，效果都是一样的：所有实例的静态字段都被修改了，原因是静态字段并不属于实例。

虽然实例可以访问静态字段，但是它们指向的其实都是`Person class`的静态字段。所以，所有实例共享一个静态字段。

因此，不推荐用`实例变量.静态字段`去访问静态字段，因为在Java程序中，实例对象并没有静态字段。在代码中，实例对象能访问静态字段只是因为编译器可以根据实例类型自动转换为`类名.静态字段`来访问静态对象。

推荐用类名来访问静态字段。可以把静态字段理解为描述`class`本身的字段(非实例字段)。对于上述代码，更好的写法是：

```java
Person.number = 99;
System.out.println(Person.number);
```
#### 1. 静态方法

有了静态字段，就有静态方法。用`static`修饰的方法称为静态方法。

调用实例方法必须通过一个实例变量，而调用静态方法则不需要实例变量，通过类名就可以调用。静态方法类似其他编程语言的函数。例如：

```java
public class Main {
    public static void main(String[] args) {
        Person.setNumber(99);
        System.out.println(Person.number);
    }
}

class Person {
    public static int number;

    public static void setNumber(int value) {
        number = value;
    }
}
```

因为静态方法属于'class'而不属于实例，因此，静态方法内部，无法访问`this`变量，也无法访问实例字段，它只能访问静态字段。

通过实例变量也可以调用静态方法，但这只是编译器自动帮我们把实例改写成类名而已。

通常情况下，通过实例变量访问静态字段和静态方法，会得到一个编译警告。

静态方法经常用于工具类。例如：

- Arrays.sort()
- Math.random()

静态方法也经常用于辅助方法。注意到Java程序的入口`main()`也是静态方法。

#### 2. 接口的静态字段

因为`interfcce`是一个纯抽象类，所以它不能定义实例字段。但是，`interface`是可以有静态字段的，但是静态字段必须为`final`类型：

```java
public interface Person {
    public static final int MALE = 1;
    public static final int FEMALE = 2;
}
```

实际上，因为`interface`的字段只能是`public static final`类型，所以我们可以把这些修饰符都去掉，上述代码可以简写为：

```java
public interface Person {
    // 编译器会自动加上public statc final:
    int MALE = 1;
    int FEMALE = 2;
}
```

编译器会自动把该字段变为`public static final`类型

### String、StringBuffer和StringBuilder的区别

#### 1. String长度不可变而StringBuffer和StringBuilder长度可变

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {}
	/** The value is used for character storage. */
	private final char value[];
```

```java
public final class StringBuffer extends AbstractStringBuilder implements java.io.Serializable, CharSequence
```

```java
public final class StringBuilder extends AbstractStringBuilder
```

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage
     */
    char[] value;
```

我们能看到，String这个类底层使用了final修饰的长度不可变的字符数组，所以它长度不可变

```java
private final char value[];
```

而StringBuffer和StringBuilder都继承自AbstractStringBuilder，且AbstractStringBuilder底层使用的是可变字符数组，所以二者长度可变。

```jav
char[] value;
```

#### 2. 他们的运行速度不同：StringBuilder > StringBuffer > String

```java
String  str = "abc";
str = str + "cd"
```

对于String对象，其字符串长度是不可变的，其实JVM是先创建的了一个str对象，将”abc“赋值给str，然后在内存中又创建了第二个str对象，将第一个str对象的”abc“与”de“相加再复制给第二个str对象，此时Java虚拟机的垃圾回收机制开始其工作将第一个str对象回收。所以说String类型的字符串要完成这样”改变长度“的操作需要不断地创建回收，创建再回收，无形中经过了很多步骤，而StringBuffer和StringBuilder数组可变，直接可进行更改，所以要快。

**而StringBuilder为什么比StringBuffer要快呢？**

```java
@Override
public StringBuilder append(Object obj) { return append(String.valueOf(obj)); }

@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
```

```java
@Override
public synchronized StringBuffer append(Object obj) {
    toStringCache = null;
    super.append(String.valueOf(obj));
    return this;
}

@Override
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```

从图中可以看出StringBuffer的append方法都被toStringCache关键字修饰了（不止途中这两个append方法包括StringBuffer源码中所有append重载方法都被toStringCache修饰了）。

toStringCache关键字是给线程加锁，加锁是会带来性能上的损耗的，故用StringBuilder比StringBuffer要快

#### 3. StringBuilder线程不安全 和 StringBuffer 线程安全

有了toStringCache关键字修饰StringBuffer的append方法，给线程加了锁所以线程安全。

这样理解，**如果一个StringBuffer对象的字符串在字符串缓冲区被多个线程同时使用，也就是多个线程同时操作，这样会有出现错误操作的频率，为了保证线程的安全性，进行加锁，这样会使同一时间只有一个线程获得权限，其他线程必须等待该线程结束并释放锁才能获得权限**，这样线程非常安全，虽然效率慢了点，但是当项目安全性要求很高时就必须使用StringBuffer。单一线程下还是使用更快的StringBuilder