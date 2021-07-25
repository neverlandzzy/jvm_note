# 本地方法接口

## 本地方法\(Native Method\)

一个本地方法就是一个Java调用非Java代码的接口，本地方法由非Java语言实现，例如C。本地方法接口的作用是融合不同的编程语言为Java所用，使用native修饰。native可以与其它所有标识符连用，但abstract除外。abstract要求方法没有方法体，但native方法有方法体，只是不在Java code中。

```text
public class Natives {
    public native void Native1(int x);
    native static public long Native2();
    native private synchronized float Native3(Object o);
    native void Native4(int[] array) throws Exception
}
```

## 为什么要使用本地方法

有些层次的任务用Java实现起来不容易或者效率低。

* 与Java环境外交互 - 有时Java应用需要与外面的环境交互，如其它语言。
* 与操作系统交互 - 通过本地方法，Java实现了jre与操作系统的交互，JVM的一些部分就是用C写的
* Sun的解释器是用C实现的，这使得它能像一些普通的C一样与外部交互。
* 目前本地方法使用越来越少，除非是与硬件有关的应用

