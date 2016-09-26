原文地址：[http://android.xsoftlab.net/training/notify-user/display-progress.html#FixedProgress](http://android.xsoftlab.net/training/notify-user/display-progress.html#FixedProgress)

通知中包含了一个进度指示器，用来向用户展示一项正在进行中的工作状态。如果你可以确保任务会花费多长时间，并且可以在任何时候得知它完成了多少工作，那么就可以使用确定样式的指示器(一个进度条)。如果不能确定任务需要花费的时间，可以使用不确定样式的指示器(一个活动的指示器)。

进度指示器由[ProgressBar](http://android.xsoftlab.net/reference/android/widget/ProgressBar.html)类实现。

使用进度指示器，可以调用[setProgress()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean))方法。确定样式与不确定样式会在下面的章节中讨论。

##显示确定进度指示器
为了显示确定进度指示器，需要调用[setProgress(max, progress, false)](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean))方法将指示器添加到通知上，然后再将该通知发布出去。该方法的第三个参数用于指示该进度条是确定性进度条(true)还是不确定性进度条(false)。随着操作的处理，进度progress会增长，这时需要更新通知。在操作结束时，progress应该等于max。一种常规的方式是将max设置为100,然后将progress以百分比的形式自增。

你也可以选择在任务完成的时候将进度条取消显示或者移除通知。在前一种情况中，要记得更新通知的文本，告诉用户任务已完成。后一种情况中，调用[setProgress(0, 0, false)](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean))就可以完成通知的移除。
```java
int id = 1;
...
mNotifyManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
mBuilder = new NotificationCompat.Builder(this);
mBuilder.setContentTitle("Picture Download")
    .setContentText("Download in progress")
    .setSmallIcon(R.drawable.ic_notification);
// Start a lengthy operation in a background thread
new Thread(
    new Runnable() {
        @Override
        public void run() {
            int incr;
            // Do the "lengthy" operation 20 times
            for (incr = 0; incr <= 100; incr+=5) {
                    // Sets the progress indicator to a max value, the
                    // current completion percentage, and "determinate"
                    // state
                    mBuilder.setProgress(100, incr, false);
                    // Displays the progress bar for the first time.
                    mNotifyManager.notify(id, mBuilder.build());
                        // Sleeps the thread, simulating an operation
                        // that takes time
                        try {
                            // Sleep for 5 seconds
                            Thread.sleep(5*1000);
                        } catch (InterruptedException e) {
                            Log.d(TAG, "sleep failure");
                        }
            }
            // When the loop is finished, updates the notification
            mBuilder.setContentText("Download complete")
            // Removes the progress bar
                    .setProgress(0,0,false);
            mNotifyManager.notify(id, mBuilder.build());
        }
    }
// Starts the thread by calling the run() method in its Runnable
).start();
```

最终的效果如下图所示：
![](http://android.xsoftlab.net/images/ui/notifications/progress_bar_summary.png)
左边的图显示了正在进行中的通知，而右边的图显示了任务完成后的通知。

##显示持续活动的指示器
为了显示不确定性的指示器，需要调用[setProgress(0, 0, true)](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean))方法将进度条显示在通知中，然后将该通知发布。第一第二个参数将会被忽略，第三个参数决定了该进度条是否是不确定性进度条。最终的显示效果为与常规进度条有相同的显示风格，除了它一直在动之外。

在操作开始之前请发布该通知，进度动画会一直持续运行，直到你修改了通知。当操作完成后，调用[setProgress(0, 0, false)](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean))方法然后更新通知以便移除活动指示器。否则的话，就算是任务完成后，该动画也不会停止。所以要记得在任务完成后更改通知文本，以便告知用户操作已完成。

```java
// Sets the progress indicator to a max value, the current completion
// percentage, and "determinate" state
mBuilder.setProgress(100, incr, false);
// Issues the notification
mNotifyManager.notify(id, mBuilder.build());
```

找到前面的代码，将下面部分替换。要记得[setProgress()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean))方法的第三个参数为true：
```java
 // Sets an activity indicator for an operation of indeterminate length
mBuilder.setProgress(0, 0, true);
// Issues the notification
mNotifyManager.notify(id, mBuilder.build());
```

最终的显示效果如下：

![](http://android.xsoftlab.net/images/ui/notifications/activity_indicator.png)