原文地址：[http://android.xsoftlab.net/training/basics/fragments/fragment-ui.html](http://android.xsoftlab.net/training/basics/fragments/fragment-ui.html)

当设计应用程序时需要支持尺寸较大宽屏设备，可以基于可用的屏幕空间在不同的布局中配置并重新使用fragment来提升用户体验。

举个例子，手持设备在同一时间可能只适合展示一个界面，相反的，你可能希望在平板设备上一边一个Fragment，因为平板有更宽的界面用来展示更多的信息。

![](http://android.xsoftlab.net/images/training/basics/fragments-screen-mock.png)
**上图中：**两个Fragment，利用同一个Activity在不同的屏幕尺寸中展示出不同的界面效果。在大屏幕中，两个fragment一边一个，但是在手持设备上，只能在同一时间内放置一个fragment，所以只能在用户使用的时候使用替换的方式来展示另一个fragment。

类FragmentManager支持在运行时添加、删除、替换fragment，以便提供更灵活的体验。

##在运行时添加Fragment到Activity中
正如上节课展示的那样，我们可以通过在布局文件中添加< fragment>标签的方式定义fragment，不过，我们还可以在activity运行的时候添加fragment到activity中。如果你计划在activity的生命周期内改变fragment的话，那么这项功能就很有必要了。

如果要执行类似添加、删除fragment的这种事务，必须通过使用FragmentManager创建一个事务对象FragmentTransaction，它提供了添加、删除、替换和其它fragment相关事务的功能。

如果activity允许fragment可以移除或者替换，那么必须在onCreate方法内初始化fragment并添加到activity中。

在处理fragment的时候有很重要的一条规则，尤其是在添加fragment的时候，就是activity必须包含一个容器View对象，以便fragment对象可以添加到这个容器中。

下面这个布局就是上一节课同时只显示一个fragment更改过后的布局，为了可以替换fragment，activity的布局需要包含一个空的FrameLayout，当做fragment的容器。

注意，文件的名称还是和上节课中布局的名字相同，但是布局的文件夹目录名称则不再包含"large"标识符，所以这个布局是在比"large"小的设备屏幕上使用的，因为这种屏幕不适合同时显示多个fragment。

res/layout/news_articles.xml:
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

在activity中调用getSupportFragmentManager()方法获得支持库中的FragmentManager对象，然后调用这个对象的beginTransaction()方法创建FragmentTransaction事务对象，通过这个事务对象的add()方法添加fragment。

你还可以使用FragmentTransaction事务对象执行多个fragment的事务。当准备确认要应用这些改变是，你应该调用commit()方法。

举个例子，下面这段代码展示了如何在上面的布局中添加fragment：
```java
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;
public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);
        // Check that the activity is using the layout version with
        // the fragment_container FrameLayout
        if (findViewById(R.id.fragment_container) != null) {
            // However, if we're being restored from a previous state,
            // then we don't need to do anything and should return or else
            // we could end up with overlapping fragments.
            if (savedInstanceState != null) {
                return;
            }
            // Create a new Fragment to be placed in the activity layout
            HeadlinesFragment firstFragment = new HeadlinesFragment();
            
            // In case this activity was started with special instructions from an
            // Intent, pass the Intent's extras to the fragment as arguments
            firstFragment.setArguments(getIntent().getExtras());
            
            // Add the fragment to the 'fragment_container' FrameLayout
            getSupportFragmentManager().beginTransaction()
                    .add(R.id.fragment_container, firstFragment).commit();
        }
    }
}
```

因为fragment在运行时被添加到了FrameLayout中，所以activity可以使用另一个不同的fragment来替换它，或者可以移除它。

##替换Fragment
替换fragment的过程和添加的过程很相似，只是需要使用replace()方法而不是add()方法。

记住，在执行fragment事务的时候，比如替换或者移除，经常需要适当的允许用户可以通过返回撤销改变。为了通过fragment事务允许用户做到这一点，必须在FragmentTransaction事务提交之前调用方法。

>**Note:**当你通过移除或者替换将fragment作为事务添加到回退栈的时候，那个被移除的fragment会进入停止状态(没有被销毁)。如果用户通过返回还原了fragment，那么它就会重新启动。如果没有添加事务到回退栈，那么fragment在移除或者替换的时候会被销毁。

这是个替换fragment的例子：
```java
// Create fragment and give it an argument specifying the article it should show
ArticleFragment newFragment = new ArticleFragment();
Bundle args = new Bundle();
args.putInt(ArticleFragment.ARG_POSITION, position);
newFragment.setArguments(args);
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack so the user can navigate back
transaction.replace(R.id.fragment_container, newFragment);
transaction.addToBackStack(null);
// Commit the transaction
transaction.commit();
```

addToBackStack()方法有一个可选的字符串参数，这个参数可以用来指定事务的唯一标示名称。这个名称不是必须的，除非你计划通过FragmentManager.BackStackEntry API执行更佳的fragment操作。