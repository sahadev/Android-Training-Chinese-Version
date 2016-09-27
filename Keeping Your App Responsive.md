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

##ANR相关优化
