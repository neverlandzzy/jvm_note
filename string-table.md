# 字符串常量池\(String Table, String Constant Pool\)

## String的基本特性

String 实例化的两种方式

* String s1 = "hello"; //字面量定义方式 - 存储在字符串常量池中
* String s2 = new String\("world"\);

String声明为final，不可被继承

String实现了Serializable接口，支持序列化，实现了Comparable接口，可以比较大小

String在JDK8及以前用final char\[\]存储，JDK9改为byte\[\]。动机：节约空间 - Most String objects contains only Latin-1 characters. Such characters require only one byte of storage, hence half of the space in the internal char arrays of such String objects is going unused. [https://openjdk.java.net/jeps/254](https://openjdk.java.net/jeps/254) 一个char用2个bytes\(16bits\)存。 改动为将String类从UTF-16 char array改为一个byte array加上一个encoding-flag field

String代表不可变的字符序列 - **String的不可变性**

* 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值
* 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值
* 当调用String的replace\(\)方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值

通过字面量定义的字符串存储在字符串常量池中，字符串常量池中不允许有相同内容的字符串，相同字符串地址相同。

```text
// 通过字面量定义的字符串存储在字符串常量池中，字符串常量池中不允许有相同内容的字符串出现
// 因此s1和s2指向字符串常量池中的同一个字符串，地址相同
String s1 = "abc";
String s2 = "abc";

// 比较s1和s2的地址，结果为true
System.out.println(s1 == s2);
```

 通过new实例化的字符串存储在堆空间，即使字符串相同，地址也不同

```text
// 通过new实例化的字符串存储在堆空间，即使字符串相同，地址也不同
String s3 = new String("test");
String s4 = new String("test");

// 比较s3和s4的地址，结果为false
System.out.println(s3 == s4);
```

 参考Chapter13 - StringTest - test1

字符串常量池\(StringTable\)是一个固定大小的HashTable，默认大小长度是60013\(JDK7\)。如果放进常量池中的String非常多，就会造成Hash冲突严重，从而导致链表很长，影响调用String.intern\(\)的性能。使用 **-XX:StringTableSize** 可设置StringTable的长度

**String.intern\(\) -** 如果字符串常量池中没有对应data的字符串，则在常量池中生成

## String的内存分配

Java6及以前，字符串常量池存放在永久代；Java7以后，字符串常量池调整到Java堆内。调整的原因：

* 永久代空间默认比较小，容易OOM
* 永久代垃圾回收的频率低，

## String的基本操作

Java语言规范要求完全相同的字符串字面量应该包含同样的Unicode字符序列（包含同一份**码点序列**的常量），并且必须指向同一个String类实例，参考Chapter13 - StringTest - test2

```text
public class Memory {
    public static void main(String[] args) {
        int i = 1;
        Object obj = new Object();
        Memory mem = new Memory();
        mem.foo(obj);
    }

    private void foo(Object param) {
        String str = param.toString();
        System.out.println(str);
    }
}
```

 

![](.gitbook/assets/screen-shot-2021-09-25-at-8.49.08-pm.png)

## String的拼接操作

* 常量与常量的拼接，结果在常量池，原理是编译期优化
* 常量池中不会存在相同内容的常量
* 只要其中有一个是变量，结果就在堆中。变量拼接的是通过StringBuilder进行的（JDK5之前是通过StringBuffer进行的） - 参考Chapter13 - StringTest - test5
* final String拼接是常量拼接 - 参考Chapter13 - StringTest - test6 
* 如果拼接的结果调用intern\(\)方法，则判断字符串常量池中是否存在该字符串，如果存在，则返回常量池中的地址，如果不存在在，则将字符串对象放入池中，并返回此对象地址
* 参考Chapter13 - StringTest - test3 & test4 **--面试常考题**

## intern\(\)的使用

如果不是通过字面量\(literal\)定义的String对象，可以使用intern方法，intern\(\)会从字符串常量池中查询当前字符串是否存在，若不存在，则会把当前字符串放入常量池中。也就是说，如果在任意字符串上调用intern\(\)方法，那么其返回结果所指向的那个类实例必须和直接以常量形式出现的字符串实例完全相同。因此下面表达式必定为true:

```text
("a" + "b" + "c").intern() == "abc"
```

**经典的面试题** 参考Chapter13 - StringIntern

```text
public static void main(String[] args) {
    String s1 = new String("1");
    s1.intern();

    String s2 = "1";
    System.out.println(s1 == s2); // jdk6: false; jdk7: false

    String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4); // jdk6: false; jdk7: true
}
```

**JDK 6 - false; false**

![](.gitbook/assets/screen-shot-2021-09-27-at-6.33.50-pm.png)

**JDK 7 : false; true**

![](.gitbook/assets/screen-shot-2021-09-27-at-6.34.35-pm.png)

相关面试题 1：new String\("ab"\)会创建几个对象？ - 2个

相关面试题 2：new String\("a"\) + new String\("b"\)会创建几个对象 - 6个

参考Chapter13 - StringNewTest

### intern\(\)使用总结

* 在JDK6中，将这个字符串对象放入字符串常量池
  * 如果字符串常量池中有，则不会放入，返回已有的对象在字符串常量池中的地址
  * 如果没有，**则会把此对象复制一份**，放入字符串常量池，并返回对象在字符串常量池中的地址
* 在JDK7中，将这个字符串对象放入字符串常量池
  * 如果字符串常量池中有，则不会放入，返回已有的对象在字符串常量池中的地址
  * 如果没有，**则会把此对象的引用地址复制一份**，放入字符串常量池，并返回对象在字符串常量池中的地址
* 参考Chapter13 - StringExercise1 & 2

![](.gitbook/assets/screen-shot-2021-09-27-at-9.02.52-pm.png)

![](.gitbook/assets/screen-shot-2021-09-27-at-9.08.07-pm.png)

![](.gitbook/assets/screen-shot-2021-09-27-at-9.20.22-pm.png)

## StringTable的垃圾回收

```text
public class StringInternGC {

    static final int MAX_COUNT = 1000 * 1000;
    static final String[] arr = new String[MAX_COUNT] ;

    // 在jvisualvm 的Sampler中分析内存空间使用情况：
    // 两者都分别在堆空间和字符串常量池中创建了对象。堆空间中创建MAX_COUNT个对象，常量池中只有11个不重复的字符串。但使用
    // intern()后，arr[]指向的是字符串常量池，因此堆空间中的字符串对象会被GC。不使用intern()时，arr[]指向堆空间
    // 中字符串对象的地址，因此这些字符串对象无法被GC，因而占用空间很大。
    public static void main(String[] args) {

        Integer[] data = new Integer[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        long start = System.currentTimeMillis();

        for (int i = 0; i < MAX_COUNT; i++) {
            // 在jvisualvm 的Sampler中:
            // java.lang.String    24322032(bytes)    24,322,032 (36.5%) 1,013,418 (47.4%) (instances)
            // Time elapsed: 87
            //arr[i] = new String(String.valueOf(data[i % data.length]));

            // 在jvisualvm 的Sampler中:
            // java.lang.String[]  4029600(bytes) 4,029,600 (18.1%)  892 (1.1%) (instances)
            // Time elapsed: 206

            arr[i] = new String(String.valueOf(data[i % data.length])).intern();
        }

        long end = System.currentTimeMillis();
        System.out.println("Time elapsed: " + (end - start));

        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## G1中的String去重操作

G1的垃圾回收器实现了自动持续对重复的字符串去重的操作，避免浪费内存。去重操作指的是对堆空间中相同的字符串**\(string1.equals\(string2\)\)**去重，而不是对字符串常量池去重（字符串常量池中没有重复的字符串）

相关参数

* **UseStringDeduplication -** 开启关闭去重，默认关闭
* **PrintStringDeduplicationStatistics** - 打印详细的去重统计信息
* **StringDeduplicationAgeThreshold** - 达到这个年龄的String对象被认为是去重的候选对象

