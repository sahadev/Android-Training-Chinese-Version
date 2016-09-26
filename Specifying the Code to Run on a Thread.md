原文地址：[http://android.xsoftlab.net/training/multiple-threads/index.html](http://android.xsoftlab.net/training/multiple-threads/index.html)

#引言
大量的数据处理往往需要花费很长的时间，但如果将这些工作切分并行处理，那么它的速度与效率就会提升很多。在多线程处理器的设备中，系统可以并行运行线程。比如，使用多线程将多个图像文件解码展示要比单一线程解码快得多。

这节课我们会学习如何设置并使用多线程及线程池。我们还会学习如何在一个线程中运行代码以及如何使该线程与UI线程进行通信。

#在线程中运行代码
这节课我们会学习如何实现[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)接口，该接口中的[run()](http://android.xsoftlab.net/reference/java/lang/Runnable.html#run())方法会在单独的线程中运行。