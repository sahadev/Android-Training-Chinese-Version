原文地址：[http://android.xsoftlab.net/training/articles/perf-anr.html#anr](http://android.xsoftlab.net/training/articles/perf-anr.html#anr)



##什么触发了ANR?
通常情况下，系统会在应用程序不再能够响应用户的输入时显示ANR对话框。比如，如果应用被阻塞在了UI线程的IO操作上，那么系统就不能够处理用户的输入事件。或者应用花费了大量的时间来处理内存模型的构建或者是在UI线程中计算了游戏的下一步动作。
即便是最高效的代码也需要花费时间来运行。

在任何情况中都
##如何避免ANR?

##ANR相关优化
