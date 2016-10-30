原文地址：[http://android.xsoftlab.net/training/activity-testing/activity-ui-testing.html](http://android.xsoftlab.net/training/activity-testing/activity-ui-testing.html)

一般来说，Activity包含UI组件，以便可以使用户与程序交互。这节课将会介绍如何测试Activity中的Button组件。往后便可以使用相同的方法对其它组件进行测试。

> **Note:**这节课中所涉及的UI测试被称为白盒测试，因为你持有被测试的源代码。Android设备框架适用于对UI组件创建白盒测试。另一种测试类型被称为黑盒测试，因为不能够访问程序的源代码，故此得名。这种测试适用于与其它APP或系统交互的情况。黑盒测试在这里并不会涵盖。有关更多如果执行黑盒测试的内容，请参见[UI测试指南](http://android.xsoftlab.net/tools/testing/testing_ui.html)。

##创建UI测试用例
虽然Activity运行于UI线程，但是测试程序本身是运行在子线程中的。这意味着，虽然TestAPP可以引用UI线程的对象，但是如果要更改这些对象的属性或者发送事件给UI线程，那么将会得到一个WrongThreadException错误。

为了可以安全的发送Intent到Activity或者在UI线程中运行测试方法，开发者可以使测试类继承于[ActivityInstrumentationTestCase2](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html)。

###设置测试先决条件
当为UI测试设置先决条件时，则需要在setUp()方法中指定TouchMode。设置TouchMode为true可以使后面的测试方法在自动化点击UI组件时防止持有焦点(例如，测试Button按钮只是调用了它的onclick方法)。另外要确保在调用[getActivity()](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html#getActivity())方法之前调用[setActivityInitialTouchMode()](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html#setActivityInitialTouchMode(boolean))。

例如：
```java
public class ClickFunActivityTest
        extends ActivityInstrumentationTestCase2 {
    ...
    @Override
    protected void setUp() throws Exception {
        super.setUp();
        setActivityInitialTouchMode(true);
        mClickFunActivity = getActivity();
        mClickMeButton = (Button) 
                mClickFunActivity
                .findViewById(R.id.launch_next_activity_button);
        mInfoTextView = (TextView) 
                mClickFunActivity.findViewById(R.id.info_text_view);
    }
}

```

