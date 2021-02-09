# JVM与JAVA体系结构

JVM -&gt; Java API -&gt; 框架\(e.g. Spring\) -&gt; 程序

![](.gitbook/assets/screen-shot-2021-02-04-at-3.19.49-pm.png)

JVM 是一个跨语言的平台。各语言用各自的编译器编译出**字节码文件\(class file\)**遵循JVM的规范，可以在JVM上解释运行。JVM不关心运行在其内部的程序是何种语言编写的，**只关心字节码文件**。

![](.gitbook/assets/screen-shot-2021-02-06-at-10.47.20-pm.png)

**虚拟机分类**

* **系统虚拟机** - 提供一个可运行完整操作系统的软件平台，是对物理计算机的仿真，例如Visual Box, VMware
* **程序虚拟机** - 专门为执行单个计算机程序而设计的虚拟机，例如Java虚拟机

**JVM的位置**

![](.gitbook/assets/screen-shot-2021-02-06-at-11.43.41-pm.png)

**JDK的构成**

![](.gitbook/assets/screen-shot-2021-02-06-at-11.46.03-pm.png)

**JVM 的整体结构**

* Class loader将字节码文件\(class files\)加载到内存当中，生成一个class对象。
* 在内存中， 多线程共享方法区\(method area\)和堆\(heap\)
* Java栈，本地方法栈和程序计数器为线程私有
* 执行引擎\(Execution Engine\)包括解释器\(Interpreter\)，JIT即时编译器\(JIT Compiler\)和垃圾回收器\(Garbage Collection\)

![](.gitbook/assets/screen-shot-2021-02-08-at-11.00.38-pm.png)

**Java代码执行流程**

![](.gitbook/assets/screen-shot-2021-02-08-at-11.18.29-pm.png)

