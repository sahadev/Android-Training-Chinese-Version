原文地址：[http://android.xsoftlab.net/training/basics/fragments/index.html](http://android.xsoftlab.net/training/basics/fragments/index.html)

#导言
为了在Android中创建动态的多面板用户界面，你需要将UI组件和活动的行为封装到模块中，这些模块可以在activity中互相交换。你可以使用类Fragment创建这些模块，这些习性看起来像一个鸟巢状的activity，它允许定义自己的布局和管理自己的生命周期。

当Fragment指定了布局，它可以与其它fragment以不同的方式组合进activity中，以便可以对不同的屏幕尺寸做适应，小屏幕可能需要在同一时刻只展示一个fragment，而大屏幕可以同时展示两个或两个以上的fragment。

这节课将会展示如何通过fragment创建动态的用户效果，并改进APP在不同屏幕尺寸上的展示的用户效果，同时继续支持运行Android1.6系统的设备。

#创建一个Fragment
你可以把fragment想象为activity的一个单独的模块，这个模块拥有自己的生命周期，并且可以接收用户输入。并且可以在activity运行的时候添加或移除fragment。这节课展示了如何继承支持库中的Fragment，从而可以使APP可以运行在Android1.6这种低版本上。

>**Note:**如果选择了APP的最低支持版本为11或者11以上，那么就不需要使用支持库。这时可以使用框架中的Fragment类。不过需要意识到这节课使用的是支持库中的Fragment，这里的API名称可能会与框架中的Fragment名称有稍微的不同。

在开始这节课之前，必须确认工程已经使用了支持库。如果还没有使用支持库的话，请选择v4包中的支持库。你也可以使用包含了ActionBar的v7支持库，不过它只能运行在Android2.1及以上的版本上。

##创建一个Fragment类
如果要创建一个Fragment，需要继承Fragment类，然后重写关键的生命周期函数，与Activity类的方式很相似。

唯一的不同就是在创建Fragment时必须使用onCreateView方法定义布局。实际上，如果要简单运行的话，这是唯一个需要重写的方法。举个例子：
```java
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.ViewGroup;
public class ArticleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.article_view, container, false);
    }
}
```

正如Activity一样，一个fragment应该实现其它生命周期函数，这些生命周期函数可以管理各种状态，比如说添加到activity的状态或是从activity移除的状态，或是像activity类似的生命周期状态。举个例子，当activity的onPause方法被调用时，在activity中的每一个fragment的onPause方法也会被调用。

有关更多fragment的生命周期以及回调函数介绍请参见[Fragments](http://android.xsoftlab.net/guide/components/fragments.html)开发文档。

##使用XML将Fragment添加到Activity中
虽然fragment有可重用化、UI组件模块化的特点，但是每一个fragment都需要与FragmentActivity关联才能使用。你可以在activity的XML布局文件中通过定义的方式达成这种关联。

>**Note:**FragmentActivity是支持库中一个特殊的activity，它使得fragment可以在版本11以下的设备上运行。如果APP的最低支持版本在11及11以上的话，可以使用正规的Activity类。

下面的这个布局例子展示了当设备的屏幕被认为是"large"时，添加了两个Fragment到Activity中(特别指明布局文件夹的目录包含"large"标识符):
*res/layout-large/news_articles.xml*
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
    <fragment android:name="com.example.android.fragments.HeadlinesFragment"
              android:id="@+id/headlines_fragment"
              android:layout_weight="1"
              android:layout_width="0dp"
              android:layout_height="match_parent" />
    <fragment android:name="com.example.android.fragments.ArticleFragment"
              android:id="@+id/article_fragment"
              android:layout_weight="2"
              android:layout_width="0dp"
              android:layout_height="match_parent" />
</LinearLayout>
```
>**Tips:**有关更多针对不同的屏幕尺寸创建布局的信息，请参见：[Supporting Different Screen Sizes(本系列也有翻译，请自行查找)](http://android.xsoftlab.net/training/multiscreen/screensizes.html)。

然后将布局应用到Activity中：
```java
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;
public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);
    }
}
```

如果你使用了[v7 appcompat library](http://android.xsoftlab.net/tools/support-library/features.html#v7-appcompat)，Activity应该替换为继承[ActionBarActivity](http://android.xsoftlab.net/reference/android/support/v7/app/ActionBarActivity.html)，它是[FragmentActivity](http://android.xsoftlab.net/reference/android/support/v4/app/FragmentActivity.html)的子类。

> **Note：**如果通过XML布局的方式将Fragment添加到了Activity中，那么在运行时是不可以移除Fragment的。如果你计划在用户使用的时候将Fragment换进换出，那么你必须在Activity第一次启动的时候把fragment添加进去。下一节课将会讨论这些。