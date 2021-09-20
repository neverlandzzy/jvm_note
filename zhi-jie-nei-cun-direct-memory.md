# 直接内存\(Direct Memory\)

## 直接内存概述

* 直接内存不是虚拟机运行时数据区的一部分，也不是JVM规范中定义的内存区域
* 直接内存是在Java堆外的、直接向系统申请的内存空间
* 来源于NIO，通过存在堆中的DirectByteBuffer操作Native内存
* 通常，访问直接内存的速度会优于Java堆，即读写性能高。
  * 因此出于性能考虑，读写频繁的场合可能会考虑使用直接内存
  * Java的NIO库允许Java程序使用直接内存，用于数据缓冲区

**&lt;相关知识&gt; IO与NIO** 参考Chapter10 - BufferTest

IO -

* 阻塞式IO
* 基于流\(stream\)
* 使用byte\[\]/char\[\]传输，对应字节流/字符流

NIO -

* 非阻塞式IO
* 基于通道\(channel\)
* 使用缓冲区\(Buffer\)

