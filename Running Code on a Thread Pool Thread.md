原文地址：[http://android.xsoftlab.net/training/multiple-threads/run-code.html#StopThread](http://android.xsoftlab.net/training/multiple-threads/run-code.html#StopThread)

上节课我们学习了如何定义一个类用于管理线程以及任务。这节课将会学习如何在线程池中运行任务。要做到这一点，只需要往线程池的工作队列中添加任务即可。当一条线程处于闲置状态时，那么ThreadPoolExecutor会从任务队列中取出一条任务并放入该线程中运行。

这节课还介绍了如何停止一个正在运行中的任务。