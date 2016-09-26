原文地址：[http://android.xsoftlab.net/training/basics/fragments/communicating.html](http://android.xsoftlab.net/training/basics/fragments/communicating.html)

为了可以重复使用Fragment UI组件，你应该将fragment构建为一个完整的独立的模块化组件，并且它可以定义自己的布局和行为习惯。你只要定义了一次这类可复用的fragment，你就可以通过activity与之相关联，然后使用应用程序逻辑与之建立连接以实现完整的复合型UI组件。

经常你会希望一个fragment对象可以和其它的fragment对象进行通信，举个例子，基于用户的事件更改内容。所有的fragment之间的通信都是通过它们关联的activity进行的。两个fragment对象应该永远不要直接通信。

##定义一个接口
为了使fragment与其关联的activity建立通信渠道，你可以在fragment中定义一个接口，然后让activity实现它。这个fragment对象可以在生命周期方法onAttach中取得这个接口的实现，然后就可以调用这个接口的方法与其关联的activity进行通信了。

下面是fragment与activity通信的例子：
```java
public class HeadlinesFragment extends ListFragment {
    OnHeadlineSelectedListener mCallback;
    // Container Activity must implement this interface
    public interface OnHeadlineSelectedListener {
        public void onArticleSelected(int position);
    }
    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        
        // This makes sure that the container activity has implemented
        // the callback interface. If not, it throws an exception
        try {
            mCallback = (OnHeadlineSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnHeadlineSelectedListener");
        }
    }
    
    ...
}
```

现在fragment就可以通过调用OnHeadlineSelectedListener接口的具体实现对象mCallback的onArticleSelected方法(或者接口中的其它方法)给activity传送消息了。

举个例子，下面这个方法会在用户点击列表条目的时候被调用，然后fragment会使用回调接口将事件传送给它所附属的activity。

##实现接口
为了可以从fragment中接收事件回调，那么包含fragment的activity必须实现在fragment中定义的接口。

举个例子，下面是上面例子中的在activity中的接口实现：
```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...
    
    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article
    }
}
```

##发送消息给fragment
宿主activity可以通过findFragmentById()方法获得fragment的对象，然后再发送消息到fragment中。可以直接调用fragment的public方法发送消息。

举个例子，想象上面例子中的activity可能还包含有其它的fragment，这些fragment可能被用来显示上面回调方法返回的数据。在这个例子中，activity会将从回调方法中接收到的信息传递给其它的fragment，其它的fragment会将这些信息显示出来：
```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...
    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article
        ArticleFragment articleFrag = (ArticleFragment)
                getSupportFragmentManager().findFragmentById(R.id.article_fragment);
        if (articleFrag != null) {
            // If article frag is available, we're in two-pane layout...
            // Call a method in the ArticleFragment to update its content
            articleFrag.updateArticleView(position);
        } else {
            // Otherwise, we're in the one-pane layout and must swap frags...
            // Create fragment and give it an argument for the selected article
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
        }
    }
}
```