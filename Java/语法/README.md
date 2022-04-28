### Java 无穷大、无穷小的写法

```java
// Integer
Integer.MAX_VALUE;
Integer.MIN_VALUE;
// Long
Long.MAX_VALUE;
Long.MIN_VALUE;
//Double
Double.MAX_VALUE;
Double.MIN_VALUE;
// Float
Float.MAX_VALUE;
Float.MIN_VALUE;
// 无穷大
Double.POSITIVE_INFINITY;
Double.NEGATIVE_INFINITY;
```

### Java length()方法，length属性和size()方法的区别

1. `length()`方法是针对字符串来说的，要求一个字符串的长度就要用到它的length()方法；
2. `length` 属性是针对Java中的数组来说的，要求数组的长度可以用其length属性；
3. Java中的`size()`方法是针对泛型集合说的，如果想看这个泛型有多少个元素，就调用此方法来查看

### Java HashMap 的用法

创建HashMap 对象 Sites

```java
HashMap<Integer, String> Sites = new HashMap<Integer, String>();
```

添加键值对

```java
Sites.put(1, "Google");
Sites.put(2, "Runoob");
Sites.put(3, "Taobao");
Sites.put(4, "Zhihu");
```

访问元素

```java
(Sites.get(3));
```

删除元素

```java
Sites.remove(4);
```

删除所有键值对(key-value)可以使用clear方法

```java
Sites.clear();
```

计算大小

```java
System.out.println(Sites.size());
```

迭代HashMap

```java
// 输出 key 和 value
for (Integer i : Sites.keySet()) {
    System.out.println("key: " + i + " value: " + Sites.get(i));
}
// 返回所有 value 值
for(String value: Sites.values()) {
    // 输出每一个value
    System.out.print(value + ", ");
}
```

检查 hashMap 中是否存在指定的 key 对应的映射关系

```java
System.out.println(Sites.containKey(4));
```

获取指定key对应的value，如果找不到key，则返回设置的默认值

```java
String value1 = sites.getOrDefault(1, "Not Found");
```

### Java HashSet 的用法

创建HashSet 对象 sites

```java
HashSet<String> sites = new HashSet<String>();
```

添加元素

```java
sites.add("Google");
sites.add("Runoob");
sites.add("Taobao");
sites.add("Zhihu");
sites.add("Runoob");  // 重复的元素不会被添加
```

判断元素是否存在

```java
System.out.println(sites.contains("Taobao"));
```

删除元素

```java
sites.remove("Taobao");  // 删除元素，删除成功返回 true，否则为 false
```

删除集合中的所有元素可以使用clear方法

```java
sites.clear(); 
```

计算大小

```java
System.out.println(sites.size());  
```

迭代HaseSet

```java
System.out.println(i);
```

### Java ArrayList的对象

创建ArrayList 对象 sites

```java
ArrayList<String> sites = new ArrayList<String>();
```

添加元素

```java
sites.add("Google");
sites.add("Runoob");
sites.add("Taobao");
sites.add("Weibo");
```

访问元素

```java
System.out.println(sites.get(1));  // 访问第二个元素
```

修改元素

```java
sites.set(2, "Wiki"); // 第一个参数为索引位置，第二个为要修改的值
```

删除元素

```java
sites.remove(3); // 删除第四个元素
```

计算大小

```java
System.out.println(sites.size());
```

迭代数组列表

```java
for (int i = 0; i < sites.size(); i++) {
    System.out.println(sites.get(i));
}
for (String i : sites) {
    System.out.println(i);
}
```

### Java LinkedList

链表是一种常见的基础数据结构，是一种线性表，但是并不会按线性顺序存储数据，而是在每一个节点里存到下一个节点的地址。

链表可分为单向链表和双向链表：

- 一个单向链表包含两个值：当前节点的值和一个指向下一个节点的链接；
- 一个双向链表有三个整数值：数值，向后的节点链接、向前的节点链接；

以下情况使用ArrayList：

- 频繁访问列表中的某一个元素；
- 只需要在列表末尾进行添加和删除元素操作

以下情况使用LinkedList：

- 需要通过循环迭代来访问列表中的某些元素
- 需要频繁的在列表开头、中间、末尾等位置进行添加和删除元素操作

### Java Stack 对象

创建 Stack 类对象

```java
Stack<Integer> st = new Stack<Integer>();
```

测试栈堆是否为空

```java
boolean empty() 
```

查看堆栈顶部的对象，但不从堆栈中移除它

```java
Object peek( )
```

移除堆栈顶部的对象，并将此函数的值返回该对象

```java
Object pop( )
```

把项压入堆栈顶部

```java
Object push(Object element)
```

返回对象在堆栈的位置，以1为基数。

```java
int search(Object element)
```

### Java中Arrays.sort()的几种用法

##### 1. Arrays.sort(int[] a)

这种形式是对一个数组的所有元素进行排序，并且是按从小到大的顺序。

```java
Arrays.sort(a);
```

##### 2. Arrays.sort(int[] a, int fromIndex, int toIndex)

这种形式对数组部分排序，也就是对数组a的下标从fromIndex到toIndex-1的元素排序，注意，下标为toIndex的元素不参与排序！

```java
Arrays.sort(a, 0, 3);
```

##### 3. `public static <T> void sort(T[] a, int fromIndex, int toIndex, Comparator<? super T> c) `

```java
// 将列表中的区间按照左端点升序排序
Arrays.sort(intervals, new Comparator<int[]>(){
    public int compare(int[] interval1, int[] interval2){
        return interval1[0] - interval2[0];
    }
});
```

#### Java-String类常用方法总结

#### 1. String 类

#### 2. String类对象的构建

字符串声明

```java
String stringName;
```

字符串创建

```java
String stringName = new String(字符串常量);
String stringName = 字符串常量;
```

#### 3. String类构造方法

##### 3.1 public String()

无参构造方法，用来创建空字符串的String对象

```java
String str1 = new String();
```

##### 3.2 public String(String value)

用已知的字符串value创建一个String对象

```java
String str2 = new String("asdf");
String str3 = new String(str2);
```

##### 3.3 public String(char[] value)

用字符数组value创建一个String对象。

```java
char[] value = {'a','b','c','d'};
String str4 = new String(value);//相当于String str4 = new String("abcd");
```

##### 3.4 public String(char chars[], int startIndex, int numChars)

用字符数组chars的startIndex开始的numChars个字符创建一个String对象。

```java
char[] value = {'a','b','c','d'};
String str5 = new String(value, 1, 2);//相当于String str5 = new String("bc");
```

##### 3.5 public String(byte[] values)

用比特数组values创建一个String对象

```java
byte[] strb = new byte[]{65,66};
String str6 = new String(strb);//相当于String str6 = new String("AB");
```

#### 4. String类常用方法

##### 4.1 求字符串长度

public int length()	//返回该字符串的长度

```java
String str = new String("asdfzxc");
int strlength = str.length();//strlength = 7
```

##### 4.2 求字符串某一位置字符

public char charAt(int index)	//返回字符串中指定位置的字符； 注意字符串中的第一个字符索引是0，最后一个是length()-1。

```java
String str = new String("asdfzxc");
char ch = str.charAt(4);//ch = z
```

##### 4.3 提取子串

用String类的subString方法可以提取字符串中的子串，该方法有两种常用参数：

1）public String substring(int beginIndex)	//该方法从beginIndex位置起，从当前字符串中取出剩余的字符作为一个新的字符串返回

2）public String substring(int beginIndex, int endIndex)	//该方法从beginIndex位置开始，从当前字符串中取到endIndex-1位置的字符作为一个新的字符串返回

```java
String str1 = new String("asdfzxc");
String str2 = str1.substring(2);	//str2 = "dfzxc"
String str3 = str1.substring(2,5);	//str3 = "dfz"
```

##### 4.4 字符串比较

1）public int compareTo(String anotherString)	//该方法是对字符串内容按字典顺序进行大小比较，通过返回的整数值指明当前字符串与参数字符串的大小关系。若当前对象比参数大则返回正整数，反之返回负整数，相等返回0。

2）public int compareToIgnore(String anotherString)	//与compareTo方法相似，但忽略大小写。

3）public boolean equals(Object anotherObject)	//比较当前字符串和参数字符串，在两个字符串相等的时候返回true，否则返回false。

4）public boolean equalsIgnoreCase(String anotherString)	//与equals方法相似，但忽略大小写

```java
String str1 = new String("abc");
String str2 = new String("ABC");
int a = str1.compareTo(str2);//a>0
int b = str1.compareToIgnoreCase(str2);//b=0
boolean c = str1.equals(str2);//c=false
boolean d = str1.equalsIgnoreCase(str2);//d=true
```

##### 4.5 字符串连接

public String concat(String str)	//将参数中的字符串str连接到当前字符串的后面，效果等价于"+"。

```java
String str = "aa".concat("bb").concat("cc");	//String str = "aa"+"bb"+"cc";
```

##### 4.6 字符串中单个字符查找

1）public int indexOf(int ch/String str)	//用于朝朝当前字符串中字符或子串，返回字符或子串在当前字符串中从左边首次出现的位置，若没有出现则返回-1。

2）public int indexOf(int ch/String str, int fromIndex)	//该方法与第一种方法类似，区别在于该方法从fromIndex位置向后查找。

3）public int lastIndexOf(int ch/String str)	//该方法与第一种方法类似，区别在于该方法从字符串的末尾位置向前查找。

4）public int lastIndexOf(int ch/String str, int fromIndex)	//该方法与第三种方法类似，区别在于该方法从fromIndex位置向前查找。

```java
String str = "I am a good student";
int a = str.indexOf('a');		//a = 2
int b = str.indexOf("good");	//b = 7
int c = str.indexOf("w",2);		//c = -1
int d = str.lastIndexOf("a");	//d = 5
int e = str.lastIndexOf("a",3);	//e = 2
```

##### 4.7 字符串中字符的大小写转换

1）public String toLowerCase()	//返回将当前字符串中所有字符转换成小写后的新串

2）public String toUpperCase()	//返回将当前字符串中所有字符转换成大写后的新串

```java
String str = new String("asDF");
String str1 = str.toLowerCase();//str1 = "asdf"
String str2 = str.toUpperCase();//str2 = "ASDF"
```

##### 4.8 字符串中字符的替换

1）public String replace(char oldChar, char newChar)	//用字符newChar替换当前字符串中所有的oldChar字符，并返回一个新的字符串。

2）public String replaceFirst(String regex, String replacement)	//该方法用字符replacement的内容替换当前字符串中第一个和字符串regex相匹配的子串，应将新的字符串返回。

3）public String replaceAll(String regex, String replacement)	//该方法用字符replacement的内容替换当前字符串中遇到的所有和字符串regex的子串，应将新的字符串返回。

```java
String str = "asdzxcasd";
String str1 = str.replace('a','g');//str1 = "gsdzxcgsd"
String str2 = str.replace("asd","fgh");//str2 = "fghzxcfgh"
String str3 = str.replaceFirst("asd","fgh");//str3 = "fghzxcasd"
String str4 = str.replaceAll("asd","fgh");//str4 = "fghzxcfgh"
```

##### 4.9 其它类方法

1）String trim()	//截取字符串两端的空格，但对于中间的空格不处理。

```java
String str = " a sd ";
String str1 = str.trim();
int a = str.length();//a = 6
int b = str1.length();//b = 4
```

2）boolean statWith(String prefix)或boolean endWith(String suffix)	//用来比较当前字符串的起始字符或子字符串prefix和终止字符或子字符串suffix是否和当前字符串相同，重载方法中同时还可以比较指定比较的开始位置offset。

```java
String str = "asdfgh";
boolean a = str.statWith("as");//a = true
boolean b = str.endWith("gh");//b = true
```

3）regionMatches(boolean b, int firstStart, String other, int otherStart, int length)	//从当前字符串的firstStart位置开始比较，取长度为length的一个子字符串，other字符串从otherStart位置开始，指定另外一个长度为length的字符串，两字符串比较，当b为true时字符串不区分大小写。

4）contains(String str)	//判断参数s是否被包含在字符串中，并返回一个布尔类型的值。

```java
String str = "student";
str.contains("stu");//true
str.contains("ok");//false
```

5）String[] split(String str)	//将str作为分隔符进行字符串分解，分解后的字符串在字符串数组中返回。

```java
String str = "asd!qwe|zxc#";
String[] str1 = str.split("!|#");//str1[0] = "asd";str1[1] = "qwe";str1[2] = "zxc";
```

#### 5. 字符串与基本类型的转换

##### 5.1 字符串转换为基本类型

java.lang包中有Byte, Short, Integer, Float, Double类的调用方法：

1. public static byte parseByte(String s)；
2. public static short parseShort(String s);
3. public static short parseInt(String s);
4. public static long parseLong(String s);
5. public static float parseFloat(String s);
6. public static double parseDouble(String s);

例如：

```java
int n = Integer.parseInt("12");
float f = Float.parseFloat("12.34");
double d = Double.parseDouble("1.124");
```

##### 5.2 基本类型转换为字符串类型

String 类中提供了String valueOf()方法，用作基本类型转换为字符串类型

1. static String valueOf(char data[]);
2. static String valueOf(char data[], int offset, int count);
3. static String valueOf(boolean b);
4. static String valueOf(char c);
5. static String valueOf(int i);
6. static String valueOf(long l);
7. static String valueOf(float f);
8. static String valueOf(double d)

例如：

```java
String s1 = String.valueOf(12);
String s1 = String.valueOf(12.34);
```

##### 5.3 进制转换

使用Loning 类中的方法得到整数之间的各种进制转换方法

Long.toBinaryString(long l)

Long.toOctalString(long l)

Long.toHexString(long l)

Long.toString(long l, int p)	//p作为任意进制

