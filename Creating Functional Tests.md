原文地址：[http://android.xsoftlab.net/training/activity-testing/activity-functional-testing.html](http://android.xsoftlab.net/training/activity-testing/activity-functional-testing.html)

功能性测试包括模拟用户操作之类的应用组件验证。例如，开发者可以通过功能性测试来验证在用户执行UI操作之后Activity是否成功的启动了目标Activity。

为Activity创建功能性测试，测试类应当继承[ActivityInstrumentationTestCase2](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html)。与ActivityUnitTestCase不同，[ActivityInstrumentationTestCase2](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html)可以与Android系统通信，也可以使应用程序接收键盘输入事件与屏幕点击事件。

##验证功能行为
一般功能性测试可能会有以下测试目标：

- 验证在某个UI控制器被按下后，目标Activity是否被启动。
- 验证目标Activity是否将在启动之前的用户输入数据正确的显示。

开发者所实现的代码可能如下：
```java
@MediumTest
public void testSendMessageToReceiverActivity() {
    final Button sendToReceiverButton = (Button) 
            mSenderActivity.findViewById(R.id.send_message_button);
    final EditText senderMessageEditText = (EditText) 
            mSenderActivity.findViewById(R.id.message_input_edit_text);
    // Set up an ActivityMonitor
    ...
    // Send string input value
    ...
    // Validate that ReceiverActivity is started
    ...
    // Validate that ReceiverActivity has the correct data
    ...
    // Remove the ActivityMonitor
    ...
}
```

测试会等待所匹配的Activity启动，否则的话，将会在超时后返回null。如果ReceiverActivity启动，那么[ActivityMonitor](http://android.xsoftlab.net/reference/android/app/Instrumentation.ActivityMonitor.html)则会收到一个命中。开发者可以通过断言方法来验证ReceiverActivity是否确实被启动，并且命中数会如所期望的那样增加。

##设置ActivityMonitor
为了监视Activity，可以注册[ActivityMonitor](http://android.xsoftlab.net/reference/android/app/Instrumentation.ActivityMonitor.html)。当目标Activity启动时，系统会通知[ActivityMonitor](http://android.xsoftlab.net/reference/android/app/Instrumentation.ActivityMonitor.html)一个事件。如果目标Activity启动，那么监视器的计数器会更新。

通常使用[ActivityMonitor](http://android.xsoftlab.net/reference/android/app/Instrumentation.ActivityMonitor.html)应当：

- 1.通过getInstrumentation()方法获得用于测试用例Instrumentation实例。
- 2.通过Instrumentation的addMonitor()重载方法将Instrumentation.ActivityMonitor的实例添加到当前的instrumentation。匹配标准可由IntentFilter或者类名指定。
- 3.等待Activity的启动。
- 4.验证监视器的数字是否增长。
- 5.移除监视器。

例如：
```java
// Set up an ActivityMonitor
ActivityMonitor receiverActivityMonitor =
        getInstrumentation().addMonitor(ReceiverActivity.class.getName(),
        null, false);
// Validate that ReceiverActivity is started
TouchUtils.clickView(this, sendToReceiverButton);
ReceiverActivity receiverActivity = (ReceiverActivity) 
        receiverActivityMonitor.waitForActivityWithTimeout(TIMEOUT_IN_MS);
assertNotNull("ReceiverActivity is null", receiverActivity);
assertEquals("Monitor for ReceiverActivity has not been called",
        1, receiverActivityMonitor.getHits());
assertEquals("Activity is of wrong type",
        ReceiverActivity.class, receiverActivity.getClass());
// Remove the ActivityMonitor
getInstrumentation().removeMonitor(receiverActivityMonitor);
```

##使用Instrumentation发送键盘事件
如果Activity含有EditText，可能需要测试用户可以对其输入数据。

一般来说，要发送字符串值到EditText，应当：

- 1.使用[runOnMainSync()](http://android.xsoftlab.net/reference/android/app/Instrumentation.html#runOnMainSync(java.lang.Runnable))方法运行[requestFocus()](http://android.xsoftlab.net/reference/android/view/View.html#requestFocus())同步调用。这样会使UI线程一直等待接收焦点。
- 2.调用[waitForIdleSync()](http://android.xsoftlab.net/reference/android/app/Instrumentation.html#waitForIdleSync())方法使主线程变为空闲状态。
- 3.通过[sendStringSync()](http://android.xsoftlab.net/reference/android/app/Instrumentation.html#sendStringSync(java.lang.String))方法发送一条字符串给EditText。

例如：
```java
// Send string input value
getInstrumentation().runOnMainSync(new Runnable() {
    @Override
    public void run() {
        senderMessageEditText.requestFocus();
    }
});
getInstrumentation().waitForIdleSync();
getInstrumentation().sendStringSync("Hello Android!");
getInstrumentation().waitForIdleSync();
```

