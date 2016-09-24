原文地址：[http://android.xsoftlab.net/training/multiple-threads/define-runnable.html](http://android.xsoftlab.net/training/multiple-threads/define-runnable.html)

这节课主要学习如何实现[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)类。你也可以将[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)对象传递给一个[Thread](http://android.xsoftlab.net/reference/java/lang/Thread.html)类。这种执行特定任务的[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)在某些时候被称为*任务*。

[Thread](http://android.xsoftlab.net/reference/java/lang/Thread.html)类与[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)类属于基础类，它们仅提供了有限的功能。它们是比如[HandlerThread](http://android.xsoftlab.net/reference/android/os/HandlerThread.html), [AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html), 以及[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)等功能的基础核心。这两个类同样属于[ThreadPoolExecutor](http://android.xsoftlab.net/reference/java/util/concurrent/ThreadPoolExecutor.html)的核心基础。这个类会自动的管理线程以及任务队列，它甚至还可以使多个线程同时执行。

##定义Runnable实现类
实现一个[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)对象很简单：
```java
public class PhotoDecodeRunnable implements Runnable {
    ...
    @Override
    public void run() {
        /*
         * Code you want to run on the thread goes here
         */
        ...
    }
    ...
}
```

##实现run()方法
在[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)的实现类中，[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)的[run()](http://android.xsoftlab.net/reference/java/lang/Runnable.html#run())方法中所含的代码将会被执行。通常来说，[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)中可以做任何事情。要记得，这里的[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)不会运行在UI线程，所以在它内部不能直接修改[View](http://android.xsoftlab.net/reference/android/view/View.html)对象这种UI对象。为了与UI线程通讯，你需要使用 [Communicate with the UI Thread](http://android.xsoftlab.net/training/multiple-threads/communicate-ui.html) 课程中所描述的技术。

在[run()](http://android.xsoftlab.net/reference/java/lang/Runnable.html#run())方法的开头处设置当前的线程使用后台优先级。这种方式可以减少[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)对象所属线程与UI线程的资源争夺问题。

这里还将[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)对象所属的线程引用存储了起来。由[Thread.currentThread()](http://android.xsoftlab.net/reference/java/lang/Thread.html#currentThread())可以获得当前的线程对象。

下面的代码展示了具体的实现方式：
```java
class PhotoDecodeRunnable implements Runnable {
...
    /*
     * Defines the code to run for this task.
     */
    @Override
    public void run() {
        // Moves the current Thread into the background
        android.os.Process.setThreadPriority(android.os.Process.THREAD_PRIORITY_BACKGROUND);
        ...
        /*
         * Stores the current Thread in the PhotoTask instance,
         * so that the instance
         * can interrupt the Thread.
         */
        mPhotoTask.setImageDecodeThread(Thread.currentThread());
        ...
    }
...
}
```

