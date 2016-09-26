原文地址：[http://developer.android.com/training/displaying-bitmaps/manage-memory.html](http://developer.android.com/training/displaying-bitmaps/manage-memory.html)

除了在上一节中描述的步骤之外，还有一些细节上的事情可以促进垃圾回收器的回收及位图的复用。其推荐的策略取决于Android的目标版本。示例APP BitmapFun展示了如何使应用程序在不同的版本上高效的工作。

为了给这节课的知识奠定一些基础，下面有一些Android系统如何管理位图内存的一些改进需要了解：

- 在Android 2.2之前，当垃圾回收器回收时，应用的线程会被停止。这会降低性能发生延迟。Android 2.3增加了并发收集垃圾的功能，这意味着内存回收不久之后位图不可再被引用。
- 在Android 2.3.3之前，位图对应的支撑数据被存放在本地内存中。这与位图本身是分离的，它被存储在Dalvik堆栈之中。在意料之中位于本地内存中的像素数据是不会被释放的，可能会导致程序超过自身的内存限制然后崩溃。