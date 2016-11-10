原文地址：[http://android.xsoftlab.net/training/activity-testing/activity-unit-testing.html](http://android.xsoftlab.net/training/activity-testing/activity-unit-testing.html)


Activity单元测试除了可以快速的验证Activity的状态之外，还可以验证Activity与底层组件的交互。单元测试通常用于测试最小的代码单元(它们通常不依赖系统或者网络资源)，它们可能是一个方法，一个类或者其它组件。例如，开发者可以通过单元测试来检查Activity是否含有正确的布局，或者是否触发了正确的Intent。

单元测试通常不适用于测试与系统有交互的UI组件。测试这种情况应当使用[ActivityInstrumentationTestCase2](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html)。

这节课将会学习如何使用单元测试来验证用于启动Activity的Intent。因为测试运行于独立的环境之中，所以Intent并不会被实际发送到Android系统，但是你可以检测该Intent中携带的数据是否正确。

##创建用于Activity单元测试的测试用例
类[ActivityUnitTestCase](http://android.xsoftlab.net/reference/android/test/ActivityUnitTestCase.html)对单个的Activity测试提供了支持。要进行Activity的单元测试，需继承[ActivityUnitTestCase](http://android.xsoftlab.net/reference/android/test/ActivityUnitTestCase.html)。

在[ActivityUnitTestCase](http://android.xsoftlab.net/reference/android/test/ActivityUnitTestCase.html)中的Activity并不会由Android系统自动启动。如果要在这里启动Activity，必须在这里显式的调用startActivity()方法，并传入要执行的Intent。

例如：
```java
public class LaunchActivityTest
        extends ActivityUnitTestCase<LaunchActivity> {
    ...
    @Override
    protected void setUp() throws Exception {
        super.setUp();
        mLaunchIntent = new Intent(getInstrumentation()
                .getTargetContext(), LaunchActivity.class);
        startActivity(mLaunchIntent, null, null);
        final Button launchNextButton =
                (Button) getActivity()
                .findViewById(R.id.launch_next_activity_button);
    }
}
```

##验证另一个Activity的启动
单元测试可能含有以下目的：

- 验证在Button按下后LaunchActivity是否启动了Intent.
- 验证被启动的Intent所包含的数据是否正确.

为了验证在Button按下后，是否有Intent被触发，开发者可以使用getStartedActivityIntent()方法。然后通过断言方法来验证该方法返回的Intent是否为null，以及该Intent所含的数据是否正确。如果两个断言方法都正确，那么可以断定成功了触发了Intent。

开发者所实现的代码可能如下：
```java
@MediumTest
public void testNextActivityWasLaunchedWithIntent() {
    startActivity(mLaunchIntent, null, null);
    final Button launchNextButton =
            (Button) getActivity()
            .findViewById(R.id.launch_next_activity_button);
    launchNextButton.performClick();
    final Intent launchIntent = getStartedActivityIntent();
    assertNotNull("Intent was null", launchIntent);
    assertTrue(isFinishCalled());
    final String payload =
            launchIntent.getStringExtra(NextActivity.EXTRAS_PAYLOAD_KEY);
    assertEquals("Payload is empty", LaunchActivity.STRING_PAYLOAD, payload);
}
```

因为LaunchActivity是独立运行的，所以不能够使用库[TouchUtils](http://android.xsoftlab.net/reference/android/test/TouchUtils.html)来直接控制UI。为了可以模拟Button的点击效果，可以直接调用[performClick()](http://android.xsoftlab.net/reference/android/view/View.html#performClick())方法。