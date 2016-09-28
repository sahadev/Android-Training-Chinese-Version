原文地址：[http://android.xsoftlab.net/training/articles/perf-anr.html#anr](http://android.xsoftlab.net/training/articles/perf-anr.html#anr)

尽管可能你写代码通过了世界上所有的性能测试，但是它还是可能会让人感觉到卡顿。当应用卡的不成样子时，系统会给你弹一个"Application Not Responding"的对话框。

在Android中，系统会对那些长时间没有响应的应用采取一些措施：弹出一个对话框告诉用户APP已经停止了响应，如下图所示：
http://android.xsoftlab.net/images/anr.png 正出于这个原因，系统会在APP长时间没有响应的时候为用户提供一个退出APP的选项。所以使APP能够及时响应这一点是至关重要的，这样系统才不会向用户显示ANR对话框。

这节课我们会学习Android系统如何检查应用程序是否未响应，以及为应用程序如何保持响应提供了改进指南。

##什么触发了ANR?
通常情况下，系统会在应用程序不再能够响应用户的输入时显示ANR对话框。比如，如果应用被阻塞在了UI线程的IO操作上，那么系统就不能够处理用户的输入事件。或者应用花费了大量的时间来处理内存模型的构建或者是在UI线程中计算了游戏的下一步动作。
即便是最高效的代码也需要花费时间来运行。

在任何情况下都不要在UI线程中执行耗时操作，而是要将这些工作放在一个单独的线程中执行。这可以使UI线程保持运行(UI线程负责驱动用户界面的事件循环)。

在Android中，应用程序的响应态由Activity Manager 及 Window Manager负责监控。系统会在侦测到以下状况时显示ANR对话框：

- 对输入事件在5秒内没有作出响应。
- BroadcastReceiver在10秒内没有执行完毕。

##如何避免ANR?
Android应用程序默认运行在UI线程中。这意味着在UI线程中的任何耗时操作都会引起ANR问题，因为应用程序并没有给输入事件或者意图广播处理的机会。

因此，在UI线程中的每个方法都应当做尽可能少的工作，尤其是Activity的生命周期回调函数。像网络或数据库操作，或者大量的计算之类的耗时操作应当在工作线程中执行。

创建用于执行耗时操作的线程最便捷的方式莫过于使用AsyncTask了。只需要继承AsyncTask，然后重写doInBackground()就可以执行了。如果要向用户展示工作进度，你可以使用publishProgress()方法，它会回调onProgressUpdate()方法(该方法运行于UI线程)。
在onProgressUpdate()内我们可以更新进度条。
```java
private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
    // Do the long-running work in here
    protected Long doInBackground(URL... urls) {
        int count = urls.length;
        long totalSize = 0;
        for (int i = 0; i < count; i++) {
            totalSize += Downloader.downloadFile(urls[i]);
            publishProgress((int) ((i / (float) count) * 100));
            // Escape early if cancel() is called
            if (isCancelled()) break;
        }
        return totalSize;
    }
    // This is called each time you call publishProgress()
    protected void onProgressUpdate(Integer... progress) {
        setProgressPercent(progress[0]);
    }
    // This is called when doInBackground() is finished
    protected void onPostExecute(Long result) {
        showNotification("Downloaded " + result + " bytes");
    }
}
```

使用execute()方法启动工作线程：
```java
new DownloadFilesTask().execute(url1, url2, url3);
```

如果不采用这种方式，我们还有另一种实现方法：创建自己的Thread或HandlerThread。如果采用这种方法，那么应该设置该线程的优先级为"background"：通过Process.setThreadPriority()方法及参数THREAD_PRIORITY_BACKGROUND设置。
如果没有设置该优先级，那么该线程会使应用感到变慢，因为该线程的优先级默认与UI线程的优先级一致，它们会互相抢占CPU资源。

如果实现了自己的Thread或HandlerThread，那么要确保在等待其它工作线程完成之前UI线程不被阻塞---不要调用Thread.wait()或Thread.sleep()。如果需要等待其它线程的执行结果，可以为UI线程创建一个Handler。这样做可以使UI线程还可以对
输入事件保持响应能力。这样就可以避免5秒内无响应的ANR对话框出现。

BroadcastReceiver在执行时间上有特殊的限制，这意味着在其内部的工作一定是轻量级的：比如在后台做一些保存设置或者发送通知的工作。所以与UI线程中执行的方法一样，广播接收器内也应当杜绝耗时操作的出现。

> **TIP:** 你可以使用StrictMode来发现UI线程中意外出现的耗时操作。

##ANR相关优化
一般来说，100~200毫秒是用户所能感知到应用卡顿的极限。下面列出了一些可以避免应用程序ANR的一些点，同样也有助于防止出现卡顿的情况：

- 如果应用需要对用户输入做大量的后台工作，可以显示一个进度表示工作正在进行。
- 对于游戏类的复杂计算，应该将这些工作放在工作线程中执行。
- 如果应用在初始化阶段需要花费一些时间，可以考虑显示一个闪屏页面或者尽可能快的显示主界面：展示加载正在进行，并进行异步数据填充。在这些情况下都应当表明任务正在进行，以免让用户认为应用已经卡死。
- 使用Systrace或Traceview等性能工具检查APP的响应瓶颈。

