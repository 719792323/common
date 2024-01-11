### 标识符和关键字的区别是什么？

在我们编写程序的时候，需要大量地为程序、类、变量、方法等取名字，于是就有了 **标识符** 。简单来说， **标识符就是一个名字** 。

有一些标识符，Java 语言已经赋予了其特殊的含义，只能用于特定的地方，这些特殊的标识符就是 **关键字** 。简单来说，**关键字是被赋予特殊含义的标识符** 。比如，在我们的日常生活中，如果我们想要开一家店，则要给这个店起一个名字，起的这个“名字”就叫标识符。但是我们店的名字不能叫“警察局”，因为“警察局”这个名字已经被赋予了特殊的含义，而“警察局”就是我们日常生活中的关键字。

| 分类                 | 关键字   |            |              |              |               |           |        |
| :------------------- | -------- | ---------- | ------------ | ------------ | ------------- | --------- | ------ |
| 访问控制             | private  | protected  | public       |              |               |           |        |
| 类，方法和变量修饰符 | abstract | class      | extends      | final        | implements    | interface | native |
|                      | new      | static     | **strictfp** | synchronized | **transient** | volatile  | enum   |
| 程序控制             | break    | continue   | return       | do           | while         | if        | else   |
|                      | for      | instanceof | switch       | case         | default       | assert    |        |
| 错误处理             | try      | catch      | throw        | throws       | finally       |           |        |
| 包相关               | import   | package    |              |              |               |           |        |
| 基本类型             | boolean  | byte       | char         | double       | float         | int       | long   |
|                      | short    |            |              |              |               |           |        |
| 变量引用             | super    | this       | void         |              |               |           |        |
| 保留字               | goto     | const      |              |              |               |           |        |

------

###  移位运算符

Java 中有三种移位运算符：

![Java 移位运算符总结](https://oss.javaguide.cn/github/javaguide/java/basis/shift-operator.png)Java 移位运算符总结

- `<<` :左移运算符，向左移若干位，高位丢弃，低位补零。`x << 1`,相当于 x 乘以 2(不溢出的情况下)。
- `>>` :带符号右移，向右移若干位，高位补符号位，低位丢弃。正数高位补 0,负数高位补 1。`x >> 1`,相当于 x 除以 2。
- `>>>` :无符号右移，忽略符号位，空位都以 0 补齐。(最高位补0)

由于 `double`，`float` 在二进制中的表现比较特殊，因此不能来进行移位操作。

移位操作符实际上支持的类型只有`int`和`long`，编译器在对`short`、`byte`、`char`类型进行移位前，都会将其转换为`int`类型再操作。

**如果移位的位数超过数值所占有的位数会怎样？**

当 int 类型左移/右移位数大于等于 32 位操作时，会先求余（%）后再进行左移/右移操作。也就是说左移/右移 32 位相当于不进行移位操作（32%32=0），左移/右移 42 位相当于左移/右移 10 位（42%32=10）。当 long 类型进行左移/右移操作时，由于 long 对应的二进制是 64 位，因此求余操作的基数也变成了 64。

也就是说：`x<<42`等同于`x<<10`，`x>>42`等同于`x>>10`，`x >>>42`等同于`x >>> 10`。

------

### Java 基本数据类型大小

Java 中有 8 种基本数据类型，分别为：

- 6 种数字类型： 
  - 4 种整数型：`byte`、`short`、`int`、`long`
  - 2 种浮点型：`float`、`double`
- 1 种字符类型：`char`
- 1 种布尔型：`boolean`。

这 8 种基本数据类型的默认值以及所占空间的大小如下：

| 基本类型    | 位数 | 字节 | 默认值  | 取值范围                                                     |
| :---------- | :--- | :--- | :------ | ------------------------------------------------------------ |
| `byte`      | 8    | 1    | 0       | -128 ~ 127                                                   |
| **`short`** | 16   | 2    | 0       | -32768（-2^15） ~ 32767（2^15 - 1）                          |
| `int`       | 32   | 4    | 0       | -2147483648 ~ 2147483647                                     |
| `long`      | 64   | 8    | 0L      | -9223372036854775808（-2^63） ~ 9223372036854775807（2^63 -1） |
| **`char`**  | 16   | 2    | 'u0000' | 0 ~ 65535（2^16 - 1）                                        |
| `float`     | 32   | 4    | 0f      | 1.4E-45 ~ 3.4028235E38                                       |
| `double`    | 64   | 8    | 0d      | 4.9E-324 ~ 1.7976931348623157E308                            |
| `boolean`   | 1    |      | false   | true、false                                                  |

------

### 基本类型和包装类型的区别（如int和Integer）？

![基本类型 vs 包装类型](https://oss.javaguide.cn/github/javaguide/java/basis/primitive-type-vs-packaging-type.png)

- **用途**：除了定义一些常量和局部变量之外（**基本类型用在这**），我们在其他地方比如**方法参数、对象属性**(**包装类型用在这**)中很少会使用基本类型来定义变量。并且，**包装类型可用于泛型，而基本类型不可以**。
- **存储方式**：基本数据类型的**局部变量存放在 Java 虚拟机栈中的局部变量表**中，**基本数据类型的成员变量（未被 `static` 修饰 ）存放在 Java 虚拟机的堆中（被static修饰放在方法区中）**。包装类型属于对象类型，我们知道几乎所有对象实例都存在于堆中。
- **占用空间**：相比于包装类型（对象类型）， 基本数据类型占用的空间往往非常小。
- **默认值**：成员变量包装类型不赋值就是 `null` ，而基本类型有默认值且不是 `null`。
- **比较方式**：***对于基本数据类型来说，`==` 比较的是值。对于包装数据类型来说，`==` 比较的是对象的内存地址***。所有整型包装类对象之间值的比较，全部使用 `equals()` 方法。

**为什么说是几乎所有对象实例都存在于堆中呢？** 这是因为 HotSpot 虚拟机引入了 JIT 优化之后，会对对象进行逃逸分析，如果发现某一个对象并没有逃逸到方法外部，那么就可能通过标量替换来实现栈上分配，而避免堆上分配内存

------

### 包装类型的缓存机制了解么？

Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能。

`Byte`,`Short`,`Integer`,`Long` 这 4 种包装类默认创建了数值 **[-128，127]** 的相应类型的缓存数据，`Character` 创建了数值在 **[0,127]** 范围的缓存数据，`Boolean` 直接返回 `True` or `False`。**两种浮点数类型的包装类 `Float`,`Double` 并没有实现缓存机制**。

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

**所有整型包装类对象之间值的比较，全部使用 equals 方法比较**，因为如Integer在-128-127由于缓存用==比较结果为true，但是超出这个范围即使对应的值相等用==比较会是false

------

### 自动装箱与拆箱了解吗？原理是什么？

**什么是自动拆装箱？**

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；

举例：



```java
Integer i = 10;  //装箱
int n = i;   //拆箱
```

- `Integer i = 10` 等价于 `Integer i = Integer.valueOf(10)`
- `int n = i` 等价于 `int n = i.intValue()`;

注意：**如果频繁拆装箱的话，也会严重影响系统的性能。我们应该尽量避免不必要的拆装箱操作。**



```java
private static long sum() {
    // 应该使用 long 而不是 Long
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

------

### 浮点数运算的时候会有精度丢失的风险

浮点数运算精度丢失代码演示：



```java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.println(a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
```

为什么会出现这个问题呢？

这个和计算机保存浮点数的机制有很大关系。我们知道计算机是二进制的，而且计算机在表示一个数字时，宽度是有限的，无限循环的小数存储在计算机时，只能被截断，所以就会导致小数精度发生损失的情况。这也就是解释了为什么浮点数没有办法用二进制精确表示。

就比如说十进制下的 0.2 就没办法精确转换成二进制小数：



```java
// 0.2 转换为二进制数的过程为，不断乘以 2，直到不存在小数为止，
// 在这个计算过程中，得到的整数部分从上到下排列就是二进制的结果。
0.2 * 2 = 0.4 -> 0
0.4 * 2 = 0.8 -> 0
0.8 * 2 = 1.6 -> 1
0.6 * 2 = 1.2 -> 1
0.2 * 2 = 0.4 -> 0（发生循环）
...
```

* 解决浮点数运算的精度丢失问题

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

  

------

### 超过 long 整型的数据应该如何表示

基本数值类型都有一个表达范围，如果超过这个范围就会有数值溢出的风险。

在 Java 中，64 位 long 整型是最大的整数类型。



```java
long l = Long.MAX_VALUE;
System.out.println(l + 1); // -9223372036854775808
System.out.println(l + 1 == Long.MIN_VALUE); // true
```

`BigInteger` 内部使用 `int[]` 数组来存储任意大小的整形数据。

相对于常规整数类型的运算来说，`BigInteger` 运算的效率会相对较低。

------

