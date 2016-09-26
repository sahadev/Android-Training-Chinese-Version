原文地址：[http://android.xsoftlab.net/training/multiscreen/adaptui.html](http://android.xsoftlab.net/training/multiscreen/adaptui.html)

基于程序当前所显示的布局来说，UI流程可能会有所不同。比如说，如果程序当前处于多面板模式，点击左面板中的项目会直接在右面版中显示具体的内容;如果当前是单面板模式，那么具体的内容则会在新的页面中显示。

##检查当前的布局
因为每种布局的实现可能会有所不同，所以首先要做的事情就是检查用户当前使用的是哪种布局。比如说，你可能需要知道用户当前处于"单面板"模式还是"多面板"模式。你可以通过查询给定的View是否存在及是否可见的方式来得知当前的模式。

```java
public class NewsReaderActivity extends FragmentActivity {
    boolean mIsDualPane;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main_layout);
        View articleView = findViewById(R.id.article);
        mIsDualPane = articleView != null && 
                        articleView.getVisibility() == View.VISIBLE;
    }
}
```

注意这部分代码在查询"article"面板是否可用，这要比查询指定布局的方式要灵活的多。

如何适配不同的组件的另一个示例是通过检查这些组件是否可用的方式来完成的。比如说，在新闻阅读APP中，有一个用于打开菜单的按钮，但是这个按钮只在3.0以上的版本才有。所以，如果要为这个按钮添加监听器，你可以这么做：
```java
Button catButton = (Button) findViewById(R.id.categorybutton);
OnClickListener listener = /* create your listener here */;
if (catButton != null) {
    catButton.setOnClickListener(listener);
}
```
##根据当前的布局做出响应
一些行为可能基于当前的布局产生不同的结果。比如说，在新闻阅读APP中，点击任意一条新闻标题，在多面板模式中，具体文章则会出现在右面板中，但是在单面板模式中，则会启动一个新的Activity来显示这些文章。
```java
@Override
public void onHeadlineSelected(int index) {
    mArtIndex = index;
    if (mIsDualPane) {
        /* display article on the right pane */
        mArticleFragment.displayArticle(mCurrentCat.getArticle(index));
    } else {
        /* start a separate activity */
        Intent intent = new Intent(this, ArticleActivity.class);
        intent.putExtra("catIndex", mCatIndex);
        intent.putExtra("artIndex", index);
        startActivity(intent);
    }
}
```

同样的，如果APP当前处于多面板模式，那么应该设置带有tab的ActionBar用于导航，然而，在单面板模式下，就应当设置带有spinner的导航控件。所以代码中还应当检查当前是哪种情况：
```java
final String CATEGORIES[] = { "Top Stories", "Politics", "Economy", "Technology" };
public void onCreate(Bundle savedInstanceState) {
    ....
    if (mIsDualPane) {
        /* use tabs for navigation */
        actionBar.setNavigationMode(android.app.ActionBar.NAVIGATION_MODE_TABS);
        int i;
        for (i = 0; i < CATEGORIES.length; i++) {
            actionBar.addTab(actionBar.newTab().setText(
                CATEGORIES[i]).setTabListener(handler));
        }
        actionBar.setSelectedNavigationItem(selTab);
    }
    else {
        /* use list navigation (spinner) */
        actionBar.setNavigationMode(android.app.ActionBar.NAVIGATION_MODE_LIST);
        SpinnerAdapter adap = new ArrayAdapter(this, 
                R.layout.headline_item, CATEGORIES);
        actionBar.setListNavigationCallbacks(adap, handler);
    }
}
```

##重用Fragment
在设计多面板的应用时会反复出现的一个场景，有一部分UI在一种屏幕配置中以面板的形式出现，而在其它的配置中，又是以独立的Activity出现。

在类似这种情况下，你可以通过重用Fragment的方式来避免代码冗余。比如，ArticleFragment就用于多面板的情况：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="400dp"
              android:layout_marginRight="10dp"/>
    <fragment android:id="@+id/article"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.ArticleFragment"
              android:layout_width="fill_parent" />
</LinearLayout>
```

在小屏幕中，又被Activity重用:
```java
ArticleFragment frag = new ArticleFragment();
getSupportFragmentManager().beginTransaction().add(android.R.id.content, frag).commit();
```

上面的代码与在XML布局中声明Fragment含有相同的效果，但是这种情况下XML布局就没必要工作了，因为article Fragment作为了这个Activity的组件。

一个非常重要的点要记住，在设计Fragment时不要与指定的Activity产生强耦合。你可以通过定义接口的方式来使Fragment与宿主Activity产生交互，宿主Activity需要实现这个接口：
```java
public class HeadlinesFragment extends ListFragment {
    ...
    OnHeadlineSelectedListener mHeadlineSelectedListener = null;
    /* Must be implemented by host activity */
    public interface OnHeadlineSelectedListener {
        public void onHeadlineSelected(int index);
    }
    ...
    public void setOnHeadlineSelectedListener(OnHeadlineSelectedListener listener) {
        mHeadlineSelectedListener = listener;
    }
}
```

因此，当用户选择了一条新闻时，Fragment通过接口的方式来通知宿主Activity:
```java
public class HeadlinesFragment extends ListFragment {
    ...
    @Override
    public void onItemClick(AdapterView<?> parent, 
                            View view, int position, long id) {
        if (null != mHeadlineSelectedListener) {
            mHeadlineSelectedListener.onHeadlineSelected(position);
        }
    }
    ...
}
```

##处理屏幕配置变更
如果使用了单独的Activity实现了UI的独立部分，那么要记得响应某些配置的变化，比如屏幕旋转，以便保持UI的一致性。

比如说，一款运行Android 3.0系统的7英寸平板，新闻阅读APP在垂直模式下使用的是独立的Activity展示文章的内容，但是在水平模式下使用的是多面板模式。

如果用户当前处于垂直模式下，那么需要检查方向更改为了水平模式，并需要通过结束结尾Activity并返回MainActivity的方式来让内容展示于双面板模式：
```java
public class ArticleActivity extends FragmentActivity {
    int mCatIndex, mArtIndex;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mCatIndex = getIntent().getExtras().getInt("catIndex", 0);
        mArtIndex = getIntent().getExtras().getInt("artIndex", 0);
        // If should be in two-pane mode, finish to return to main activity
        if (getResources().getBoolean(R.bool.has_two_panes)) {
            finish();
            return;
        }
        ...
}
```
