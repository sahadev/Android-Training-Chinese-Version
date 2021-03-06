原文地址：[http://android.xsoftlab.net/training/activity-testing/activity-ui-testing.html](http://android.xsoftlab.net/training/activity-testing/activity-ui-testing.html)

一般来说，正因为Activity含有UI组件，所以使得用户可以与程序交互。这节课将会介绍如何测试Activity中的Button组件。在课程学习之后便可以使用相同的方法对其它UI组件进行测试。

> **Note:**这节课中所涉及的UI测试被称为白盒测试，因为开发者本身持有被测试的源代码。Android设备框架适用于UI组件的白盒测试。另一种测试类型被称为黑盒测试，因为不能够得到程序的源代码，故此得名。这种测试适用于与其它APP或系统交互的测试情况。黑盒测试在这里并不会涵盖。有关更多如何执行黑盒测试的相关内容，请参见[UI测试指南](http://android.xsoftlab.net/tools/testing/testing_ui.html)。

##创建UI测试用例
虽然Activity运行于UI线程，但是测试程序本身是运行在子线程中的。这意味着，虽然TestAPP可以引用UI线程的对象，但是如果要更改这些对象的属性或者发送事件给UI线程，那么将会得到一个WrongThreadException错误。

为了可以安全的发送Intent到Activity或者在UI线程中运行测试方法，开发者可以使测试类继承于[ActivityInstrumentationTestCase2](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html)类。

###设置测试先决条件
当为UI测试设置先决条件时，则需要在setUp()方法中指定TouchMode。设置TouchMode为true可以使后面的测试方法在自动化点击UI组件时防止真正点击(例如，测试Button只是调用了它的onclick方法)。另外要确保在调用[getActivity()](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html#getActivity())方法之前调用了[setActivityInitialTouchMode()](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html#setActivityInitialTouchMode(boolean))方法。

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

##添加测试方法
一般需要测试的点会包含以下部分：

- 当Activity启动时，验证Button的布局是否显示正确。
- 验证TextView在初始化时是否是隐藏的。
- 验证Button按下后，TextView上的文本是否变为了期望的值。

下面将会演示如何测试以上部分：
###验证Button的布局参数
开发者可能需要以下代码来验证Button的布局是否正确：
```java
@MediumTest
public void testClickMeButton_layout() {
    final View decorView = mClickFunActivity.getWindow().getDecorView();
    ViewAsserts.assertOnScreen(decorView, mClickMeButton);
    final ViewGroup.LayoutParams layoutParams =
            mClickMeButton.getLayoutParams();
    assertNotNull(layoutParams);
    assertEquals(layoutParams.width, WindowManager.LayoutParams.MATCH_PARENT);
    assertEquals(layoutParams.height, WindowManager.LayoutParams.WRAP_CONTENT);
}
```

在调用[assertOnScreen()](http://android.xsoftlab.net/reference/android/test/ViewAsserts.html#assertOnScreen(android.view.View, android.view.View))方法时，应当将rootView以及需要验证的View传递进去。如果需要验证的View没有在rootView中出现，那么判断方法会抛出一个[AssertionFailedError](http://android.xsoftlab.net/reference/junit/framework/AssertionFailedError.html)异常。

开发者还可以通过Button的布局参数来验证Button的布局是否正确，然后通过判断方法来验证Button的高宽是否是期望中的值。

@MediumTest注解说明了这个测试方法应当如何分类。如何分类取决于测试方法的执行时间。

###验证TextView的布局参数
开发者也可能需要通过以下代码来验证TextView在初始化时是否是隐藏的：
```java
@MediumTest
public void testInfoTextView_layout() {
    final View decorView = mClickFunActivity.getWindow().getDecorView();
    ViewAsserts.assertOnScreen(decorView, mInfoTextView);
    assertTrue(View.GONE == mInfoTextView.getVisibility());
}
```
开发者可以通过[getDecorView()](http://android.xsoftlab.net/reference/android/view/Window.html#getDecorView())方法获得Activity的DecorView引用。DecorView在布局层级中属于最高等级的ViewGroup.

###验证Button的行为
开发者可以根据以下测试方法来验证在Button按下后TextView是否变为可见状态。
```
@MediumTest
public void testClickMeButton_clickButtonAndExpectInfoText() {
    String expectedInfoText = mClickFunActivity.getString(R.string.info_text);
    TouchUtils.clickView(this, mClickMeButton);
    assertTrue(View.VISIBLE == mInfoTextView.getVisibility());
    assertEquals(expectedInfoText, mInfoTextView.getText());
}
```

为了可以自动点击Button，需要调用[clickView()](http://android.xsoftlab.net/reference/android/test/TouchUtils.html#clickView%28android.test.InstrumentationTestCase,%20android.view.View%29)方法。该方法需要传入测试用例的引用以及对应Button的引用。

> **Note:** 辅助类[TouchUtils](http://android.xsoftlab.net/reference/android/test/TouchUtils.html)提供了一些用于模拟交互的简单方法，开发者可以使用这些方法来模拟点击，拖拽等事件。

> **Note:** [TouchUtils](http://android.xsoftlab.net/reference/android/test/TouchUtils.html)中的方法用于从测试线程向UI线程中发送事件。开发者最好不要在UI线程中直接调用[TouchUtils](http://android.xsoftlab.net/reference/android/test/TouchUtils.html)的相关方法，否则会引起WrongThreadException异常.

##测试注解
以下注解可以用来标明测试方法的大小：

- @SmallTest

- @MediumTest

- @LargeTest

一般来说，一个只有几毫秒的剪短测试一般应该标为@SmallTest。稍长一点的，大概100毫秒左右的，通常应该标为@MediumTest或@LargeTest，测试的执行时间通常取决于是否需要访问本地资源或者网络资源。

开发者应当通过注解来标记测试方法，以便更好的组织、运行测试。