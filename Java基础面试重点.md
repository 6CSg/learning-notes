# 一、基础大杂烩

## 1.JVM vs JDK vs JRE

JDK 是 Java Development Kit 缩写，它是功能齐全的 Java SDK。它拥有 JRE 所拥有的一切，还有编译器（javac）和工具（如 javadoc 和 jdb）。它能够创建和编译程序。

JRE 是 Java 运行时环境。它是运行已编译 Java 程序所需的所有内容的集合，包括 Java 虚拟机（JVM），Java 类库，java 命令和其他的一些基础构件。但是，它不能用于创建新程序。

**JVM 并不是只有一种！只要满足 JVM 规范，每个公司、组织或者个人都可以开发自己的专属 JVM。** 也就是说我们平时接触到的 HotSpot VM 仅仅是是 JVM 规范的一种实现而已。

## 2.标识符和关键字的区别是什么？

在我们编写程序的时候，需要大量地为**程序、类、变量、方法等取名字**，于是就有了 **标识符** 。简单来说， **标识符就是一个名字** 。

有一些标识符，Java 语言已经赋予了其特殊的含义，只能用于特定的地方，这些特殊的标识符就是 **关键字** 。简单来说，关键字是java语言与生俱来的 。

**Java中的关键字：**

|                      |          |            |          |              |            |           |        |
| -------------------- | -------- | ---------- | -------- | ------------ | ---------- | --------- | ------ |
| 访问控制             | private  | protected  | public   |              |            |           |        |
| 类，方法和变量修饰符 | abstract | class      | extends  | final        | implements | interface | native |
|                      | new      | static     | strictfp | synchronized | transient  | volatile  | enum   |
| 程序控制             | break    | continue   | return   | do           | while      | if        | else   |
|                      | for      | instanceof | switch   | case         | default    | assert    |        |
| 错误处理             | try      | catch      | throw    | throws       | finally    |           |        |
| 包相关               | import   | package    |          |              |            |           |        |
| 基本类型             | boolean  | byte       | char     | double       | float      | int       | long   |
|                      | short    |            |          |              |            |           |        |
| 变量引用             | super    | this       | void     |              |            |           |        |
| 保留字               | goto     | const      |          |              |            |           |        |

## 3.基本/包装数据类型

Java 中有 8 种基本数据类型，分别为：

- 6 种数字类型：
  - 4 种整数型：`byte`、`short`、`int`、`long`
  - 2 种浮点型：`float`、`double`
- 1 种字符类型：`char`
- 1 种布尔型：`boolean`。

这 8 种基本数据类型的默认值以及所占空间的大小如下：

| 基本类型  | 位数 | 字节 | 默认值  | 取值范围                                   |
| :-------- | :--- | :--- | :------ | ------------------------------------------ |
| `byte`    | 8    | 1    | 0       | -128 ~ 127                                 |
| `short`   | 16   | 2    | 0       | -32768 ~ 32767                             |
| `int`     | 32   | 4    | 0       | -2147483648 ~ 2147483647                   |
| `long`    | 64   | 8    | 0L      | -9223372036854775808 ~ 9223372036854775807 |
| `char`    | 16   | 2    | 'u0000' | 0 ~ 65535                                  |
| `float`   | 32   | 4    | 0f      | 1.4E-45 ~ 3.4028235E38                     |
| `double`  | 64   | 8    | 0d      | 4.9E-324 ~ 1.7976931348623157E308          |
| `boolean` | 1    |      | false   | true、false                                |

这八种基本类型都有对应的包装类分别为：`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Character`、`Boolean` 。

### 基本类型和包装类型的区别？

- 成员变量包装类型不赋值就是 `null` ，而基本类型有默认值且不是 `null`。
- 包装类型可用于泛型，而基本类型不可以。
- 基本数据类型的局部变量**存放在 Java 虚拟机栈中的局部变量表中**，基本数据类型的成员变量（未被 `static` 修饰 ）存放在 Java 虚拟机的堆中。包装类型属于对象类型，我们知道**几乎所有对象实例都存在于堆中**。
- 相比于对象类型， 基本数据类型占用的空间非常小。

**为什么说是几乎所有对象实例呢？** 这是因为 HotSpot 虚拟机引入了 JIT 优化之后，会对对象进行**逃逸分析**，如果发现某一个对象并没有逃逸到方法外部，那么就可能通过标量替换来实现栈上分配，而避免堆上分配内存

### 包装类型的缓存机制

Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能。

`Byte`,`Short`,`Integer`,`Long` 这 4 种包装类默认创建了数值 **[-128，127]** 的相应类型的缓存数据，`Character` 创建了数值在 **[0,127]** 范围的缓存数据，`Boolean` 直接返回 `True` or `False`。

**Integer 缓存源码：**

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static {
        // high value may be configured by property
        int h = 127;
    }
}
```

### 自动装箱与拆箱，原理是什么？

**自动装箱：**将基本数据类型打包为包装类型

**自动拆箱：**将包装类变为基本数据类型

**自动装箱和自动拆箱是Java中的一个语法糖。**

- `Integer i = 10` 等价于 `Integer i = Integer.valueOf(10)`
- `int n = i` 等价于 `int n = i.intValue()`;

注意：**如果频繁拆装箱的话，也会严重影响系统的性能。我们应该尽量避免不必要的拆装箱操作。**

### 为什么浮点数运算的时候会有精度丢失的风险？

浮点数运算精度丢失代码演示：

```java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.println(a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
```

这与浮点数在计算机的底层存储有关，浮点数在计算机底层存储为 **符号位 + 阶码 + 尾数**

![image-20221112163421410](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221112163421410.png)

**只有当尾数（小数点后的数字）时1/2的n次方时，它才可以用二进制精确表示**，比如`0.2、0.3`等小数的尾数在转为二进制后是无限循环的，所以会超过位数的位宽，当超过位宽时，位数就会被截断，所以造成精度丢失。

### 如何解决浮点数运算的精度丢失问题？

`BigDecimal` 可以实现对浮点数的运算，不会造成精度丢失。通常情况下，大部分需要浮点数精确运算结果的业务场景（比如涉及到钱的场景）都是通过 `BigDecimal` 来做的。

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);

System.out.println(x); /* 0.1 */
System.out.println(y); /* 0.1 */
System.out.println(Objects.equals(x, y)); /* true */
```

### 超过 long 整型的数据应该如何表示？

基本数值类型都有一个表达范围，如果超过这个范围就会有数值溢出的风险。

在 Java 中，64 位 long 整型是最大的整数类型。

```java
long l = Long.MAX_VALUE;
System.out.println(l + 1); // -9223372036854775808
System.out.println(l + 1 == Long.MIN_VALUE); // true
```

`BigInteger` 内部使用 `int[]` 数组来存储任意大小的整形数据。

相对于常规整数类型的运算来说，`BigInteger` 运算的效率会相对较低。

## 4.String

### I.String类型的对象为什么不可变？

- String类是被final修饰的，所以String类不可以被继承
- String类底层使用char[]数组存储数据的，并且char[]数组变量被final和private修饰，所以该引用的指向不能被修改并且该引用没有提供给外界访问的入口

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

```

### II.String类型的对象不可变的好处

**1. 可以缓存 hash 值**

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

**2. String Pool 的需要**

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

![img](https://pdai.tech/_images/pics/f76067a5-7d5f-4135-9549-8199c77d8f1c.jpg)

**3. 安全性**

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

**4. 线程安全**

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。因为该对象被创建出来就不可能被修改所以线程安全

### III.String、 StringBuffer 、StringBuilder

在 JDK 6 及以前版本，字符串常量池保存字符串对象，因为JDK6中的方法区由Perm实现，而Perm在堆中，所以字符串对象内存直接在堆中分配

JDK 6 之后的版本中，既保存了字符串对象，又保存了字符串对象的引用。

**1. 可变性**

- String 不可变
- StringBuffer 和 StringBuilder 可变

**2. 线程安全**

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步

**对于三者使用的总结：**

1. 操作少量的数据: 适用 `String`
2. 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`
3. 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`

### IV.字符串拼接用“+” 还是 StringBuilder?

Java 语言本身并不支持运算符重载，“+”和“+=”是专门为 String 类重载过的运算符，也是 Java 中仅有的两个重载过的运算符。

```java
String str1 = "he";
String str2 = "llo";
String str3 = "world";
String str4 = str1 + str2 + str3;
```

上面的代码对应的字节码如下：

<img src="https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/image-20220422161637929.png" alt="img" style="zoom:67%;" />

可以看出，字符串对象通过“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。

不过，在循环内使用“+”进行字符串的拼接的话，存在比较明显的缺陷：**编译器不会创建单个 `StringBuilder` 以复用，会导致创建过多的 `StringBuilder` 对象**。

```java
String[] arr = {"he", "llo", "world"};
String s = "";
for (int i = 0; i < arr.length; i++) {
    s += arr[i];
}
System.out.println(s);
```

`StringBuilder` 对象是在循环内部被创建的，这意味着每循环一次就会创建一个 `StringBuilder` 对象。

<img src="https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/image-20220422161320823.png" alt="img" style="zoom:67%;" />

如果直接使用 `StringBuilder` 对象进行字符串拼接的话，就不会存在这个问题了。

```java
String[] arr = {"he", "llo", "world"};
StringBuilder s = new StringBuilder();
for (String value : arr) {
    s.append(value);
}
```

<img src="https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/image-20220422162327415.png" alt="img" style="zoom:67%;" />

## 5.字符串常量池详解

`String`类型的常量池比较特殊。它的主要使用方法有两种：

- 直接使用双引号声明出来的`String`对象会直接存储在常量池中。
- 如果不是用双引号声明的`String`对象，可以使用`String`提供的`intern`方法。intern 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中

```cpp
oop StringTable::lookup(int index, jchar* name,  
                        int len, unsigned int hash) {  
  for (HashtableEntry<oop>* l = bucket(index); l != NULL; l = l->next()) {  
    if (l->hash() == hash) {  
      if (java_lang_String::equals(l->literal(), name, len)) {  
        return l->literal();  
      }  
    }  
  }  
  return NULL;  
}  
```

**String的String Pool**是一个固定大小的`Hashtable`，默认值大小长度是1009，如果放进String Pool的String非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用`String.intern`时性能会大幅下降（因为要一个一个找）。

- 在 jdk6中`StringTable`是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。
- 在jdk7中，`StringTable`的长度可以通过一个参数指定：
  - `-XX:StringTableSize=99991`

### 字符串对象创建的三种方式：

- 使用双引号声明的字符串对象会保存在字符串常量池中。
- 使用 new 关键字创建的字符串对象会先从字符串常量池中找，如果没找到就创建一个，然后再在堆中创建字符串对象；如果找到了，就直接在堆中创建字符串对象。
- 针对没有使用双引号声明的字符串对象来说，就像下面代码中的 s1 那样，如果想把 s1 的内容也放入字符串常量池的话，可以调用 `intern()` 方法来完成。
  - 

```java
String s1 = new String("二哥") + new String("三妹");
```

### String.intern():

Java 7 的时候，字符串常量池从永久代中移动到了堆中，虽然此时永久代还没有完全被移除。Java 8 的时候，永久代被彻底移除。

这个变化也直接影响了 `String.intern()` 方法在执行时的策略，**Java 7 之前**，执行 `String.intern()` 方法的时候，不管对象在堆中是否已经创建，字符串常量池中仍然会创建一个内容完全相同的新对象； Java 7 之后呢，由于字符串常量池放在了堆中，执行 `String.intern()` 方法的时候，如果对象在堆中已经创建了，字符串常量池中就不需要再创建新的对象了，而是**直接保存堆中对象的引用**，也就节省了一部分的内存空间。

**由此可得出结论：**

- 在 JDK 6 及以前版本，字符串常量池保存字符串对象。
- JDK 6 之后的版本中，既保存了字符串对象（`new String()创建的`），又保存了字符串对象的引用(`String.intern()扔进去的`)。

**案例：**

```java
 String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4);
	//JDK1.6 false
	//JDK1.7 true
```

**分析：**

在JDK1.6中，首先s3指向堆中”11“对象，执行s3.intern()会在常量池中再创建一个"11"对象，s4指向的是常量池中的对象，所以结果为false。整个过程创建了两个对象

再JDK1.7中，首先s3指向堆中”11“对象，执行s3.intern()，由于常量池在堆中，所以不再重复创建对象，直接将”11”对象的引用传入常量池，即常量池中存放的引用与s3引用指向的对象相当，最后s4指向的对象与s3指向的对象相同，返回true。整个过程只创建了一个对象。

### String 类型的变量和常量做“+”运算时发生了什么？

先来看字符串不加 `final` 关键字拼接的情况**（JDK1.8）**：

```java
String str1 = "str";
String str2 = "ing";
String str3 = "str" + "ing";
String str4 = str1 + str2;
String str5 = "string";
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/java-guide-blog/image-20210817123252441.png)

**对于编译期可以确定值的字符串，也就是常量字符串 ，jvm 会将其存入字符串常量池。并且，字符串常量拼接得到的字符串常量在编译阶段就已经被存放字符串常量池，这个得益于编译器的优化。**

在编译过程中，Javac 编译器（下文中统称为编译器）会进行一个叫做 **常量折叠(Constant Folding)** 的代码优化。

常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

- 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量。
- `final` 修饰的基本数据类型和字符串变量
- 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

**引用的值在程序编译期是无法确定的，编译器无法对其进行优化。**

对象引用和“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。



```java
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

1

我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 `StringBuilder` 或者 `StringBuffer`。

不过，字符串使用 `final` 关键字声明之后，可以让编译器当做常量来处理。

常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

- 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量。
- `final` 修饰的基本数据类型和字符串变量
- 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

**引用的值在程序编译期是无法确定的，编译器无法对其进行优化。**

对象引用和“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。



```java
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

1

我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 `StringBuilder` 或者 `StringBuffer`。

不过，字符串使用 `final` 关键字声明之后，可以让编译器当做常量来处理。

常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

- 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量。
- `final` 修饰的基本数据类型和字符串变量
- 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

**引用的值在程序编译期是无法确定的，编译器无法对其进行优化。**

对象引用和“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。



```java
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

1

我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 `StringBuilder` 或者 `StringBuffer`。

不过，字符串使用 `final` 关键字声明之后，可以让编译器当做常量来处理。

常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

- 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量。
- `final` 修饰的基本数据类型和字符串变量
- 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

**引用的值在程序编译期是无法确定的，编译器无法对其进行优化。**

对象引用和“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。



```java
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

1

我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 `StringBuilder` 或者 `StringBuffer`。

不过，字符串使用 `final` 关键字声明之后，可以让编译器当做常量来处理。

常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

- 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量。
- `final` 修饰的基本数据类型和字符串变量
- 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

**引用的值在程序编译期是无法确定的，编译器无法对其进行优化。**

对象引用和“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。

```java
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 `StringBuilder` 或者 `StringBuffer`。

不过，字符**串使用 `final` 关键字声明之后，可以让编译器当做常量来处理**：

```java
final String str1 = "str";
final String str2 = "ing";
// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 常量池中的对象
System.out.println(c == d);// true
```

如果 ，**编译器在运行时才能知道其确切值的话，就无法对其优化**。

示例代码（`str2` 在运行时才能确定其值）：

```java
final String str1 = "str";
final String str2 = getStr();
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 在堆上创建的新的对象
System.out.println(c == d);// false
public static String getStr() {
      return "ing";
}
```

## 6.Object

### I.Object 类的常见方法有哪些？

Object 类是一个特殊的类，是所有类的父类。它主要提供了以下 11 个方法：

```java
/**
 * native 方法，用于返回当前运行时对象的 Class 对象，使用了 final 关键字修饰，故不允许子类重写。
 */
public final native Class<?> getClass()
/**
 * native 方法，用于返回对象的哈希码，主要使用在哈希表中，比如 JDK 中的HashMap。
 */
public native int hashCode()
/**
 * 用于比较 2 个对象的内存地址是否相等，String 类对该方法进行了重写以用于比较字符串的值是否相等。
 */
public boolean equals(Object obj)
/**
 * naitive 方法，用于创建并返回当前对象的一份拷贝。
 */
protected native Object clone() throws CloneNotSupportedException
/**
 * 返回类的名字实例的哈希码的 16 进制的字符串。建议 Object 所有的子类都重写这个方法。
 */
public String toString()
/**
 * native 方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
 */
public final native void notify()
/**
 * native 方法，并且不能重写。跟 notify 一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
 */
public final native void notifyAll()
/**
 * native方法，并且不能重写。暂停线程的执行。注意：sleep 方法没有释放锁，而 wait 方法释放了锁 ，timeout 是等待时间。
 */
public final native void wait(long timeout) throws InterruptedException
/**
 * 多了 nanos 参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上 nanos 毫秒。。
 */
public final void wait(long timeout, int nanos) throws InterruptedException
/**
 * 跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念
 */
public final void wait() throws InterruptedException
/**
 * 实例被垃圾回收器回收的时候触发的操作
 */
protected void finalize() throws Throwable { }
```

### II.== 和 equals() 的区别

如果是基本数据类型，`==`比较的是数据的值

如果是引用数据类型，`==`比较的是引用指向对象的内存地址

**`equals()`** 不能用于判断基本数据类型的变量，只能用来判断两个对象是否等价。`equals()`方法存在于`Object`类中，而`Object`类是所有类的直接或间接父类，因此所有的类都有`equals()`方法。

`Object` 类 `equals()` 方法：

```java
public boolean equals(Object obj) {
     return (this == obj);
}
```

实际上Object类中的equals()方法比较的还是对象的内存地址，所以我们一般要重写equals()方法，使其按照程序员的比较规则来判定对象是否等价

### III.hashCode()方法 

- `hashCode()` 的作用是获取哈希码（`int` 整数），也称为散列码。这个哈希码的作用是确定该对象在哈希表中的索引位置。
- `hashCode()`定义在 JDK 的 `Object` 类中，这就意味着 Java 中的任何类都包含有 `hashCode()` 函数。另外需要注意的是： `Object` 的 `hashCode()` 方法是本地方法，也就是用 C 语言或 C++ 实现的，该方法通常用来将对象的内存地址转换为整数之后返回。

```java
public native int hashCode();
```

散列表存储的是键值对(key-value)，它的特点是：**能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）**

### IV.为什么要有hashcode()方法

我们以“`HashSet` 如何检查重复”为例子来说明为什么要有 `hashCode`？

下面这段内容摘自我的 Java 启蒙书《Head First Java》:

> 当你把对象加入 `HashSet` 时，`HashSet` 会先计算对象的 `hashCode` 值来判断对象加入的位置，同时也会与其他已经加入的对象的 `hashCode` 值作比较，如果没有相符的 `hashCode`，`HashSet` 会假设对象没有重复出现。但是如果发现有相同 `hashCode` 值的对象，这时会调用 `equals()` 方法来检查 `hashCode` 相等的对象是否真的相同。如果两者相同，`HashSet` 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样我们就大大减少了 `equals` 的次数，相应就大大提高了执行速度。

其实， `hashCode()` 和 `equals()`都是用于比较两个对象是否相等。

### V.为什么重写 equals() 时必须重写 hashCode() 方法？

因为两个相等的对象的 `hashCode` 值必须是相等。也就是说**如果 `equals` 方法判断两个对象是相等的，那这两个对象的 `hashCode` 值也要相等**。当两个hashcode相等的对象却不一定等价。

如果重写 `equals()` 时没有重写 `hashCode()` 方法的话就可能会导致 `equals` 方法判断是相等的两个对象，`hashCode` 值却不相等。所以重写equals()方法同时重写hashcode()，目的就是**要保证两个等价的对象hashcode值必定相等**。

**思考** ：重写 `equals()` 时没有重写 `hashCode()` 方法的话，使用 `HashMap` 可能会出现什么问题。

如果没有重写hashcode()，这两个对象逻辑上本是相等，但计算出来的hashcode却不同，导致两个程序员主观认定的两个等价对象进入了两个不同的bucket，这逻辑上是有错误的。

**例子：**

下面的代码中，新建了两个等价的对象，并将它们添加到 HashSet 中。我们希望将这两个对象当成一样的，只在集合中添加一个对象，但是因为 EqualExample 没有实现 hasCode() 方法，因此这两个对象的散列值是不同的，最终导致集合添加了两个等价的对象。

```java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2
```

**总结** ：

- `equals` 方法判断两个对象是相等的，那这两个对象的 `hashCode` 值也要相等。
- 两个对象有相同的 `hashCode` 值，他们也不一定是相等的（哈希碰撞）。

### VI.深拷贝与浅拷贝

#### ~区别：

- 浅拷贝：以一个对象为模板，在堆上的再次创建一个对象，拷贝出来的对象与源对象有部分重合。也就是说，**如果源对象的属性是引用类型，浅拷贝会只会拷贝该引用指向的内存地址。**
- 深拷贝：以一个对象为模板，在堆上的再次创建一个对象，拷贝出来的对象是一个全新的对象。也就是说，**如果源对象的属性是引用类型，深拷贝会对该引用数据类型所指向的对象在进行一次拷贝，即又创建了次对象。**

**浅拷贝只会创建出一个对象，而深拷贝至少创建出一个对象。**

#### ~实现：

:o:**实现cloneable接口，重写clone方法：**

```java
public class Address implements Cloneable{
    private String name;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Address clone() {
        try {
            return (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Person implements Cloneable {
    private Address address;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Person clone() {
        try {
            Person person = (Person) super.clone();
            return person;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
//深拷贝，在实现clone方法的时候要通过setter方法对要拷贝的属性进行拷贝
@Override
public Person clone() {
    try {
        Person person = (Person) super.clone();
        person.setAddress(person.getAddress().clone());
        return person;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

:package:**直接在构造函数中拷贝**：

使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

```java
/**
 * @author VictorG
 * @date 2022/11/13 16:52
 */
public class Person implements Cloneable {
    private Address address;

    public Person(Address address) {
        this.address = address;
    }
	//拷贝构造函数，可通过拷贝构造函数对该对象进行浅拷贝或深拷贝
    public Person(Person personClone) {
        this.address = personClone.address;

    }

    public void setAddress(Address address) {
        this.address = address;
    }

    public static void main(String[] args) {
        Address address = new Address("芒市");
        Person person = new Person(address);
        Person personClone = new Person(person);
        System.out.println(person.address == personClone.address);

    }
}
class Address implements Cloneable{
    private String name;
    
    public Address(String name) {
        this.name = name;
    }
   
    @Override
    public Address clone() {
        try {
            return (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

## 7.关键字

### I.final

**1. 数据**

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就**不能引用其它对象，但是被引用的对象本身是可以修改的。**相当于C中的常量指针。

```java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

**2. 方法**

声明方法不能被子类重写。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

**3. 类**

声明类不允许被继承。

### II.static

**1. 静态变量**

- 静态变量: 又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它；静态变量在内存中只存在一份。
- 实例变量: 每创建一个实例就会产生一个实例变量，它与该实例同生共死。

**2. 静态方法**

静态方法在类加载的时候就存在了，**它不依赖于任何实例**。所以静态方法必须有实现，也就是说它**不能是抽象方法(abstract)，并且接口中的方法不能被声明为static**

**3. 静态语句块**

静态语句块在类初始化时运行一次。

**4. 静态内部类**

非静态内部类依赖于外部类的实例，而静态内部类不需要。

**3. 静态语句块**

静态语句块在类初始化时运行一次。

```java
public class A {
    static {
        System.out.println("123");
    }

    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new A();
    }
}
//123
```

### III.访问权限控制符

![img](https://img-blog.csdnimg.cn/20200404154707363.jpg)

类可以跨包继承。

**如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别**。这是为了确保可以使用父类实例的地方都可以使用子类实例，也就是确保满足里氏替换原则。

以下这个例子，**Person和Address不在同一个包下，如果Address中的x方法是default的，那么Person也访问不了，而且Person重写该方法时访问权限不能低于protected。**

![image-20221113171752420](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221113171752420.png)

![image-20221113171805571](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221113171805571.png)

### IV.隐式类型转换

因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型下转型为 short 类型。

```java
short s1 = 1;
// s1 = s1 + 1;
```

但是使用 += 运算符可以执行隐式类型转换。

```java
s1 += 1;
```

上面的语句相当于将 s1 + 1 的计算结果进行了向下转型:

```java
s1 = (short) (s1 + 1);
```

### V.switch

从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象，其实也是一个语法糖，底层用equals比较。

```java
String s = "a";
switch (s) {
    case "a":
        System.out.println("aaa");
        break;
    case "b":
        System.out.println("bbb");
        break;
}
```

switch 不支持 long，是因为 switch 的设计初衷是对那些只有少数的几个值进行等值判断，如果值过于复杂，那么还是用 if 比较合适。

## 8.抽象类与接口

**1. 抽象类**

抽象类和抽象方法都使用 abstract 关键字进行声明。抽象类一般会包含抽象方法，抽象方法一定位于抽象类中。

抽象类和普通类最大的区别是，**抽象类不能被实例化，需要继承抽象类才能实例化其子类**。

**2. 接口**

接口是抽象类的延伸，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。

从 Java 8 开始，**接口也可以拥有默认的方法实现**，这是因为不支持默认方法的接口的维护成本太高了。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类。

**接口的成员(字段 + 方法)默认都是 public 的**，并且不允许定义为 private 或者 protected。

**接口的字段默认都是 static 和 final 的。**

```java
public interface InterfaceExample {
    void func1();

    default void func2(){
        System.out.println("func2");
    }

    int x = 123; //实际上是 --> public static final int x = 123;
    // int y;               // Variable 'y' might not have been initialized
    public int z = 0;       // Modifier 'public' is redundant for interface fields
    // private int k = 0;   // Modifier 'private' not allowed here
    // protected int l = 0; // Modifier 'protected' not allowed here
    // private void fun3(); // Modifier 'private' not allowed here
}
```

**3. 比较**

- 从设计层面上看，抽象类提供了一种 IS-A 关系，那么就必须满足里式替换原则，即子类对象必须能够替换掉所有父类对象。而接口更像是一种 LIKE-A 关系，它只是提供一种方法实现契约，并不要求接口和实现接口的类具有 IS-A 关系。
- 从使用上来看，**一个类可以实现多个接口，但是不能继承多个抽象类。**
- 接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。
- 接口的成员只能是 public 的，而抽象类的成员可以有多种访问权限。

# 二、Java中的异常

## 1.异常的层次结构

在 Java 中，所有的异常都有一个共同的祖先 `java.lang` 包中的 `Throwable` 类。`Throwable` 类有两个重要的子类:

- **Exception：**程序本身可以处理的异常，可以通过 `catch` 来进行捕获。异常又分为运行时异常和非运行时异常。
  - 运行时异常：都是RuntimeException类及其子类异常，如NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常)等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理，这类异常如果不手动捕获也能过通过编译。
  - 非运行时异常（编译异常）：是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。

- **Error：**`Error` 属于程序无法处理的错误 ，我们没办法通过 `catch` 来进行捕获不建议通过`catch`捕获 。例如 Java 虚拟机运行错误（`Virtual MachineError`）、虚拟机内存不够错误(`OutOfMemoryError`)、类定义错误（`NoClassDefFoundError`）等 。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

<img src="https://pdai.tech/_images/java/java-basic-exception-1.png" alt="img" style="zoom:67%;" />

## 2.Throwable 类常用方法有哪些？

- `String getMessage()`: 返回异常发生时的简要描述
- `String toString()`: 返回异常发生时的详细信息
- `String getLocalizedMessage()`: 返回异常对象的本地化信息。使用 `Throwable` 的子类覆盖这个方法，可以生成本地化信息。如果子类没有覆盖该方法，则该方法返回的信息与 `getMessage()`返回的结果相同
- `void printStackTrace()`: 在控制台上打印 `Throwable` 对象封装的异常信息

## 3.异常关键字

**try** – 用于监听。将要被监听的代码(可能抛出异常的代码)放在try语句块之内，当try语句块内发生异常时，异常就被抛出。

**catch** – 用于捕获异常。catch用来捕获try语句块中发生的异常。

**finally** – finally语句块总是会被执行。它主要用于回收在try块里打开的物力资源(如数据库连接、网络连接和磁盘文件)。只有finally块，执行完成之后，才会回来执行try或者catch块中的return或者throw语句，如果finally中使用了return或者throw等终止方法的语句，则就不会跳回执行，直接停止。

**throw** – 用于手动抛出异常。

**throws** – 用在方法签名中，用于声明该方法可能抛出的异常，让调用者去处理。

### try-catch-finally 如何使用？

- `try`块 ： 用于捕获异常。其后可接零个或多个 `catch` 块，如果没有 `catch` 块，则必须跟一个 `finally` 块。
- `catch`块 ： 用于处理 try 捕获到的异常。
- `finally` 块 ： 无论是否捕获或处理异常，`finally` 块里的语句都会被执行。当在 `try` 块或 `catch` 块中遇到 `return` 语句时，`finally` 语句块将在方法返回之前被执行。

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
} finally {
    System.out.println("Finally");
}
```

输出：

```text
Try to do something
Catch Exception -> RuntimeException
Finally
```

**注意：不要在 finally 语句块中使用 return!** 当 try 语句和 finally 语句中都有 return 语句时，try 语句块中的 return 语句会被忽略。这是因为 try 语句中的 return 返回值会先被暂存在一个本地变量中，当执行到 finally 语句中的 return 之后，这个本地变量的值就变为了 finally 语句中的 return 返回值。

### finally 中的代码一定会执行吗？

不一定的！在某些情况下，finally 中的代码不会被执行。

就比如说 finally 之前虚拟机被终止运行的话，finally 中的代码就不会被执行。

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
    // 终止当前正在运行的Java虚拟机
    System.exit(1);
} finally {
    System.out.println("Finally");
}
```

输出：

```text
Try to do something
Catch Exception -> RuntimeException
```

另外，在以下 2 种特殊情况下，`finally` 块的代码也不会被执行：

1. 程序所在的线程死亡。
2. 关闭 CPU。

### 如何使用 try-with-resources 代替try-catch-finally？

**try-with-resource是Java 7中引入的，很容易被忽略。**

Java 中类似于`InputStream`、`OutputStream` 、`Scanner` 、`PrintWriter`等的资源都需要我们调用`close()`方法来手动关闭，一般情况下我们都是通过`try-catch-finally`语句来实现这个需求，如下：

```java
//读取文本文件的内容
Scanner scanner = null;
try {
    scanner = new Scanner(new File("D://read.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

JAVA 7 提供了更优雅的方式来实现资源的自动释放，自动释放的资源需要是实现了 **AutoCloseable 接口的类**。

```java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

**try 代码块退出时，会自动调用 scanner.close 方法**

当然多个资源需要关闭的时候，使用 `try-with-resources` 实现起来也非常简单，如果你还是用`try-catch-finally`可能会带来很多问题。

通过使用分号分隔，可以在`try-with-resources`块中声明多个资源。

```java
try (BufferedInputStream bin = new BufferedInputStream(new FileInputStream(new File("test.txt")));
     BufferedOutputStream bout = new BufferedOutputStream(new FileOutputStream(new File("out.txt")))) {
    int b;
    while ((b = bin.read()) != -1) {
        bout.write(b);
    }
}
catch (IOException e) {
    e.printStackTrace();
}
```

### 异常关键字小结：

- try、catch和finally都不能单独使用，只能是try-catch、try-finally或者try-catch-finally。
- try语句块监控代码，出现异常就停止执行下面的代码，然后将异常移交给catch语句块来处理。
- finally语句块中的代码一定会被执行，常用于回收资源 。
- throws：声明一个异常，告知方法调用者。
- throw ：抛出一个异常，至于该异常被捕获还是继续抛出都与它无关。

**Java编程思想一书中，对异常的总结：**

- 在恰当的级别处理问题。（在知道该如何处理的情况下了捕获异常。）
- 解决问题并且重新调用产生异常的方法。
- 进行少许修补，然后绕过异常发生的地方继续执行。
- 用别的数据进行计算，以代替方法预计会返回的值。
- 把当前运行环境下能做的事尽量做完，然后把相同的异常重抛到更高层。
- 把当前运行环境下能做的事尽量做完，然后把不同的异常抛到更高层。
- 终止程序。
- 进行简化（如果你的异常模式使问题变得太复杂，那么用起来会非常痛苦）。
- 让类库和程序更安全

# 三、泛型详解

## 1.为什么引入泛型？

泛型的本质是为了**参数化类型**（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，**这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法**

引入泛型的意义在于：

- **适用于多种数据类型执行相同的代码**（代码复用），如果不使用泛型，可能在一个类中要重载多个参数类型不同但参数个数相同且功能相同的方法以适应不同的对象，代码复用性很差。
- 泛型中的类型在使用时指定，不需要强制类型转换（**类型安全**，编译器会**检查类型**）引入泛型，它将提供类型的约束，提供编译前的检查

## 2.泛型的基本使用

### I.泛型类

在类上声明泛型，Java中的集合类都为泛型类

```java
class Point<T>{         // 此处可以随便写标识符号，T是type的简称  
    private T var ;     // var的类型由T指定，即：由外部指定  
    public T getVar(){  // 返回值的类型由外部决定  
        return var ;  
    }  
    public void setVar(T var){  // 设置的类型也由外部决定  
        this.var = var ;  
    }  
}  

class Notepad<K,V>{       // 此处指定了两个泛型类型  
    private K key ;     // 此变量的类型由外部决定  
    private V value ;   // 此变量的类型由外部决定  
    public K getKey(){  
        return this.key ;  
    }  
    public V getValue(){  
        return this.value ;  
    }  
    public void setKey(K key){  
        this.key = key ;  
    }  
    public void setValue(V value){  
        this.value = value ;  
    }  
} 
```

### II.泛型接口

在接口中声明泛型，java中的函数式接口都为泛型接口。

```java
interface Info<T>{        // 在接口上定义泛型  
    public T getVar() ; // 定义抽象方法，抽象方法的返回值就是泛型类型  
}  
class InfoImpl<T> implements Info<T>{   // 定义泛型接口的子类  
    private T var ;             // 定义属性  
    public InfoImpl(T var){     // 通过构造方法设置属性内容  
        this.setVar(var) ;    
    }  
    public void setVar(T var){  
        this.var = var ;  
    }  
    public T getVar(){  
        return this.var ;  
    }  
} 
```

### III.泛型方法

泛型方法，是在调用方法的时候指明泛型的具体类型。

<img src="https://pdai.tech/_images/java/java-basic-generic-4.png" alt="img" style="zoom:67%;" />

- 调用泛型方法语法格式

<img src="https://pdai.tech/_images/java/java-basic-generic-5.png" alt="img" style="zoom: 80%;" />

说明一下，定义泛型方法时，**必须在返回值前边加一个`<T>`，来声明这是一个泛型方法，持有一个泛型`T`，然后才可以用泛型T作为方法的返回值。**

当然，泛型方法不是仅仅可以有一个参数`Class<T>`，可以根据需要添加其他参数。

**为什么要使用泛型方法呢**？因为泛型类要在实例化的时候就指明类型，如果想换一种类型，不得不重新new一次，可能不够灵活；而泛型方法可以在调用的时候指明类型，更加灵活。

```java
class X<T> {
    //<T, R>表示该方法中又两个泛型T，R
    //<T，R>后的R表示该方法的返回值为R，由调用者规定
    //泛型方法中的泛型T与类上的泛型T没有关系，不是同一个类型
    //即使类上没有声明泛型，在方法中可以单独声明泛型
    public <T, R> R m1(T p1, R r) {
        System.out.println("haha");
        return r;
    }
    //该方法不是泛型方法，该参数类型T与类上的泛型T是同一个类型
    public void m2(T x) {

    }
}
```

### IV.泛型的上下限

在使用泛型的时候，我们可以为**传入的泛型类型实参进行上下边界的限制**，如：类型实参只准传入某种类型的父类或某种类型的子类。

- **泛型的上限 <T extends Class\>**，利用 `<? extends Class>` 形式的通配符，可以实现泛型的向上转型。

```java
class Info<T extends Number>{    // 此处泛型只能是数字类型
    private T var ;        // 定义泛型变量
    public void setVar(T var){
        this.var = var ;
    }
    public T getVar(){
        return this.var ;
    }
    public String toString(){    // 直接打印
        return this.var.toString() ;
    }
}

```

- **泛型下限<T super Class\>**

```java
public static void fun(Info<? super String> temp){    // 只能接收String或Object类型的泛型，String类的父类只有Object类
        System.out.print(temp + ", ") ;
}
```

通配符使用英文的问号（?）来表示。在我们创建一个泛型对象时，可以使用关键字 `extends` 限定子类，也可以使用关键字 `super` 限定父类。

```java
<?> 无限制通配符
<? extends E> extends 关键字声明了类型的上界，表示参数化的类型可能是所指定的类型，或者是此类型的子类
<? super E> super 关键字声明了类型的下界，表示参数化的类型可能是指定的类型，或者是此类型的父类
```

 使用原则《Effictive Java》： 为了获得最大限度的灵活性，**要在表示 生产者或者消费者 的输入参数上使用通配符**，使用的规则就是：生产者有上限、消费者有下限

1. 如果参数化类型表示一个 T 的生产者，使用 < ? extends T>，子生产父;
2. 如果它表示一个 T 的消费者，就使用 < ? super T>，父被子消费；
3. 如果既是生产又是消费，那使用通配符就没什么意义了，因为你需要的是精确的参数类型。

:pear: 看一个实际例子，**加深印象**

```java
private  <E extends Comparable<? super E>> E max(List<? extends E> e1) {
    if (e1 == null){
        return null;
    }
    //迭代器返回的元素属于 E 的某个子类型
    Iterator<? extends E> iterator = e1.iterator();
    E result = iterator.next();
    while (iterator.hasNext()){
        E next = iterator.next();
        if (next.compareTo(result) > 0){
            result = next;
        }
    }
    return result;
}
```

上述代码中的类型参数 E 的范围是`<E extends Comparable<? super E>>`，我们可以分步查看：

- 要进行比较，所以 E 需要是可比较的类，因此需要 `extends Comparable<…>`**（E的上限为Comparable）**
- `Comparable< ? super E>` 要对 E 进行比较，即 E 的消费者，所以需要用 super
- 而参数 `List< ? extends E>` 表示要操作的数据是 E 的子类的列表，指定上限，这样容器才够大

## 3.如何理解Java中的泛型是伪泛型？泛型中类型擦除

Java泛型这个特性是从JDK 1.5才开始加入的，因此为了兼容之前的版本，Java泛型的实现采取了“**伪泛型**”的策略，即Java在语法上支持泛型，但是在编译阶段会进行所谓的“**类型擦除**”（Type Erasure），将所有的泛型表示（尖括号中的内容）都替换为具体的类型（其对应的原生态类型），就像完全没有泛型一样。

**泛型的类型擦除原则**是：

- 消除类型参数声明，即删除`<>`及其包围的部分。
- 根据类型参数的上下界推断并替换所有的类型参数为原生态类型：如果类型参数是无限制通配符或没有上下界限定则替换为Object，如果存在上下界限定则根据子类替换原则取类型参数的最左边限定类型（即父类）。
- 为了保证类型安全，必要时插入强制类型转换代码。
- 自动产生“桥接方法”以保证擦除类型后的代码仍然具有泛型的“多态性”。

- 擦除类定义中的类型参数 - 无限制类型擦除

当类定义中的类型参数没有任何限制时，在类型擦除中直接被替换为Object，即形如`<T>`和`<?>`的类型参数都被替换为Object。

<img src="https://pdai.tech/_images/java/java-basic-generic-1.png" alt="img" style="zoom:67%;" />

- 擦除类定义中的类型参数 - 有限制类型擦除

当类定义中的类型参数存在限制（上下界）时，在类型擦除中替换为类型参数的上界或者下界，比如形如`<T extends Number>`和`<? extends Number>`的类型参数被替换为`Number`，`<? super Number>`被替换为Object。

<img src="https://pdai.tech/_images/java/java-basic-generic-2.png" alt="img" style="zoom:67%;" />

- 擦除方法定义中的类型参数

擦除方法定义中的类型参数原则和擦除类定义中的类型参数是一样的，这里仅以擦除方法定义中的有限制类型参数为例。

<img src="https://pdai.tech/_images/java/java-basic-generic-3.png" alt="img" style="zoom:67%;" />

### 如何理解类型擦除后保留的原始类型?

- 原始类型Object

```java
class Pair<T> {  
    private T value;  
    public T getValue() {  
        return value;  
    }  
    public void setValue(T  value) {  
        this.value = value;  
    }  
}
```

因为在`Pair<T>`中，`T` 是一个无限定的类型变量，所以用Object替换，其结果就是一个普通的类，如同泛型加入Java语言之前的已经实现的样子。在程序中可以包含不同类型的Pair，如`Pair<String>`或`Pair<Integer>`，但是擦除类型后他们的就成为原始的Pair类型了，**原始类型都是Object。**

- Pair这样声明的话

```java
public class Pair<T extends Comparable> {}
```

**那么原始类型就是Comparable。**

### 如何理解泛型的编译期检查？

Java编译器是通过**先检查代码中泛型的类型，然后在进行类型擦除，再进行编译**。

```java
public class Test {  

    public static void main(String[] args) {  

        ArrayList<String> list1 = new ArrayList();  
        list1.add("1"); //编译通过  
        list1.add(1); //编译错误  
        String str1 = list1.get(0); //返回类型就是String  

        
        ArrayDeque queue = new ArrayDeque<>();
        queue.add(3);//编译通过
        queue.add("asd");//编译通过
   		Object o = queue.poll();//返回类型就是Object  

        new ArrayList<String>().add("11"); //编译通过  
        new ArrayList<String>().add(22); //编译错误  

        String str2 = new ArrayList<String>().get(0); //返回类型就是String  
    }  
} 
```

**类型检查就是针对引用的，谁是一个引用，用这个引用调用泛型方法，就会对这个引用调用的方法进行类型检测，而无关它真正引用的对象**。

## 4.如何理解基本类型不能作为泛型类型？

> 比如，我们没有`ArrayList<int>`，只有`ArrayList<Integer>`, 为何？

因为当类型擦除后，ArrayList的原始类型变为Object，但是Object类型不能存储int值，只能引用Integer的值。

另外需要注意，我们能够使用`list.add(1)`是因为Java基础类型的自动装箱拆箱操作。

## 5.如何理解泛型类型不能实例化？

> 不能实例化泛型类型, 这本质上是由于类型擦除决定的：

我们可以看到如下代码会在编译器中报错：

```java
T test = new T(); // ERROR
```

因为在 Java 编译期没法确定泛型参数化类型，也就找不到对应的类字节码文件，所以自然就不行了，此外由于`T` 被擦除为 `Object`，如果可以 `new T()` 则就变成了 `new Object()`，失去了本意。   如果我们确实需要实例化一个泛型，应该如何做呢？可以通过反射实现：

```java
static <T> T newTclass (Class < T > clazz) throws InstantiationException, IllegalAccessException {
    T obj = clazz.newInstance();
    return obj;
}
```

### 如何理解泛型类中的静态方法和静态变量？

> 泛型类中的静态方法和静态变量不可以使用泛型类所声明的泛型类型参数

举例说明：

```java
public class Test2<T> {    
    public static T one;   //编译错误    
    public static  T show(T one){ //编译错误    
        return null;    
    }    
}
```

因为泛型类中的泛型参数的实例化是在定义对象的时候指定的，而静态变量和静态方法不需要使用对象来调用。对象都没有创建，如何确定这个泛型参数是何种类型，所以当然是错误的。

:scream_cat:**但是要注意区分下面的一种情况：**

```java
public class Test2<T> {    

    public static <T >T show(T one){ //这是正确的    
        return null;    
    }    
}
```

**因为这是一个泛型方法，在泛型方法中使用的T是自己在方法中定义的 T，而不是泛型类中的T，这两个T没有关系。**

## 6.如何获取泛型的参数类型？

> 既然类型被擦除了，那么如何获取泛型的参数类型呢？可以通过反射（`java.lang.reflect.Type`）获取泛型

`java.lang.reflect.Type`是Java中所有类型的公共高级接口, 代表了Java中的所有类型. Type体系中类型的包括：数组类型(GenericArrayType)、参数化类型(ParameterizedType)、类型变量(TypeVariable)、通配符类型(WildcardType)、原始类型(Class)、基本类型(Class), 以上这些类型都实现Type接口。

```java
public class GenericType<T> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public static void main(String[] args) {
        GenericType<String> genericType = new GenericType<String>() {};
        Type superclass = genericType.getClass().getGenericSuperclass();
        //getActualTypeArguments 返回确切的泛型参数, 如Map<String, Integer>返回[String, Integer]
        Type type = ((ParameterizedType) superclass).getActualTypeArguments()[0]; 
        System.out.println(type);//class java.lang.String
    }
}
```

**其中 `ParameterizedType`:**

```java
public interface ParameterizedType extends Type {
    // 返回确切的泛型参数, 如Map<String, Integer>返回[String, Integer]
    Type[] getActualTypeArguments();
    
    //返回当前class或interface声明的类型, 如List<?>返回List
    Type getRawType();
    
    //返回所属类型. 如,当前类型为O<T>.I<S>, 则返回O<T>. 顶级类型将返回null 
    Type getOwnerType();
}
```

# 四、集合

## 1.集合概述

Java集合，也就是容器，由两大接口派生而来，一个是Collection接口，一个是Map接口，前者用于存放单一对象，后者用于存储键值对。

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/java-collection-hierarchy.png)

### I.说说 List, Set, Queue, Map 四者的区别？

- List接口的实现类有Vector，ArrayList，LinkedList等，它们都是顺序存储的，而且元素可以重复
- Set接口的实现类有HashSet，TreeSet等，它们是无序不可重复的
- Queue接口下有PriorityQueue和ArrayDeque两个实现类，支持队列先进先出队列操作
- Map接口使用键值对（key-value）存储，key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值。

## 2.ArrayList详解

### I.概述

ArrayList底层是基于Object[]数组实现的，默认初始化容量为10，如果在创建对象时不主动指定容量，返回的引用会指向一个空数组，直到向集合中添加元素，容量才真正初始化。

```java
 private static final Object[] EMPTY_ELEMENTDATA = {};
 /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;
    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;

```

### II.自动扩容

每当向数组中添加元素时，都要去检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。数组扩容通过一个公开的方法**`ensureCapacity(int minCapacity)`**来实现。在实际添加大量元素前，我也可以使用ensureCapacity来手动增加ArrayList实例的容量，以减少递增式再分配的数量。

数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的**增长大约是其原容量的1.5倍(`oldCapacity + (oldCapacity >> 1)`)**。这种操作的代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。当我们可预知要保存的元素的多少时，要在构造ArrayList实例时，就指定其容量，以避免数组扩容的发生。或者根据实际需求，通过调用**`ensureCapacity`**方法来手动增加ArrayList实例的容量。

```java
public void ensureCapacity(int minCapacity) {
        if (minCapacity > elementData.length
            && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
                 && minCapacity <= DEFAULT_CAPACITY)) {
            modCount++;
            grow(minCapacity);
        }
    } 
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);//位运算
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

```

## 3.LinkedList详解

### I.概述

*LinkedList*同时实现了*List*接口和*Deque*接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列(*Queue*)，同时又可以看作一个栈(*Stack*)。当你需要使用栈或者队列时，可以考虑使用*LinkedList*，一方面是因为Java官方已经声明不建议使用*Stack*类，更遗憾的是，Java里根本没有一个叫做*Queue*的类(它是个接口名字)。关于栈或队列，现在的首选是*ArrayDeque*，它有着比*LinkedList*(当作栈或队列使用时)有着更好的性能。

LinkedList时基于**双向链表**实现的，LinkedList类中维护着一个Node数组

```java
 transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;

  private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

### II.增

add()方法有两个版本，一个是`add(E e)`，该方法在LinkedList的**末尾插入**元素，因为有`last`指向链表末尾，在末尾插入元素的花费是**常数时间**。只需要简单修改几个相关引用即可；另一个是`add(int index, E element)`，该方法是在指定下标处插入元素，需要先通过**线性查找**找到具体位置，然后修改相关引用完成插入操作。

```java
public boolean add(E e) {
        linkLast(e);
        return true;
    }
    
    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

### III.查

- indexOf(Object)：查找某个元素所在的位置
- get(int)：查找某个位置上的元素

indexOf 的内部分为两种，一种是元素为 null 的时候，必须使用 == 来判断；一种是元素为非 null 的时候，要使用 equals 来判断。因为 equals 是不能用来判 null 的，会抛出 NPE 错误。

```java
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

get 方法的内核其实还是 node 方法，这个之前已经说明过了，这里略过。

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

**可见，查找方法都是从头节点开始，复杂度为O(N)。**

### IV.**删**

我这个删的招式还挺多的：

- `remove()`：删除第一个节点
- `remove(int)`：删除指定位置的节点
- `remove(Object)`：删除指定元素的节点
- `removeFirst()`：删除第一个节点
- `removeLast()`：删除最后一个节点

remove 内部调用的是 removeFirst，所以这两个招式的功效一样。

`remove(int)` 内部其实调用的是 unlink 方法。

```java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

unlink 方法其实很好理解，就是更新当前节点的 next 和 prev，然后把当前节点上的元素设为 null。

```java
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

remove(Object) 内部也调用了 unlink 方法，**只不过在此之前要先找到元素所在的节点**

```java
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

**综上，只有在删除第一个节点或者删除最后一个节点的时间复杂度才是O(1)，否则还是先要线性查找到元素，时间复杂度为O(N)**

:question: **思考：**不都说链表的增加和删除时间复杂度为O(1)吗？为什么在LinkedList除了删头和删尾都是O(N)？

因为我们指定删除的对象（Object）和LinkedList底层维护的Node并不是同一个对象，Node是对Object的包装，Node没有暴露给用户。

### **V.改**

可以调用 `set()` 方法来更新元素：

```java
list.set(0, "沉默王五");
```

来看一下 `set()` 方法：

```java
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```

首先对指定的下标进行检查，看是否越界；然后根据下标查找原有的节点：

```java
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

`size >> 1`：也就是右移一位，相当于除以 2。对于计算机来说，移位比除法运算效率更高，因为数据在计算机内部都是二进制存储的。

换句话说，node 方法会对下标进行一个初步判断，如果靠近前半截，就从下标 0 开始遍历；如果靠近后半截，就从末尾开始遍历。

找到指定下标的节点就简单了，直接把原有节点的元素替换成新的节点就 OK 了，prev 和 next 都不用改动。

## 4.Arraylist 与 LinkedList 区别?

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；
2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别）
3. **插入和删除是否受元素位置的影响：** 
   - ① **`ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1),但如果此时发生扩容，时间复杂度就会变为O(N)，并且如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。
   -  ② **`LinkedList` 采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，但是与ArrayList的尾插法相比略慢，因为涉及节点间引用的修改，如果是要在指定位置`i`插入和删除元素的话（`(add(int index, E element)`） 时间复杂度近似为`o(n))`因为需要先移动到指定位置再插入。**
   - 如果是从集合的头部新增元素，ArrayList 花费的时间应该比 LinkedList 多，因为需要对头部以后的元素进行复制。如果是从集合的中间位置新增元素，ArrayList 花费的时间搞可能比 LinkedList 少，因为 LinkedList 需要遍历，但ArrayList也会涉及数据的复制。
   - for 循环遍历的时候，ArrayList 花费的时间远小于 LinkedList，因为在for循环中调用LinkedList的get()，在其内部实现中还会遍历链表；迭代器遍历的时候，两者性能差不多。
4. **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
5. **内存空间占用：** `ArrayList` 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 `LinkedList` 的空间花费则体现在它的每一个元素都需要消耗比 `ArrayList` 更多的空间（因为要存放直接后继和直接前驱以及数据）。

## 5.Java中Iterator和Iterable有什么区别？

### I.遍历List的三种方式

在 Java 中，我们对 List 进行遍历的时候，主要有这么三种方式。

第一种：for 循环。

```java
for (int i = 0; i < list.size(); i++) {
    System.out.print(list.get(i) + "，");
}
```

第二种：迭代器。

```java
Iterator it = list.iterator();
while (it.hasNext()) {
    System.out.print(it.next() + "，");
}
```

第三种：for-each。

```java
for (String str : list) {
    System.out.print(str + "，");
}
```

第一种我们略过，第二种用的是 Iterator，第三种看起来是 for-each，其实背后也是 Iterator，看一下反编译后的代码就明白了。

```java
Iterator var3 = list.iterator();

while(var3.hasNext()) {
    String str = (String)var3.next();
    System.out.print(str + "，");
}
```

**for-each 只不过是个语法糖，让我们在遍历 List 的时候代码更简洁明了。**

Iterator 是个接口，JDK 1.2 的时候就有了

### II.Iterator

```java
public interface Iterator<E> {
    // 判断集合中是否存在下一个对象
    boolean hasNext();
    // 返回集合中的下一个对象，并将访问指针移动一位
    E next();
    // 删除集合中调用next()方法返回的对象
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
} 
private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;
        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

      //…………
        }
```

**Iterator是一个接口，集合内部有一个实现了Interator接口的内部类**

### III.Iterable

```java
public interface Iterable<T> {
    
    Iterator<T> iterator();

    
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}

```

Iterable是一个被Collection接口继承的接口，Interable接口中维护着一个Interator成员变量，JDK 1.8 时，Iterable 接口中新增了 forEach 方法，它对 Iterable 的每个元素执行给定操作，具体指定的操作需要自己写Consumer接口通过accept方法回调出来。

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
list.forEach(integer -> System.out.println(integer));
```

原则上，只要一个 List 实现了 Iterable 接口，那么它就可以使用 for-each 这种方式来遍历，那具体该怎么遍历，还是要看它自己是怎么实现 Iterator 接口的。也就是说，List 的关系图谱中并没有直接使用 Iterator，而是使用 Iterable 做了过渡。

<img src="http://cdn.tobebetterjavaer.com/tobebetterjavaer/images/collection/iterator-iterable-01.png" alt="img" style="zoom:67%;" />

回头再来看一下第二种遍历 List 的方式。

```java
Iterator it = list.iterator();
while (it.hasNext()) {
}
```

发现刚好呼应上了。拿 ArrayList 来说吧，**它重写了 Iterable 接口的 iterator 方法**：

```java
public Iterator<E> iterator() {
    return new Itr();
}
```

**返回的对象 Itr 是个内部类，实现了 Iterator 接口，并且按照自己的方式重写了 hasNext、next、remove 等方法**。

### IV.为什么不要在foreach里执行删除操作

因为每次在foreach中每次调用next，都会调用**`checkForComodification()`**，该方法会检测 modCount，`new Itr()` 的时候 expectedModCount 被赋值为 modCount，它是List 的一个成员变量，表示集合被修改的次数，当我们用list的remove删除元素时expectedModCount 和modCount就不相等了，这时就会抛异常。

这时一种fail-fast机制，fail-fast 是一种通用的系统设计思想，一旦检测到可能会发生错误，就立马抛出异常，程序将不再往下执行。

```java
public E next() {
    checkForComodification();
    int i = cursor;
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}
```

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

所以要在foreach中删除元素，要调用迭代器内部的remove方法，该方法会帮我们修改expectedModCount ，这就可以保证expectedModCount 和modCount始终相等了。

```java
public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();

    try {
        ArrayList.this.remove(lastRet);
        cursor = lastRet;
        lastRet = -1;
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```

## 6.comparable 和 Comparator 的区别

- 一个类实现了 Comparable 接口，意味着该类的对象可以直接进行比较（排序），但比较（排序）的方式只有一种，很单一。
- 一个类如果想要保持原样，又需要进行不同方式的比较（排序），就可以定制比较器（实现 Comparator 接口），向sort函数中穿一个比较器即可。
- Comparable 接口在 `java.lang` 包下，而 `Comparator` 接口在 `java.util` 包下，算不上是亲兄弟，但可以称得上是表（堂）兄弟。

一般我们需要对一个集合使用自定义排序时，我们就要重写`compareTo()`方法或`compare()`方法，当我们需要对某一个集合实现两种排序方式，比如一个 song 对象中的歌名和歌手名分别采用一种排序方法的话，我们可以重写`compareTo()`方法和使用自制的`Comparator`方法或者以两个 Comparator 来实现歌名排序和歌星名排序，第二种代表我们只能使用两个参数版的 `Collections.sort()`

## 7.Queue&Deque接口详解

### I.Queue 与 Deque 的区别

`Queue` 是单端队列，只能从一端插入元素，另一端删除元素，实现上一般遵循 **先进先出（FIFO）** 规则。

`Queue` 扩展了 `Collection` 的接口，根据 **因为容量问题而导致操作失败后处理方式的不同** 可以分为两类方法: 一种在操作失败后会抛出异常，另一种则会返回特殊值。

| `Queue` 接口 | 抛出异常  | 返回特殊值 |
| ------------ | --------- | ---------- |
| 插入队尾     | add(E e)  | offer(E e) |
| 删除队首     | remove()  | poll()     |
| 查询队首元素 | element() | peek()     |

`Deque` 是双端队列，在队列的两端均可以插入或删除元素。

`Deque` 扩展了 `Queue` 的接口, 增加了在队首和队尾进行插入和删除的方法，同样根据失败后处理方式的不同分为两类：

| `Deque` 接口 | 抛出异常      | 返回特殊值      |
| ------------ | ------------- | --------------- |
| 插入队首     | addFirst(E e) | offerFirst(E e) |
| 插入队尾     | addLast(E e)  | offerLast(E e)  |
| 删除队首     | removeFirst() | pollFirst()     |
| 删除队尾     | removeLast()  | pollLast()      |
| 查询队首元素 | getFirst()    | peekFirst()     |
| 查询队尾元素 | getLast()     | peekLast()      |

事实上，`Deque` 还提供有 `push()` 和 `pop()` 等其他方法，可用于模拟栈。

### II.ArrayDeque的实现

*ArrayDeque*和*LinkedList*是*Deque*的两个通用实现，官方更推荐使用*AarryDeque*用作栈和队列，从名字可以看出*ArrayDeque*底层通过数组实现，**为了满足可以同时在数组两端插入或删除元素的需求**，该数组还必须是循环的，即**循环数组（circular array）**，也就是说数组的任何一点都可能被看作起点或者终点。

*ArrayDeque*是**非线程安全的**（not thread-safe），当多个线程同时使用的时候，需要手动同步；另外，**该容器不允许放入`null`元素。**

```java
 transient Object[] elements;

    /**
     * The index of the element at the head of the deque (which is the
     * element that would be removed by remove() or pop()); or an
     * arbitrary number 0 <= head < elements.length equal to tail if
     * the deque is empty.
     */
    transient int head;

    /**
     * The index at which the next element would be added to the tail
     * of the deque (via addLast(E), add(E), or push(E));
     * elements[tail] is always null.
     */
    transient int tail;
```

<img src="http://cdn.tobebetterjavaer.com/tobebetterjavaer/images/collection/arraydeque-1e7086a3-3d31-4553-aa16-5eaf2193649e.png" alt="ArrayDeque_base.png" style="zoom: 80%;" />

上图中我们看到，**`head`指向首端第一个有效元素，`tail`指向尾端第一个可以插入元素的空位**。因为是循环数组，所以`head`不一定总等于 0，`tail`也不一定总是比`head`大。

#### ~addFirst()

`addFirst(E e)`的作用是在*Deque*的首端插入元素，也就是在`head`的前面插入元素，在空间足够且下标没有越界的情况下，只需要将`elements[--head] = e`即可。

![ArrayDeque_addFirst.png](https://pdai.tech/_images/collection/ArrayDeque_addFirst.png)

实际需要考虑: 1.空间是否够用，以及2.下标是否越界的问题。上图中，如果`head`为`0`之后接着调用`addFirst()`，虽然空余空间还够用，但`head`为`-1`，下标越界了。下列代码很好的解决了这两个问题。

```Java
//addFirst(E e)
public void addFirst(E e) {
    if (e == null)//不允许放入null
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;//2.下标是否越界
    if (head == tail)//1.空间是否够用
        doubleCapacity();//扩容
}
```

------

下标越界的处理解决起来非常简单，`head = (head - 1) & (elements.length - 1)`就可以了，**这段代码相当于取余，同时解决了`head`为负值的情况**。因为`elements.length`必需是`2`的指数倍，`elements - 1`就是二进制低位全`1`，跟`head - 1`相与之后就起到了取模的作用，如果`head - 1`为负数(其实只可能是-1)，则相当于对其取相对于`elements.length - 1`本身。

#### ~扩容

下面再说说扩容函数`doubleCapacity()`，其逻辑是申请一个更大的数组(原数组的两倍)，然后将原数组复制过去。过程如下图所示:

![ArrayDeque_doubleCapacity.png](https://pdai.tech/_images/collection/ArrayDeque_doubleCapacity.png)

图中我们看到，复制分两次进行，**第一次复制`head`右边的元素，第二次复制`head`左边的元素**。

```java
//doubleCapacity()
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // head右边元素的个数
    int newCapacity = n << 1;//原空间的2倍
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    System.arraycopy(elements, p, a, 0, r);//复制右半部分，对应上图中绿色部分
    System.arraycopy(elements, 0, a, r, p);//复制左半部分，对应上图中灰色部分
    elements = (E[])a;
    head = 0;
    tail = n;
}
```

### III.ArrayDeque 与 LinkedList 的区别

`ArrayDeque` 和 `LinkedList` 都实现了 `Deque` 接口，两者都具有队列的功能，但两者有什么区别呢？

- `ArrayDeque` 是基于可变长的数组和双指针来实现，而 `LinkedList` 则通过链表来实现。
- `ArrayDeque` 不支持存储 `NULL` 数据，但 `LinkedList` 支持。
- `ArrayDeque` 是在 JDK1.6 才被引入的，而`LinkedList` 早在 JDK1.2 时就已经存在。
- `ArrayDeque` 插入时可能存在扩容过程, 不过均摊后的插入操作依然为 O(1)。虽然 `LinkedList` 不需要扩容，**但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢。**

从性能的角度上，**选用 `ArrayDeque` 来实现队列要比 `LinkedList` 更好**。此外，`ArrayDeque` 也可以用于实现栈。

### IV.PriorityQueue的实现

Java 中的 PriorityQueue 事通过二叉小顶堆实现的，可以用一棵完全二叉树表示。**优先队列的作用是能保证每次取出的元素都是队列中权值最小的**(Java的优先队列每次取最小元素，C++的优先队列每次取最大元素)。这里牵涉到了大小关系，**元素大小的评判可以通过元素本身的自然顺序(natural ordering)，也可以通过构造时传入的比较器**(***Comparator***)。

Java中*PriorityQueue*实现了*Queue*接口，不允许放入`null`元素；其通过堆实现，具体说是通过完全二叉树(*complete binary tree*)实现的**小顶堆**(任意一个非叶子节点的权值，都不大于其左右子节点的权值)，也就意味着可以通过数组来作为*PriorityQueue*的底层实现。

<img src="https://pdai.tech/_images/collection/PriorityQueue_base.png" alt="PriorityQueue_base.png" style="zoom: 50%;" />

父子节点的编号之间有如下关系:

```
leftNo = parentNo*2+1
rightNo = parentNo*2+2
parentNo = (nodeNo-1)/2
```

**通过上述三个公式，可以轻易计算出某个节点的父节点以及子节点的下标。这也就是为什么可以直接用数组来存储堆的原因。**

#### ~节点的调整

<img src="https://pdai.tech/_images/collection/PriorityQueue_offer.png" alt="PriorityQueue_offer.png" style="zoom:50%;" />

新加入的元素可能会破坏小顶堆的性质，因此需要进行必要的调整。

```Java
//offer(E e)
public boolean offer(E e) {
    if (e == null)//不允许放入null元素
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);//自动扩容
    size = i + 1;
    if (i == 0)//队列原来为空，这是插入的第一个元素
        queue[0] = e;
    else
        siftUp(i, e);//调整
    return true;
```

上述代码中，扩容函数`grow()`类似于`ArrayList`里的`grow()`函数，就是再申请一个更大的数组，并将原数组的元素复制过去，这里不再赘述。需要注意的是`siftUp(int k, E x)`方法，该方法用于插入元素`x`并维持堆的特性。

```Java
//siftUp()
private void siftUp(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;//parentNo = (nodeNo-1)/2
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)//调用比较器的比较方法
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```

新加入的元素`x`可能会破坏小顶堆的性质，因此需要进行调整。调整的过程为 : 从`k`指定的位置开始，将`x`逐层与当前点的`parent`进行比较并交换，直到满足`x >= queue[parent]`为止。注意这里的比较可以是元素的自然顺序，也可以是依靠比较器的顺序。

#### ~remove()和poll()

`remove()`和`poll()`方法的语义也完全相同，都是获取并删除队首元素，区别是当方法失败时前者抛出异常，后者返回`null`。由于删除操作会改变队列的结构，为维护小顶堆的性质，需要进行必要的调整。

<img src="https://pdai.tech/_images/collection/PriorityQueue_poll.png" alt="PriorityQueue_poll.png" style="zoom:50%;" />

上述代码首先记录`0`下标处的元素，并用**最后一个元素（树结构的右下角）**替换`0`下标位置的元素，之后调用`siftDown()`方法对堆进行调整，最后返回原来`0`下标处的那个元素(也就是最小的那个元素)。重点是`siftDown(int k, E x)`方法，该方法的作用是**从`k`指定的位置开始，将`x`逐层向下与当前点的左右孩子中较小的那个交换，直到`x`小于或等于左右孩子中的任何一个为止**。

```java
//siftDown()
private void siftDown(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
    	//首先找到左右孩子中较小的那个，记录到c里，并用child记录其下标
        int child = (k << 1) + 1;//leftNo = parentNo*2+1
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0)
            c = queue[child = right];
        if (comparator.compare(x, (E) c) <= 0)
            break;
        queue[k] = c;//然后用c取代原来的值
        k = child;
    }
    queue[k] = x;
}
```

#### ~PriorityQueue中方法的时间复杂度

- boolean add(E e)——将指定的元素插入到此优先级队列中。O(logN)
- boolean contains(Object o)——是否包含指定元素，包含返回 true。O(n)。方法直接调用indexOf方法查找指定元素索引，从源码中可以看到，查找索引是遍历一遍数组，时间是O(n)。
  所以总的时间复杂度是**O(n)**.
- E poll()——检索并删除此队列的头，如果此队列为空，则返回 null 。O(logN)，删除队头元素维护堆的时间是O(log),所以总的时间是**O(log)**。
- boolean remove(Object o)——从该队列中删除指定元素的单个实例（如果存在）。O(n)可以看出indexOf方法遍历了一遍数组，时间复杂度为O(n)，removeAt方法删除元素维护堆结构，时间复杂度为O(log)，所以总的时间复杂度为**O(n)**。

#### ~优先队列时间复杂度

<img src="https://img-blog.csdnimg.cn/afd984af60394954a4aebbf92b0d3a39.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAYmV5b25kLm15c2VsZg==,size_20,color_FFFFFF,t_70,g_se,x_16" alt="img" style="zoom:67%;" />

假设一个深度是h层的满二叉树(因为是考虑最坏情况，所以我们就算满二叉树时间复杂度即可)，第1层有2^0个节点:若执行向下调整法，下面有多少层就最多执行多少次，下面有h-1层:执行h-1次

第2层有2^1个节点:最多向下调整h-2层:执行h-2次

第3层有2^2个节点:最多向下调整h-3层:执行h-3次

……

第h-1层有2^ (h-2)个节点:最多向下调整h-1层:执行1次

第h层有2^ (h-1)个节点:最多向下调整h-1层:执行0次

把他们相乘再相加就是总的循环次数。总次数设为T(h)

①T(h)= 2^0*(h-1) + 2^1*(h-2) +  2^2*(h-3) + …… + 2^(h-2)* 1   + 2^(h-1)*0

等比数列×等差数列求和，继续用错位相减法：

②2T(h)=                 2^1*(h-1) + 2^2*(h-2) +  2^3*(h-3) + …… + 2^(h-1)* 1  

(错位相减法：等式两边同时乘q，再相减)     

②-①：

T(h) = -2^0*(h-1) + 2^1 +  2^2 + …… + 2^(h-1) 

       =  -2^0*(h-1) + 2*(1-2^(h-1))/(1-2)      等比数列求和公式：a1*(1-q^n)/(1-q)
    
       =  1-h + 2^h -2
    
       =  -1-h + 2^h

由：总结点数公式 N = 2^h - 1 , N + 1 = 2^h , h = log2(N+1) 带入上式：

T(h) = -1-log2(N+1)  + 2^log2(N+1) = -1-log2(N+1) + N+1 = -log2(N+1) + N

时间复杂度的计算省去所有常数和系数和对表达式影响小的项，因为logN相比N对表达式影响很小，所以省略logN，因此 -log2(N+1) + N 就是N

**求得 向下调整法建堆 的 时间复杂度就是 O(N)**

![img](https://img-blog.csdnimg.cn/24efe57fa1264309a223b76c59946b4a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAYmV5b25kLm15c2VsZg==,size_20,color_FFFFFF,t_70,g_se,x_16)

假设一个深度是h层的满二叉树(因为是考虑最坏情况，所以我们就算满二叉树时间复杂度即可)，它的第一层是1个节点： 2^0, 若执行向上调整法，就他自己一个没必要执行：比较0次

它的第二层是2个节点：2^1,若执行向上调整法，最多往上比较一层就够了：比较1次

它的第三层是4个节点： 2^2,若执行向上调整法，最多往上比较二层就够了：比较2次

……

它的第h层有这么多节点：2^(h-1),若执行向上调整法，最多往上比较h-1层就够了：比较h-1次

把他们相乘再相加就是总的循环次数。总次数设为T(h)

①T(h)= 2^0*0 + 2^1*1 +  2^2*2 + …… + 2^(h-1)*(h-1)  等比数列×等差数列求和，用错位相减法：

②2T(h)=           2^1*0 + 2^2*1 +  2^3*2 + …… 2^(h-1)*(h-2) + 2^h*(h-1)     

**(错位相减法：等式两边同时乘q，再相减)**     

②-①：

T(h) = -2^0*0 - 2^1 -  2^2 - …… -2^(h-1) + 2^h*(h-1) 

       = -( 2^1 + 2^2 + …… +2^(h-1)) ) + 2^h*(h-1)         等比数列求和公式：a1*(1-q^n)/(1-q)
    
       = -2*(1-2^(h-1))/(1-2) + 2^h*(h-1)
    
       = 2 - 2^h + 2^h*(h-1)
    
       = 2 + 2^h*(h-2)

由：总结点数公式 N = 2^h - 1 , N + 1 = 2^h , h = log2(N+1) 带入上式：

T(h) = 2 + 2^log2(N+1) * ( log2(N+1) - 2 ) = 2 + (N+1) * ( log2(N+1) - 2 )

时间复杂度的计算省去所有常数和系数，凡是log以几为底N的对数统统写成logN，

因此 2 + (N+1) * ( log2(N+1) - 2 ) 就是N*logN

**求得 向上调整法建堆 的 时间复杂度就是 O(N*logN)**

**结论：**

​    **向上调整法建堆** 的 时间复杂度是 **O(NlogN)**

​    **向下调整法建堆** 的 时间复杂度是 **O(N)**， **因此向下调整法建堆更优！！！**

![image-20221115142826382](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221115142826382.png)

## 8、Map接口

### I.HashMap的实现

#### ~JDK1.8 之前

JDK1.8 之前 `HashMap` 底层是 **数组和链表** 结合在一起使用也就是 **链表散列**。HashMap 通过 key 的 `hashcode` 经过扰动函数处理过后得到 hash 值，然后通过 `(n - 1) & hash` 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，**不相同就通过拉链法解决冲突。**

所谓扰动函数指的就是 HashMap 的 `hash` 方法。使用 `hash` 方法也就是扰动函数是为了防止一些实现比较差的 `hashCode()` 方法 换句话说使用扰动函数之后可以减少碰撞。

#### ~JDK1.8 之后

相比于之前的版本， JDK1.8 之后在解决哈希冲突时有了较大的变化，**当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64**，那么会选择先进行数组扩容（扩容后数组的长度为原来的2倍）而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

#### ~HashMap 的长度为什么是 2 的幂次方

**为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀**。我们上面也讲到了过了，Hash 值的范围值-2147483648 到 2147483647，前后加起来大概 40 亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ `(n - 1) & hash`”。（n 代表数组长度）。这也就解释了 HashMap 的长度为什么是 2 的幂次方。

2 ^n - 1转为二进制后相当于是一个“低位掩码”，该数有效位数都为1，其余高位都为0，这样就可以将哈希值的高位全部归零，只保留低位值，用来做数组下标访问。而这个机制恰好满足了取模运算，并且位运算效率比%运算符高很多，**%运算符在计算机底层要有很多条指令支持**。

**一句话，为了使用位运算来替代%实现Hash取模运算以提高访问数组下标的效率。**

#### ~加载因子为什么是0.75

- 数组的容量过小，经过哈希计算后的下标，容易出现冲突；
- 数组的容量过大，导致空间利用率不高。

加载因子是用来表示 HashMap 中数据的填满程度：

> 加载因子 = 填入哈希表中的数据个数 / 哈希表的长度

这就意味着：

- 加载因子越小，填满的数据就越少，哈希冲突的几率就减少了，但浪费了空间，而且还会提高扩容的触发几率；
- 加载因子越大，填满的数据就越多，空间利用率就高，但哈希冲突的几率就变大了。

这就必须在“**哈希冲突**”与“**空间利用率**”两者之间有所取舍，尽量保持平衡，谁也不碍着谁。

为了减少哈希冲突发生的概率，当 HashMap 的数组长度达到一个**临界值**的时候，就会触发扩容，扩容后会将之前小数组中的元素转移到大数组中，这是一个相当耗时的操作。

这个临界值由什么来确定呢？

> 临界值 = 初始容量 * 加载因子

为什么加载因子会选择 0.75 呢？为什么不是0.8、0.6呢？

这跟统计学里的一个很重要的原理——泊松分布有关。

是时候上维基百科了：

> 泊松分布，是一种统计与概率学里常见到的离散概率分布，由法国数学家西莫恩·德尼·泊松在1838年时提出。它会对随机事件的发生次数进行建模，适用于涉及计算在给定的时间段、距离、面积等范围内发生随机事件的次数的应用情形。

**在 HashMap 的 doc 文档里，曾有这么一段描述：**

```text
factorial(k)). The first values are:
0:    0.60653066
1:    0.30326533
2:    0.07581633
3:    0.01263606
4:    0.00157952
5:    0.00015795
6:    0.00001316
7:    0.00000094
8:    0.00000006
就是在理想情况下,使用随机哈希码,节点出现的频率在hash桶中遵循泊松分布，同时给出了桶中元素个数和概率的对照表。
从上面的表中可以看到当桶中元素到达8个的时候，概率已经变得非常小，也就是说用0.75作为加载因子，每个碰撞位置的链表长度超过８个是几乎不可能的。
```

理想情况下，我们使用随机的hashcode值，**加载因子为0.75情况**，尽管由于粒度调整会产生较大的方差，**节点的分布频率仍然会服从参数为0.5的泊松分布**。链表的**长度为 8 发生的概率仅有 0.00000006**。

虽然这段话的本意更多的是表示 jdk 8中为什么拉链长度超过8的时候进行了红黑树转换，但提到了 0.75 这个加载因子——但这并不是为什么加载因子是 0.75 的答案。

考虑到 HashMap的容量有一个要求：它必须是2的n 次幂。当加载因子选择了0.75就可以保证它与容量的乘积为整数。

```text
16*0.75=12
32*0.75=24
```

除了 0.75，0.5~1 之间还有 0.625（5/8）、0.875（7/8）可选，从中位数的角度，挑 0.75 比较完美。另外，维基百科上说，拉链法（解决哈希冲突的一种）的加载因子最好限制在 0.7-0.8以下，超过0.8，查表时的**CPU缓存不命中（cache missing）会按照指数曲线上升**。

**综上，0.75 是个比较完美的选择**。

#### ~扩容流程

resize() 方法用于初始化数组或数组扩容，每次扩容后，**容量为原来的 2 倍，并进行数据迁移。**

```java
// newCapacity为新的容量
void resize(int newCapacity) {
    // 小数组，临时过度下
    Entry[] oldTable = table;
    // 扩容前的容量
    int oldCapacity = oldTable.length;
    // MAXIMUM_CAPACITY 为最大容量，2 的 30 次方 = 1<<30
    if (oldCapacity == MAXIMUM_CAPACITY) {
        // 容量调整为 Integer 的最大值 0x7fffffff（十六进制）=2 的 31 次方-1
        threshold = Integer.MAX_VALUE;
        return;
    }

    // 初始化一个新的数组（大容量）
    Entry[] newTable = new Entry[newCapacity];
    // 把小数组的元素转移到大数组中
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    // 引用新的大数组
    table = newTable;
    // 重新计算阈值
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

#### ~线程安全问题

- **多线程下扩容会死循环**

众所周知，HashMap 是通过拉链法来解决哈希冲突的，也就是当哈希冲突时，会将相同哈希值的键值对通过链表的形式存放起来。

JDK 7 时，采用的是**头部插入的方式来存放链表的**，也就是**下一个冲突的键值对会放在上一个键值对的前面**（同一位置上的新元素被放在链表的头部）。并且扩容时时**先扩容再插值**（JDK 8变为先插值再扩容），**扩容的时候就有可能导致出现环形链表，造成死循环。**

**不过，JDK 8 时已经修复了这个问题，扩容时会保持链表原来的顺序。**

- **多线程下 put 会导致元素丢失**

正常情况下，当发生哈希冲突时，HashMap 是这样的：

<img src="http://cdn.tobebetterjavaer.com/tobebetterjavaer/images/collection/hashmap-thread-nosafe-08.png" alt="img" style="zoom:50%;" />

但多线程同时执行 put 操作时，**如果计算出来的索引位置是相同的，那会造成前一个 key 被后一个 key 覆盖，从而导致元素的丢失。**

- **put 和 get 并发时会导致 get 到 null**

线程 A 执行put时，因为元素个数超出阈值而出现扩容，线程B 此时执行get，有可能导致这个问题。

### II.HashMap 和 Hashtable 的区别

- **线程是否安全：** `HashMap` 是非线程安全的，`Hashtable` 是线程安全的,因为 `Hashtable` 内部的方法基本都经过`synchronized` 修饰，**锁的是整个实例**。（如果你要保证线程安全的话就使用 `ConcurrentHashMap` 吧！）；
- **效率：** 因为线程安全的问题，`HashMap` 要比 `Hashtable` 效率高一点。另外，`Hashtable` 基本被淘汰，不要在代码中使用它；
- **对 Null key 和 Null value 的支持：** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；Hashtable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
- **初始容量大小和每次扩充容量大小的不同 ：** ① 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。② 创建时如果给定了容量初始值，那么 `Hashtable` 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。
- **底层数据结构：** JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树），以减少搜索时间（后文中我会结合源码对这一过程进行分析）。**`Hashtable` 没有这样的机制。**

### III.HashMap 和 HashSet 区别

*HashSet*是对*HashMap*的简单包装，对*HashSet*的函数调用都会转换成合适的*HashMap*方法，因此*HashSet*的实现非常简单，（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

```Java
//HashSet是对HashMap的简单包装
public class HashSet<E>
{
	......
	private transient HashMap<E,Object> map;//HashSet里面有一个HashMap
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
    public HashSet() {
        map = new HashMap<>();
    }
    ......
    public boolean add(E e) {//简单的方法转换
        return map.put(e, PRESENT)==null;
    }
    ......
}
```

### IV.TreeMap

#### ~概述：

Java *TreeMap*实现了*SortedMap*接口，也就是说会按照`key`的大小顺序对*Map*中的元素进行排序，`key`大小的评判可以通过其本身的自然顺序(natural ordering)，也可以通过构造时传入的比较器(Comparator)。

**TreeMap底层通过红黑树(Red-Black tree)实现**，也就意味着`containsKey()`, `get()`, `put()`, `remove()`都有着**`log(n)`**的时间复杂度。

出于性能原因，*TreeMap*是非同步的(not synchronized)，如果需要在多线程环境使用，需要程序员手动同步；或者通过如下方式将*TreeMap*包装成(wrapped)同步的:

```
SortedMap m = Collections.synchronizedSortedMap(new TreeMap(...));
```

**红黑树是一种近似平衡的二叉查找树，它能够确保任何一个节点的左右子树的高度差不会超过二者中较低那个的一倍**。具体来说，红黑树是满足如下条件的二叉查找树(binary search tree):

1. 每个节点要么是红色，要么是黑色。
2. 根节点必须是黑色
3. 红色节点不能连续(也即是，红色节点的孩子和父亲都不能是红色)。
4. 对于每个节点，从该点至`null`(树尾端)的任何路径，都含有相同个数的黑色节点。

在树的结构发生改变时(插入或者删除操作)，往往会破坏上述条件3或条件4，需要通过调整使得查找树重新满足红黑树的约束条件。

```java
private static final boolean RED   = false;
private static final boolean BLACK = true;

static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;
//…………
}
```



<img src="https://pdai.tech/_images/collection/TreeMap_base.png" alt="TreeMap_base.png" style="zoom:67%;" />

#### ~HashMap 和 TreeMap 区别

`TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但是需要注意的是`TreeMap`它还实现了`NavigableMap`接口和`SortedMap` 接口。

![TreeMap 继承关系图](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/treemap_hierarchy.png)

实现 `NavigableMap` 接口让 `TreeMap` 有了对集合内元素的搜索的能力。

实现`SortedMap`接口让 `TreeMap` 有了对集合中的元素根据键排序的能力。**默认是按 key 的升序排序，不过我们也可以指定排序的比较器。**

**综上，相比于`HashMap`来说 `TreeMap` 主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力。**

### V.LinkedHashMap

*LinkedHashMap*实现了*Map*接口，即允许放入`key`为`null`的元素，也允许插入`value`为`null`的元素。从名字上可以看出该容器是***LinkedList***和***HashMap***的混合体，也就是说它同时满足***HashMap***和***LinkedList***的某些特性。可将LinkedHashMap看作采用LinkedList增强的HashMap。

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
```



<img src="https://pdai.tech/_images/collection/LinkedHashMap_base.png" alt="LinkedHashMap_base.png" style="zoom:50%;" />

事实上*LinkedHashMap*是*HashMap*的直接子类，**二者唯一的区别是\*LinkedHashMap\*在\*HashMap\*的基础上，采用双向链表(doubly-linked list)的形式将所有`entry`连接起来，这样是为保证元素的迭代顺序跟插入顺序相同**。

上图给出了*LinkedHashMap*的结构图，主体部分跟*HashMap*完全一样，多了`header`指向双向链表的头部(是一个哑元)，**该双向链表的迭代顺序就是`entry`的插入顺序**。

除了可以保迭代历顺序，这种结构还有一个好处 : **迭代\*LinkedHashMap\*时不需要像\*HashMap\*那样遍历整个`table`，而只需要直接遍历`header`指向的双向链表即可**，也就是说*LinkedHashMap*的迭代时间就只跟`entry`的个数相关，而跟`table`的大小无关。

有两个参数可以影响*LinkedHashMap*的性能: **初始容量(inital capacity)和负载系数(load factor)。这两个值是由super()调用父类构造函数来实现的**初始容量指定了初始`table`的大小，负载系数用来指定自动扩容的临界值。当`entry`的数量超过`capacity*load_factor`时，容器将自动扩容并重新哈希。对于插入元素较多的场景，将初始容量设大可以减少重新哈希的次数。

```java
 public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        putMapEntries(m, false);
    }
```

#### LinkedHashMap经典用法

*LinkedHashMap*除了可以保证迭代顺序外，还有一个非常有用的用法: 可以轻松实现一个采用了FIFO替换策略的缓存。具体说来，LinkedHashMap有一个子类方法`protected boolean removeEldestEntry(Map.Entry<K,V> eldest)`，该方法的作用是告诉Map是否要删除“最老”的Entry，所谓最老就是当前Map中最早插入的Entry，如果该方法返回`true`，最老的那个元素就会被删除。在每次插入新元素的之后LinkedHashMap会自动询问removeEldestEntry()是否要删除最老的元素。这样只需要在子类中重载该方法，当元素个数超过一定数量时让removeEldestEntry()返回true，就能够实现一个固定大小的FIFO策略的缓存。示例代码如下:

```Java
/** 一个固定大小的FIFO替换策略的缓存 */
class FIFOCache<K, V> extends LinkedHashMap<K, V>{
    private final int cacheSize;
    public FIFOCache(int cacheSize){
        this.cacheSize = cacheSize;
    }

    // 当Entry个数超过cacheSize时，删除最老的Entry
    @Override
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
       return size() > cacheSize;
    }
```

# 五、反射详解

RTTI（Run-Time Type Identification）运行时类型识别。在《Thinking in Java》一书第十四章中有提到，其作用是在**运行时识别一个对象的类型和类的信息。**主要有两种方式：一种是“传统的”RTTI，它假定我们在编译时已经知道了所有的类型；另一种是“反射”机制，它允许我们在运行时发现和使用类的信息。

**反射就是把java类中的各种成分映射成一个个的Java对象**

## 1.Class类

Class类，Class类也是一个实实在在的类，存在于JDK的java.lang包中。Class类的实例表示java应用运行时的类(class ans enum)或接口(interface and annotation)

每个java类运行时都在JVM里表现为一个class对象，可通过以下方法来获取Class对象：

- **类名.class**
- **对象引用.getClass()**
- **Class.forName("全类名")**

```java
public final class Class<T> implements java.io.Serializable,
                              GenericDeclaration,
                              Type,
                              AnnotatedElement {
    private static final int ANNOTATION= 0x00002000;
    private static final int ENUM      = 0x00004000;
    private static final int SYNTHETIC = 0x00001000;

    private static native void registerNatives();
    static {
        registerNatives();
    }

    /*
     * Private constructor. Only the Java Virtual Machine creates Class objects.   //私有构造器，只有JVM才能调用创建Class对象
     * This constructor is not used and prevents the default constructor being
     * generated.
     */
    private Class(ClassLoader loader) {
        // Initialize final field for classLoader.  The initialization value of non-null
        // prevents future JIT optimizations from assuming this final field is null.
        classLoader = loader;
    }
```

到这我们也就可以得出以下几点信息：

- Class类也是类的一种，与class关键字是不一样的。
- 手动编写的类被编译后会产生一个Class对象，其表示的是创建的类的类型信息，而且这个Class对象保存在同名.class的文件中(字节码文件)
- 每个通过关键字class标识的类，在内存中有且只有一个与之对应的Class对象来描述其类型信息，无论创建多少个实例对象，其依据的都是用一个Class对象。
- Class类只存私有构造函数，因此对应Class对象只能有JVM创建和加载
- Class类的对象作用是运行时提供或获得某个对象的类型信息

## 2.反射的使用

### I.Class类对象的获取

在类加载的时候，jvm会创建一个class对象

class对象是可以说是反射中最常用的，获取class对象的方式的主要有三种

- 根据类名：类名.class
- 根据对象：对象.getClass()
- 根据全限定类名：Class.forName(全限定类名)

### II.Class类中的方法

| 方法名                                              | 说明                                                         |
| --------------------------------------------------- | ------------------------------------------------------------ |
| forName()                                           | (1)获取Class对象的一个引用，但引用的类还没有加载(该类的第一个对象没有生成)就加载了这个类。 |
| (2)为了产生Class引用，forName()立即就进行了初始化。 |                                                              |
| Object-getClass()                                   | 获取Class对象的一个引用，返回表示该对象的实际类型的Class引用。 |
| getName()                                           | 取全限定的类名(包括包名)，即类的完整名字。                   |
| getSimpleName()                                     | 获取类名(不包括包名)                                         |
| getCanonicalName()                                  | 获取全限定的类名(包括包名)                                   |
| isInterface()                                       | 判断Class对象是否是表示一个接口                              |
| getInterfaces()                                     | 返回Class对象数组，表示Class对象所引用的类所实现的所有接口。 |
| getSupercalss()                                     | 返回Class对象，表示Class对象所引用的类所继承的直接基类。应用该方法可在运行时发现一个对象完整的继承结构。 |
| newInstance()                                       | 返回一个Oject对象，是实现“虚拟构造器”的一种途径。使用该方法创建的类，必须带有无参的构造器。 |
| getFields()                                         | 获得某个类的所有的公共（public）的字段，包括继承自父类的所有公共字段。 类似的还有getMethods和getConstructors。 |
| getDeclaredFields                                   | 获得某个类的自己声明的字段，即包括public、private和proteced，默认但是不包括父类声明的任何字段。类似的还有getDeclaredMethods和getDeclaredConstructors。 |

### III.Constructor类及其用法

> Constructor类存在于反射包(java.lang.reflect)中，反映的是Class 对象所表示的类的构造方法。

获取Constructor对象是通过Class类中的方法获取的，Class类与Constructor相关的主要方法如下：

获取Constructor对象是通过Class类中的方法获取的，Class类与Constructor相关的主要方法如下：

| 方法返回值       | 方法名称                                           | 方法说明                                                  |
| ---------------- | -------------------------------------------------- | --------------------------------------------------------- |
| static Class<?>  | forName(String className)                          | 返回与带有给定字符串名的类或接口相关联的 Class 对象。     |
| Constructor      | getConstructor(Class<?>... parameterTypes)         | 返回指定参数类型、具有public访问权限的构造函数对象        |
| Constructor<?>[] | getConstructors()                                  | 返回所有具有public访问权限的构造函数的Constructor对象数组 |
| Constructor      | getDeclaredConstructor(Class<?>... parameterTypes) | 返回指定参数类型、所有声明的（包括private）构造函数对象   |
| Constructor<?>[] | getDeclaredConstructors()                          | 返回所有声明的（包括private）构造函数对象                 |
| T                | newInstance()                                      | 调用无参构造器创建此 Class 对象所表示的类的一个新实例。   |

### IV.…………

### V.反射的主要应用场景有：

- **开发通用框架**：像 Spring，为了保持通用性，通过配置文件来加载不同的对象，调用不同的方法。
- **动态代理**：在面向切面编程中，需要拦截特定的方法，就会选择动态代理的方式，而动态代理的底层技术就是反射。
- **注解**：注解本身只是起到一个标记符的作用，它需要利用发射机制，根据标记符去执行特定的行为。

## 3.反射机制执行的流程

### I.反射获取类实例

首先调用了 java.lang.Class 的静态方法，获取类信息。

```java
    @CallerSensitive
    public static Class<?> forName(String className)
                throws ClassNotFoundException {
        // 先通过反射，获取调用进来的类信息，从而获取当前的 classLoader
        Class<?> caller = Reflection.getCallerClass();
        // 调用native方法进行获取class信息
        return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
    }
```

forName()反射获取类信息，并没有将实现留给了java,而是交给了jvm去加载。

主要是先获取 ClassLoader, 然后调用 native 方法，获取信息，加载类则是回调 java.lang.ClassLoader.

最后，**jvm又会回调 ClassLoader 进类加载，加载过程中是线程安全的。**

```java
  public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }
    
        // sun.misc.Launcher
        public Class<?> loadClass(String var1, boolean var2) throws ClassNotFoundException {
            int var3 = var1.lastIndexOf(46);
            if(var3 != -1) {
                SecurityManager var4 = System.getSecurityManager();
                if(var4 != null) {
                    var4.checkPackageAccess(var1.substring(0, var3));
                }
            }

            if(this.ucp.knownToNotExist(var1)) {
                Class var5 = this.findLoadedClass(var1);
                if(var5 != null) {
                    if(var2) {
                        this.resolveClass(var5);
                    }

                    return var5;
                } else {
                    throw new ClassNotFoundException(var1);
                }
            } else {
                return super.loadClass(var1, var2);
            }
        }
    // java.lang.ClassLoader
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        // 先获取锁
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            // 如果已经加载了的话，就不用再加载了
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 双亲委托加载
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                // 父类没有加载到时，再自己加载
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
    
    protected Object getClassLoadingLock(String className) {
        Object lock = this;
        if (parallelLockMap != null) {
            // 使用 ConcurrentHashMap来保存锁
            Object newLock = new Object();
            lock = parallelLockMap.putIfAbsent(className, newLock);
            if (lock == null) {
                lock = newLock;
            }
        }
        return lock;
    }
    
    protected final Class<?> findLoadedClass(String name) {
        if (!checkName(name))
            return null;
        return findLoadedClass0(name);
    }
```

### II. 创建对象——newInstance() 

```java
// 首先肯定是 Class.newInstance
    @CallerSensitive
    public T newInstance()
        throws InstantiationException, IllegalAccessException
    {
        if (System.getSecurityManager() != null) {
            checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), false);
        }

        // NOTE: the following code may not be strictly correct under
        // the current Java memory model.

        // Constructor lookup
        // newInstance() 其实相当于调用类的无参构造函数，所以，首先要找到其无参构造器
        if (cachedConstructor == null) {
            if (this == Class.class) {
                // 不允许调用 Class 的 newInstance() 方法
                throw new IllegalAccessException(
                    "Can not call newInstance() on the Class for java.lang.Class"
                );
            }
            try {
                // 获取无参构造器
                Class<?>[] empty = {};
                final Constructor<T> c = getConstructor0(empty, Member.DECLARED);
                // Disable accessibility checks on the constructor
                // since we have to do the security check here anyway
                // (the stack depth is wrong for the Constructor's
                // security check to work)
                java.security.AccessController.doPrivileged(
                    new java.security.PrivilegedAction<Void>() {
                        public Void run() {
                                c.setAccessible(true);
                                return null;
                            }
                        });
                cachedConstructor = c;
            } catch (NoSuchMethodException e) {
                throw (InstantiationException)
                    new InstantiationException(getName()).initCause(e);
            }
        }
        Constructor<T> tmpConstructor = cachedConstructor;
        // Security check (same as in java.lang.reflect.Constructor)
        int modifiers = tmpConstructor.getModifiers();
        if (!Reflection.quickCheckMemberAccess(this, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            if (newInstanceCallerCache != caller) {
                Reflection.ensureMemberAccess(caller, this, null, modifiers);
                newInstanceCallerCache = caller;
            }
        }
        // Run constructor
        try {
            // 调用无参构造器
            return tmpConstructor.newInstance((Object[])null);
        } catch (InvocationTargetException e) {
            Unsafe.getUnsafe().throwException(e.getTargetException());
            // Not reached
            return null;
        }
    }
```

newInstance() 主要做了三件事：

- 1. 权限检测，如果不通过直接抛出异常；
- 1. 查找无参构造器，并将其缓存起来；
- 1. 调用具体方法的无参构造方法，生成实例并返回；

### 反射调用流程小结

1. 反射类及反射方法的获取，都是通过从列表中搜寻查找匹配的方法，所以查找性能会随类的大小方法多少而变化；
2. 每个类都会有一个与之对应的Class实例，从而每个类都可以获取method反射方法，并作用到其他实例身上；
3. 反射也是考虑了线程安全的，放心使用；
4. 反射使用软引用relectionData缓存class信息，避免每次重新从jvm获取带来的开销；
5. 反射调用多次生成新代理Accessor, 而通过字节码生存的则考虑了卸载功能，所以会使用独立的类加载器；
6. 当找到需要的方法，都会copy一份出来，而不是使用原来的实例，从而保证数据隔离；
7. 调度反射方法，最终是由jvm执行invoke0()执行；

# 六、注解机制详解

注解（Annotation）是在 Java 1.5 时引入的概念，同 class 和 interface 一样，也属于一种类型。注解提供了一系列数据用来装饰程序代码（类、方法、字段等），但是注解并不是所装饰代码的一部分，它对代码的运行效果没有直接影响，由编译器决定该执行哪些操作。

它主要的作用有以下四方面：

- 生成文档，通过代码里标识的元数据生成javadoc文档。
- 编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
- 编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
- 运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例。

下注解的常见分类：

- **Java自带的标准注解**，包括`@Override`、`@Deprecated`和`@SuppressWarnings`，分别用于标明重写某个方法、标明某个类或方法过时、标明要忽略的警告，用这些注解标明后编译器就会进行检查。
- **元注解**，元注解是用于定义注解的注解，包括`@Retention`、`@Target`、`@Inherited`、`@Documented`，`@Retention`用于标明注解被保留的阶段，`@Target`用于标明注解使用的范围，`@Inherited`用于标明注解可继承，`@Documented`用于标明是否生成javadoc文档。
- **自定义注解**，可以根据自己的需求定义注解，并可用元注解对自定义注解进行注解

## 1.注解分类详解

### I.Java内置注解

Java 1.5开始自带的标准注解，包括`@Override`、`@Deprecated`和`@SuppressWarnings`：

- `@Override`：表示当前的方法定义将覆盖父类中的方法
- `@Deprecated`：表示代码被弃用，如果使用了被@Deprecated注解的代码则编译器将发出警告
- `@SuppressWarnings`：表示关闭编译器警告信息

#### 内置注解 - @Override

告诉编译器被修饰的方法是重写的父类的中的相同签名的方法，编译器会对此做出检查，**若发现父类中不存在这个方法或是存在的方法签名不同，则会报错。**

```java
@Target(ElementType.METHOD) //表示修饰方法
@Retention(RetentionPolicy.SOURCE) //只在编译时有效，在编译后的class文件中便不再存在
public @interface Override {
}
```

#### 内置注解 - @Deprecated

**这个注解的作用是告诉编译器被修饰的程序元素已被“废弃”，不再建议用户使用。**

这个注解的定义如下：

```java
@Documented//它会被文档化
@Retention(RetentionPolicy.RUNTIME)//能够保留到运行时
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})//能够修饰构造方法、属性、局部变量、方法、包、参数、类型。
public @interface Deprecated {
}
```

#### 内置注解 - @SuppressWarnings

**它的作用是告诉编译器忽略指定的警告信息**

这个注解我们也比较常用到，先来看下它的定义：

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})//能够修饰的程序元素包括类型、属性、方法、参数、构造器、局部变量
@Retention(RetentionPolicy.SOURCE)//只能存活在源码里
public @interface SuppressWarnings {
    String[] value();
}
```

### II.元注解

在JDK 1.5中提供了4个标准的元注解：`@Target`，`@Retention`，`@Documented`，`@Inherited`, 在JDK 1.8中提供了两个元注解 `@Repeatable`和`@Native`。

####  元注解 - @Target

Target注解用来说明那些被它所注解的注解类可修饰的对象范围：

```java
public enum ElementType {
 
    TYPE, // 类、接口、枚举类
 
    FIELD, // 成员变量（包括：枚举常量）
 
    METHOD, // 成员方法
 
    PARAMETER, // 方法参数
 
    CONSTRUCTOR, // 构造方法
 
    LOCAL_VARIABLE, // 局部变量
 
    ANNOTATION_TYPE, // 注解类
 
    PACKAGE, // 可用于修饰：包
 
    TYPE_PARAMETER, // 类型参数，JDK 1.8 新增
 
    TYPE_USE // 使用类型的任何地方，JDK 1.8 新增
}
```

#### 元注解 - @Retention & @RetentionTarget

Reteniton注解的作用是：描述注解保留的时间范围（即：被描述的注解在它所修饰的类中可以被保留到何时） 。

```java
public enum RetentionPolicy {
 
    SOURCE,    // 源文件保留
    CLASS,       // 编译期保留，默认值
    RUNTIME   // 运行期保留，可通过反射去获取注解信息
}
```

#### 元注解 - @Documented

Documented注解的作用是：描述在使用 javadoc 工具为类生成帮助文档时是否要保留其注解信息。

以下代码在使用Javadoc工具可以生成`@TestDocAnnotation`注解信息。

```java
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;
 
@Documented
@Target({ElementType.TYPE,ElementType.METHOD})
public @interface TestDocAnnotation {
 
	public String value() default "default";
}
@TestDocAnnotation("myMethodDoc")
public void testDoc() {

}
```

#### 元注解 - @Inherited

Inherited注解的作用：**被它修饰的Annotation将具有继承性。**

即如果**某个类使用了被@Inherited修饰的Annotation，则其子类将自动具有该注解。**

#### 元注解 - @Repeatable (Java8)



#### 元注解 - @Native (Java8)

使用 @Native 注解修饰成员变量，则表示这个变量**可以被本地代码引用**，常常被代码生成工具使用。对于 @Native 注解不常使用

### III.注解与反射接口

:question: **定义注解后，如何获取注解中的内容呢？**

反射包**`java.lang.reflect`**下的**`AnnotatedElement`**接口提供这些方法。这里注意：**只有注解被定义为RUNTIME后，该注解才能是运行时可见**，当class文件被装载时被保存在class文件中的Annotation才会被虚拟机读取。

A**nnotatedElement 接口是所有程序元素（Class、Method和Constructor）的父接口**，所以程序通过反射获取了某个类的AnnotatedElement对象之后，程序就可以调用该对象的方法来访问Annotation信息。我们看下具体的先关接口：

- `boolean isAnnotationPresent(Class<?extends Annotation> annotationClass)`

判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false。注意：此方法会忽略注解对应的注解容器。

- `<T extends Annotation> T getAnnotation(Class<T> annotationClass)`

返回该程序元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。

- `Annotation[] getAnnotations()`

返回该程序元素上存在的所有注解，若没有注解，返回长度为0的数组。

- `<T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass)`

返回该程序元素上存在的、指定类型的注解数组。没有注解对应类型的注解时，返回长度为0的数组。该方法的调用者可以随意修改返回的数组，而不会对其他调用者返回的数组产生任何影响。`getAnnotationsByType`方法与 `getAnnotation`的区别在于，`getAnnotationsByType`会检测注解对应的重复注解容器。若程序元素为类，当前类上找不到注解，且该注解为可继承的，则会去父类上检测对应的注解。

- `<T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass)`

返回直接存在于此元素上的所有注解。与此接口中的其他方法不同，该方法将忽略继承的注释。如果没有注释直接存在于此元素上，则返回null

- `<T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass)`

返回直接存在于此元素上的所有注解。与此接口中的其他方法不同，该方法将忽略继承的注释

- `Annotation[] getDeclaredAnnotations()`

返回直接存在于此元素上的所有注解及注解对应的重复注解容器。与此接口中的其他方法不同，该方法将忽略继承的注解。如果没有注释直接存在于此元素上，则返回长度为零的一个数组。该方法的调用者可以随意修改返回的数组，而不会对其他调用者返回的数组产生任何影响。

