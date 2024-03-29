# 堆

## 堆的核心概念

### 概述

一个进程对应一个JVM实例，一个JVM实例对应一个运行时数据区，对应一个方法区和堆。一个进程中的所有线程共享方法区和堆空间。每个线程独立拥有一套程序计数器，本地方法栈和虚拟机栈。

Java堆区在JVM实例启动的时候被创建，其空间大小也就确定了。堆是JVM管理的最大一块内存空间。**堆内存的大小可以调节**。参考Chapter08 - HeapDemo and HeapDemo2

堆可以处于**物理上不连续**的内存空间中，但在**逻辑上应该视为是连续的**。

所有线程共享堆，但堆中可以划分线程私有的缓冲区\(Thread Local Allocation Buffer, **TLAB**\)

**几乎所有的**对象实例以及数组都在运行时分配在堆上，例外：逃逸分析，栈上分配。数组和对象可能永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置。

在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除。堆是GC执行垃圾回收的重点区域。

![](.gitbook/assets/screen-shot-2021-07-24-at-5.31.30-pm.png)

### 堆的细分内存结构

* 现代垃圾收集器大部分都是基于**分代收集理论**设计
* Java7及以前堆内存逻辑上分为：新生区+养老区+**永久区**
* Java8及之后堆内存逻辑上分为：新生区+养老区+**元空间**
* 新生区又被分为**Eden区**和**Survivor区**
* 命名：
  * 新生区 = 新生代 = 年轻代 = Young/New Generation Space
  * 养老区 = 老年区 = 老年代 = Old/Tenure Generation Space 
  * 永久区 = 永久代 = Permanent Space
  * 元空间 = Meta Space

![](.gitbook/assets/screen-shot-2021-07-24-at-7.53.18-pm.png)

![](.gitbook/assets/screen-shot-2021-07-24-at-7.54.44-pm.png)

## 设置堆内存大小与OOM

### 设置堆内存大小

* -Xms = -XX:InitialHeapSize - 用于设置堆区起始内存
* -Xmx = -XX:MaxHeapSize - 用于设置堆区最大内存
* 这里的堆空间**不包括永久代或元空间，只包括年轻代+老年代**
* 一旦堆区中的内存大小超过堆区最大内存，将会抛出OutOfMemoryError
* 通常会将 -Xms和-Xmx两个参数配置相同的值，目的是为了能够在Java垃圾回收机制清理完堆区后不需要重新分隔计算堆区大小，提高性能
* 默认情况下，初始内存 = 物理内存/64  最大内存 = 物理内存/4
* 查看设置的参数 
  * 方式1 - jps / jstat -gc ,进程id&gt;
  * 方式2 - -XX:+PrintGCDetails
* 参考Chapter08 - HeapSpaceInitial

### OOM

堆区内存大小超过堆区最大内存时，会抛出OutOfMemoryError，是Error，不是Exception。但广义上也可以认为是Exception

**Error vs Exception**

* 都继承于Throwable类。
* Exception是可以预见的意外情况，可以被捕获和处理
* Error是比较严重的错误，例如系统崩溃、虚拟机错误、内存不足、栈溢出。无法捕获和处理，一般只能让程序终止
* 面试中问到常见异常时，两者都可以广义地被认为是异常

## 年轻代与老年代

* 存储在JVM中的Java对象可以被分为两类
  * 一类对象生命周期较短的瞬间对象，这类对象的创建和消亡都非常迅速
  * 另一类对象生命周期非常长，极端情况下能够与JVM周期一致
* 堆区空间细分可分为年轻代\(YoungGen\)和老年代\(OldGen\)。年轻代又可细分为Eden空间，Survivor0空间和Survivor1空间

![](.gitbook/assets/screen-shot-2021-07-29-at-12.18.47-am.png)

* 几乎所有的Java对象都是在Eden空间中被new出来
* 绝大部分的Java对象的销毁都在新生代进行

### 相关参数

**-XX:NewRatio -** 配置年轻代与老年代在堆结构的占比，默认值为2

![](.gitbook/assets/screen-shot-2021-07-29-at-12.20.23-am.png)

默认 -XX:NewRatio=2 表示新生代占1， 老年代占2， 新生代占整个堆的1/3

```text
neverland@neverlands-mbp ~ % jps   # JVM Process status                              
4385 Launcher
4386 EdenSurvivor
4035 
4523 Main
4558 Jps
neverland@neverlands-mbp ~ % jinfo -flag SurvivorRatio 4386
-XX:SurvivorRatio=8
neverland@neverlands-mbp ~ % jinfo -flag NewRatio 4386 #check NewRation
-XX:NewRatio=2
```

参考Chapter08 - EdenSurvivor

**-XX:SurvivorRatio -** 配置年轻代中，Eden空间和另外两个Survivor空间的比例，默认值是8:1:1 但在jvisualvm中观察实际的比例是6:1:1，因为有自适应分配策略。-XX:-UseAdaptiveSizePolicy \(不使用自适应分配策略\(-表示不使用，+表示使用\)\)

**-Xmn** 设置新生代最大内存大小，当与-XX:NewRatio冲突时，以-Xmn为准

## 图解对象分配过程

### 步骤概述

* new的对象先放在伊甸园区\(Eden\)，此区有大小限制
* 当伊甸园区空间填满，程序又需要创建对象时，JVM的垃圾回收器将对伊甸园区进行垃圾回收**\(Minor GC，又叫YGC\(Young GC\)\)**，将伊甸园区中的不再被其它对象引用的对象进行销毁。再加载新的对象放到伊甸园区
* **当伊甸园区满时，会触发Minor GC，Minor GC会将伊甸园区和幸存者区一起垃圾回收。但幸存者区满时，不会触发Minor GC，有的对象可以直接晋升到老年区  \*面试考点**
* 然后将伊甸园区中的剩余对象移动到幸存者0区\(Survivor 0\)
* **特殊情况：**Minor GC之后伊甸园区应该为空（原来的对象要么被销毁，要么被放到幸存者区），如果伊甸园区空间仍然不够，说明新创建的对象是个超大对象，对象本身比伊甸园区空间还大，这时会尝试直接放到养老区。如果养老区空间不够，会触发Major GC，之后如果空间依然不够，则产生OOM异常
* 如果再次触发垃圾回收，此时上次幸存下来的在幸存者0区的对象如果没有被回收，就会被放到幸存者1区\(Survivor 1\)
* 如果再次经历垃圾回收，此时会重新放回0区，接着再去1区
* Survivor 0\(S0\)和Survivor 1\(S1\)又被称为From空间和To空间，S0, S1哪个空哪个为To空间。S0,S1必然有一个是空的
* 默认15次之后（每个对象有一个年龄计数器），会被放到养老区\(可以设置**-XX:MaxTenuringThreshold**更改默认次数\)
* 当养老区内存不足时，会触发Major GC，进行养老区内存清理
* 如果养老区在Major GC之后依然内存不足，就会产生OOM异常`java.lang.OutOfMemoryError: Java heap space`
* 垃圾回收：**频繁在新生代收集，很少在养老区收集，几乎不在永久代/元空间收集**
* 参考Chapter08 - HeapInstance

### **图解**

![&#xFF08;&#x7EA2;&#x8272;&#x4EE3;&#x8868;&#x8981;&#x9500;&#x6BC1;&#x7684;&#x5783;&#x573E;&#x5BF9;&#x8C61;&#xFF0C;&#x7EFF;&#x8272;&#x4EE3;&#x8868;&#x8981;&#x4FDD;&#x7559;&#x7684;&#x5BF9;&#x8C61;&#xFF0C;&#x6570;&#x5B57;&#x8868;&#x793A;&#x5BF9;&#x8C61;&#x5E74;&#x9F84;&#xFF09;](.gitbook/assets/screen-shot-2021-07-30-at-12.10.03-am.png)

![](.gitbook/assets/screen-shot-2021-07-30-at-12.12.22-am.png)

![](.gitbook/assets/screen-shot-2021-07-30-at-12.14.57-am.png)

![](.gitbook/assets/screen-shot-2021-07-30-at-12.16.17-am.png)

## Minor GC, Major GC, Full GC

### 概述

* HotSpot JVM的GC按照回收区域分为两大种类型：**部分回收\(Partial GC\)和整堆回收\(Full GC\)**
* 部分回收\(Partial GC\)：不是完整回收整个Java堆的GC
  * **新生代回收\(Minor GC/Yound GC\)**
  * **老年代回收\(Major GC/Old GC\)**
  * 目前，只有CMS GC会有单独回收老年代的行为
  * 很多时候Major GC和Full GC混淆使用，需要具体分辨是老年代还是整堆回收
  * **混合回收\(Mixed GC\)**：回收整个新生代以及部分老年代的GC，目前只有G1 GC会有这个行为
* 整堆回收\(Full GC\)：收集整个Java堆和方法区的GC

### **新生代回收\(Minor GC\)的触发机制**

* 当新生代空间不足时，会触发Minor GC。这里是指，Eden空间满会触发，Survivor满不会触发（只会晋升到老年代）
* 因为Java对象大多数具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。
* Minor GC会引发STW\(Stop-The-World \)，暂停其它的用户线程，等垃圾回收结束，用户线程才恢复运行。但因为速度比较快，对性能影响较小。

### 老年代回收\(Major GC\)的触发机制

* Major GC发生在老年代，对象会从老年代消失
* 当老年代空间不足时，会尝试触发Minor GC，如果之后空间依然不足，则触发Major GC（一般来说，出现了Major GC会伴随至少一次的Minor GC，但并不绝对）
* Major GC的速度一般比Minor GC慢10倍以上，STW的时间更长
* 如果Major GC后，内存依然不足，则会报OOM

### 整堆回收\(Full GC\)的触发机制

* 调用System.gc\(\)时，系统建议执行Full GC，但不是必然执行
* 老年代空间不足
* 方法区空间不足
* 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
* 由Eden区，s0区向s1区复制时，对象大于To Space可用内存，则把对象转存到老年代，且老年代可用内存小于该对象大小时
* **Full GC是开发和调优中要尽量避免的**

参考Chapter08 - GCTest

## 堆空间分代思想

**为什么要把Java堆分代？**

经研究，不同对象的生命周期不同，70%-99%的对象是临时对象，可以进行内存清理\(GC\)。**分代是为了优化GC性能**。如果没有分代，那么所有的对象都在一起，GC时就需要对堆的所有区域进行扫描。

## 内存分配策略（小结）

如果对象在Eden出生并经过一次MinorGC后仍然存活，并且能被Survivor容纳，将被移到Survivor空间中，并将对象年龄设为1岁。对象在Survivor没经过一次GC存活，年龄增加1岁。当年龄增加到一定程度（默认15岁）时，就会被晋升到老年代。

**内存分配策略**

* 优先分配到Eden
* 大对象直接分配到老年代 - 尽量避免程序中出现过多的大对象
* 长期存活的对象分配到老年代
* 动态对象年龄判断 - 如果Survivor区中相同年龄的所有对象大小总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无需等到MaxTenuringThreshold中要求的年龄（默认15）
* 空间分配担保 - -XX:HandlePromotionFailure 当大量对象在GC之后仍然存活，把Survivor中容纳不下的对象放到老年代。

## 为对象分配内存 - TLAB\(Thread Local Allocation Buffer\)

### 为什么有TLAB

* 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
* 由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
* 为避免多个线程操作同一个地址，需要使用加锁等机制，因而影响分配速度

### 什么是TLAB

* 从内存模型而不是垃圾回收的角度，对Eden区继续进行划分，**JVM为每个线程分配了一个私有缓存区域**，包含在Eden空间内
* 多线程同时分配内存时，使用TLAB可以避免一系列的线程安全问题，同时还能提升内存分配的吞吐量，因此我们可以**将这种内存分配方式称为快速分配策略**

![](.gitbook/assets/screen-shot-2021-08-06-at-12.30.34-am.png)

* 尽管不是所有的对象实例都能在TLAB中成功分配内存（因为TLAB空间有限），但JVM将TLAB作为内存分配的首选
* 可以通过 **-XX:UseTLAB** 设置是否开启TLAB空间，默认开启
* 默认情况下，TLAB空间非常小，**仅占有Eden空间的1%**。可以通过 **-XX:TLABWasteTargetPercent** 设置TLAB空间占用Eden空间的百分比
* 一旦对象在TLAB空间分配内存失败，JVM就会尝试**通过加锁机制**在Eden空间中直接分配内存，从而确保数据操作的原子性
* 参考Chapter08 - TLABArgs

![](.gitbook/assets/screen-shot-2021-08-06-at-12.42.45-am.png)

## 小结堆空间的常用参数设置

官网文档：[https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)

-XX:+PrintFlagsInitial - 查看**所有参数**的默认初始值

-XX:+PrintFlagsFinal - 查看**所有参数**的最终值（可能会被修改为不是初始值）

* 查看**具体某个参数**的指令
  * jps 查看当前运行中的进程，获取进程id
  * jinfo -flag SurvivorRatio &lt;进程id&gt; 

-Xms - 初始堆空间内存（默认为物理内存的1/64）

-Xmx - 最大堆空间内存（默认为物理内存的1/4）

-Xmn - 设置新生代的大小（初始值与最大值）

-XX:NewRatio - 配置新生代与老年代在堆结构的占比（默认值为2，表示新生代占1， 老年代占2， 新生代占整个堆的1/3）

-XX:SurvivorRatio - 配置新生代中Eden和S0/S1空间的比例（默认值为8）

-XX:MaxTenuringThreshold - 配置新生代垃圾的最大年龄

-XX:+PrintGCDetails - 打印详细的GC处理日志

-XX:+PrintGC \(or -verbose:gc\) - 打印简要的GC日志

-XX:HandlePromotionFailure - 是否设置空间分配担保

## 堆是分配对象存储的唯一选择吗

普遍情况下，JVM中的对象是在堆中分配内存的。但有一种特殊情况，即如果经过**逃逸分析\(Escape Analysis\)**后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成**栈上分配**。这样就无需再堆上分配内存，也就无需进行GC。

另一种情况是在淘宝虚拟机\(TaoBaoVM\)中的GCIH\(GC invisible heap\)技术，将生命周期较长的对象从heap中移至heap外，并且GC不能管理，从而降低GC的频率，提升回收效率。

### 逃逸分析

* 逃逸分析是一种可以有效减少Java程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法
* 通过逃逸分析，Java编译器能够分析出一个新的对象引用的范围从而决定是否要将这个对象分配到堆上
* 逃逸分析的基本行为就是分析对象动态作用域
  * 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸，可以分配到栈上。随着方法执行结束，栈空间就被移除
  * 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸，例如作为调用参数传递到其它地方中
* 判断是否发生了逃逸，看new的对象实体是否可能在方法外被调用 - 参考Chapter08 - EscapeAnalysis
* 在JDK7以后，逃逸分析默认开启
  * -XX:+DoEscapeAnalysis - 设置开启逃逸分析
  * -XX:+PrintEscapeAnalysis - 查看逃逸分析的筛选结果
* **开发中，能使用局部变量的就不要在方法外定义。**

### 代码优化

使用逃逸分析，编译器可以对代码进行如下优化

* **栈上分配** 
  * 将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配
  * JIT编译器在编译期间根据逃逸分析的结果，如果发现一个对象没有逃逸，就可能将这个对象优化为栈上分配。分配完成后，继续在调用栈内执行，最后线程结束，栈空间被回收，无需GC
  * 常见的**逃逸**场景 - 给成员变量赋值，方法返回值，实例引用传递（Chapter08 - EscapeAnalysis中的例子）
  * 参考Chapter08 - StackAllocation
* **同步省略** 
  * 如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步
  * 线程同步的代价很高，降低并发性和运行性能
  * 在动态编译同步块的时候，JIT编译器可以借助逃逸分析来判断同步块所使用的的锁对象是否只能被一个线程访问，而没有发布到其它线程。如果没有，那么JIT编译器在编译这个同步块时会取消这部分代码的同步，这样可以大大提高并发性和运行性能。这个取消同步的过程叫做**同步省略**，也叫**锁消除**
  * 下面的代码会被JIT在编译阶段自动优化

```text
public void f() {
    Object hollis = new Object();
    synchronized(hollis) {
        System.out.println(hollis);
    }
}
```

```text
public void f() {
    Object hollis = new Object();
    System.out.println(hollis);
}
```

* **分离对象或标量替换** 
  * 有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么该对象的部分（或全部）可以不存储在内存里，而是存在CPU的寄存器（对于Java来说，存在栈空间）中
  * **标量\(Scalar\)**是指一个无法再分解成更小数据的数据。Java中的原始数据类型就是标量。相对应的，可以分解的数据叫做**聚合量\(Aggreate\)**。Java中的对象就是聚合量。
  * 在JIT编译阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含若干个成员的变量来代替。这个过程叫做**标量替换**。
  * 下面的代码会被JIT在编译阶段自动优化。alloc\(\)中的Point这个聚合量被替换为两个标量，从而不需要分配堆内存。
  * -XX:+EliminateAllocations - 设置开启标量替换（默认开启）

```text
public static void main(String[] args) {
    alloc();
}

private static void alloc() {
    Point point = new Point();
    System.out.println("point");
}

static class Point {
    private int x;
    private int y;
}
```

```text
private static void alloc() {
    int x = 1;
    int y = 2;
    System.out.println("point");
}
```

