原文地址:[http://android.xsoftlab.net/training/run-background-service/index.html](http://android.xsoftlab.net/training/run-background-service/index.html)

#引言
除非特别指定，否则所有的操作都是在UI线程中执行的。不过这会引起问题，因为长时间的耗时操作会妨碍UI线程的运行。这会惹恼用户，并可能会引起系统错误。为了避免这样的情况出现，Android为此提供了一些类，可以使这些耗时操作放在单独的线程中执行。这里用到最多的就是[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)了。

这节课主要学习如何实现[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)，以及如何向它发送工作请求，以及如何响应它的执行结果。

#创建后台服务
[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)提供了一个非常简单的构造方法。[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)允许执行耗时操作，而又不会引起UI线程的阻塞。同样的，[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)还不受UI生命周期的影响。所以它可以在一个单独的环境中持续运行。

不过[IntentService]([IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html))也是有限制的，列举如下:

- 它不可以与UI线程直接交互。为了将结果递送到UI，不得不采用广播的形式将结果发送出去。
- 工作请求是按顺序执行的。如果目前已经有一个操作在IntentService中执行，那么再往其中发送操作请求的话，这些操作请求都将会等待，直至第一个操作完成。
- IntentService中的操作不可以被中断。

不管怎样，大多数情况下，IntentService是执行后台操作的首选方式。

这节课主要学习如何实现IntentService，以及如何创建请求回调方法[onHandleIntent()](http://android.xsoftlab.net/reference/android/app/IntentService.html#onHandleIntent(android.content.Intent))，最后我们还会学习如何在清单文件中声明该IntentService。

##创建IntentService
首先需要创建一个类并继承IntentService，然后重写它的onHandleIntent()方法:
```java
public class RSSPullService extends IntentService {
    @Override
    protected void onHandleIntent(Intent workIntent) {
        // Gets data from the incoming Intent
        String dataString = workIntent.getDataString();
        ...
        // Do work here, based on the contents of dataString
        ...
    }
}
```

这里要注意普通[Service](http://android.xsoftlab.net/reference/android/app/Service.html)的那些回调方法，比如[onStartCommand()](http://android.xsoftlab.net/reference/android/app/Service.html#onStartCommand(android.content.Intent, int, int))方法，它会被IntentService自动调用，所以在IntentService内部最好不要重写这些方法。

##在清单文件中声明IntentService
IntentService同样需要在清单文件中定义。它的定义方式与普通Service是一样的。
```xml
    <application
        android:icon="@drawable/icon"
        android:label="@string/app_name">
        ...
        <!--
            Because android:exported is set to "false",
            the service is only available to this app.
        -->
        <service
            android:name=".RSSPullService"
            android:exported="false"/>
        ...
    <application/>
```

其中android:name属性说明了IntentService的类名。

这里要注意：Service标签内并没有包含Intent Filter。因为其它组件是通过显示Intent发送工作请求的。所以这里不需要定义Intent Filter。这也意味着只有相关组件内的APP或者拥有相同用户ID的程序才可以访问该服务。

现在我们已经定义好了一个IntentService类，接下来就可以通过[Intent](http://android.xsoftlab.net/reference/android/content/Intent.html)对象向其发送工作请求了。关于如何构建相关对象以及如何发送请求到IntentService的相关内容将会在下节课学习。