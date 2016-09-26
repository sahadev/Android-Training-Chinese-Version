原文地址：[http://android.xsoftlab.net/training/run-background-service/send-request.html](http://android.xsoftlab.net/training/run-background-service/send-request.html)

上节课我们学习了如何创建IntentService类。这节课我们主要学习如何通过Intent使IntentService运行工作请求。Intent可以携带任意数据交给IntentService处理。你可以在Activity或者Fragment的任意方法内发送Intent给IntentService。

##创建并发送工作请求到IntentService
为了创建一个工作请求并将其发送给IntentService，首先我们需要创建一个显示的Intent对象，然后向其添加请求数据，最后在通过[startService()](http://android.xsoftlab.net/reference/android/content/Context.html#startService(android.content.Intent))将它发送到IntentService。

下面的代码演示了这个过程：

 - 1. 为名RSSPullService的IntentService创建一个显示的Intent。

```java
/*
 * Creates a new Intent to start the RSSPullService
 * IntentService. Passes a URI in the
 * Intent's "data" field.
 */
mServiceIntent = new Intent(getActivity(), RSSPullService.class);
mServiceIntent.setData(Uri.parse(dataUrl));
```

 - 2. 调用[startService()](http://android.xsoftlab.net/reference/android/content/Context.html#startService(android.content.Intent))。

```java
// Starts the IntentService
getActivity().startService(mServiceIntent);
```

注意，你可以在Activity或者Fragment的任何地方发送工作请求。举个例子，如果你首先需要获得用户的输入，那么你就可以将工作请求的发送代码放在Button按钮的点击回调内。

一旦调用了[startService()](http://android.xsoftlab.net/reference/android/content/Context.html#startService(android.content.Intent))，那么[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)将会在[onHandleIntent()](http://android.xsoftlab.net/reference/android/app/IntentService.html#onHandleIntent(android.content.Intent))方法内执行工作请求，并且它会任务完成后自动终止。

下一个步骤就是将工作的完成结果反馈给请求调用处。下一节课将会学习如何使用[BroadcastReceiver](http://android.xsoftlab.net/reference/android/content/BroadcastReceiver.html)完成这个过程。