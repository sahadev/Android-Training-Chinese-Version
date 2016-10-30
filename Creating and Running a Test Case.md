原文地址：[http://android.xsoftlab.net/training/activity-testing/activity-basic-testing.html](http://android.xsoftlab.net/training/activity-testing/activity-basic-testing.html)

为了验证在布局设计及功能行为上没有差池，很重要的一点就是需要为每个Activity创建对应的测试。对于每个测试还需要创建单独的测试用例，这包含测试工具，条件测试方法以及Activity的测试方法。这样便可以进行测试并获取测试结果。如果其中一项测试失败了，这便意味着在代码中可能存在潜在的问题。

##创建测试用例
Activity测试都以结构化的方式实现。要确保将所有的测试放在一个单独的包中，与被测试代码区别开来。

依照惯例，测试包的包名应当由应用的包名+后缀".tests"组成。在完成测试包的创建之后，添加一个Java类以用于测试。依照惯例，该类的名称应当由要测试的类的类名+后缀"Test"组成。

在Eclipse中创建测试用例的步骤如下：
a.在工程中新建一个包。
b.设置包名为<your_app_package_name>.tests(例如，com.example.android.testingfun.tests)，并点击Finish。
c.在该包名下创建一个类。
d.设置类名为<your_app_activity_name>Test(例如，MyFirstTestActivityTest)，并点击Finish。

##设置测试先决条件
测试先决条件由一系列用于测试的对象组成。设置这些先决条件可以重写setUp()方法以及tearDown()方法。TestRunner会在进行测试之前自动的运行[setUp()](http://android.xsoftlab.net/reference/junit/framework/TestCase.html#setUp())，[tearDown()](http://android.xsoftlab.net/reference/junit/framework/TestCase.html#tearDown())方法在所有的测试结束之后运行。你可以使用这些方法来保证测试的正常初始化以及测试结束之后的清理工作。

在Eclipse中设置测试先决条件：
1.将上面创建好的测试类继承于[ActivityTestCase](http://android.xsoftlab.net/reference/android/test/ActivityTestCase.html)的任一子类。例如：
```java
public class MyFirstTestActivityTest
        extends ActivityInstrumentationTestCase2<MyFirstTestActivity> {
```
2.接下来，在这个类的内部添加构造方法以及[setUp()](http://android.xsoftlab.net/reference/junit/framework/TestCase.html#setUp())方法，添加要测试的Activity的变量声明。例如：
```java
public class MyFirstTestActivityTest
        extends ActivityInstrumentationTestCase2<MyFirstTestActivity> {
    private MyFirstTestActivity mFirstTestActivity;
    private TextView mFirstTestText;
    public MyFirstTestActivityTest() {
        super(MyFirstTestActivity.class);
    }
    @Override
    protected void setUp() throws Exception {
        super.setUp();
        mFirstTestActivity = getActivity();
        mFirstTestText =
                (TextView) mFirstTestActivity
                .findViewById(R.id.my_first_test_text_view);
    }
}
```
构造方法会在类初始化时由TestRunner调用，而[setUp()](http://android.xsoftlab.net/reference/junit/framework/TestCase.html#setUp())方法会在开始任何测试之前调用。

通常情况下，在[setUp()](http://android.xsoftlab.net/reference/junit/framework/TestCase.html#setUp())方法内，应当实现以下内容：
- 调用父类的[setUp()](http://android.xsoftlab.net/reference/junit/framework/TestCase.html#setUp())方法。
- 通过以下步骤初始化先决条件：
 - 定义实例变量用于存储先决条件的状态。
 - 创建并存储接下来要测试的Activity的引用。
 - 持有Activity中需要进行测试的UI组件的引用。

可以通过[getActivity()](http://android.xsoftlab.net/reference/android/test/ActivityInstrumentationTestCase2.html#getActivity())获取要测试的Activity的引用。

##添加测试条件
在进行测试之前，还有一个步骤就是需要验证前一步是否设置正确，以及需要测试的对象是否已被正确的实例化、初始化。这样的话，便不需要查看测试失败，因为测试的先决条件已经发生了错误。依照惯例，用于验证先决条件的方法被称为testPreconditions().

例如：
```java
public void testPreconditions() {
    assertNotNull(“mFirstTestActivity is null”, mFirstTestActivity);
    assertNotNull(“mFirstTestText is null”, mFirstTestText);
}
```
其中的判断方法来自于JUnit的[Assert](http://android.xsoftlab.net/reference/junit/framework/Assert.html)类。通常情况下可以使用这些判断方法来验证需要测试的指定条件是否为true。
- 如果条件为false，那么判断方法会抛出一个[AssertionFailedError](http://android.xsoftlab.net/reference/android/test/AssertionFailedError.html)异常。该异常由TestRunner抛出。如果判断失败，那么可以通过判断方法的第一个参数得知是哪个条件失败。
- 如果条件为true，那么测试会顺利执行。

在这两种情况中，TestRunner会继续执行其它的测试方法。

##添加测试方法
接下来，添加测试方法来验证Activity的布局与功能。

例如，如果Activity包含了一个TextView，你可以像下面这样添加一个测试方法来验证该TextView是否有正确的标签文本：
```java
public void testMyFirstTestTextView_labelText() {
    final String expected =
            mFirstTestActivity.getString(R.string.my_first_test);
    final String actual = mFirstTestText.getText().toString();
    assertEquals(expected, actual);
}
```

testMyFirstTestTextView_labelText()方法用于检测TextView的默认文本与定义在string.xml中的文本是否一致。

> **Note:** 当命名测试方法时，可以使用下划线来分开要测试的内容。这种编写风格可以更容易明确测试的内容。

要执行比较，将期望的值与实际的值传给[assertEquals()](http://android.xsoftlab.net/reference/junit/framework/Assert.html#assertEquals(java.lang.String,%20java.lang.String))方法。如果两个值不相等，那么将会抛出一个[AssertionFailedError](http://android.xsoftlab.net/reference/junit/framework/AssertionFailedError.html)异常。

如果添加testPreconditions()方法，那么请将测试代码放在testPreconditions()之后。

##构建运行测试
在Eclipse中进行代码测试非常容易。

请执行以下步骤：
1.将Android设备连接到计算机。打开Setting菜单，选择Developer选项，并确保USB调试模式已开启。
2.在测试的类中选择RunAs > Android Junit Test.
3.在Android设备选择对话框中，选择刚刚连接好的设备，点击OK。
4.在JUnit界面中，验证测试是否通过。

