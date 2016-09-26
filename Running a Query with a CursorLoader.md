原文地址：[http://android.xsoftlab.net/training/load-data-background/index.html](http://android.xsoftlab.net/training/load-data-background/index.html)

#引言
在ContentProvider中查询数据是需要花点时间的。如果你直接在Activity进行查询，那么这可能会导致UI线程阻塞，并会引起"Application Not Responding"异常。就算是不会发生这些事情，那么用户也能感觉到卡顿，这会非常恼人的。为了避免这样的问题，应该将查询的工作放在单独的线程中执行，然后等待它执行完毕后将结果显示出来。

你可以使用一个异步查询对象在后台查询，然后等查询结束之后再与Activity建立连接。这个对象就是我们要说的[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)。[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)除了可以进行基本查询之外，还可以在数据发生变化后自动的重新运行查询。

这节课主要会学习如何使用[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)在后台进行查询。

#使用[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)进行查询
[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)对象在后台运行着一个异步查询，当查询结束之后会将结果返回到Activity或FragmentActivity。这使得查询在进行的过程中Activity或FragmentActivity还可以继续与用户交互。

##定义使用[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)的Activity
如果要在Activity中使用[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)，需要用到[LoaderCallbacks<Cursor>](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html)接口。[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)会调用该接口中的方法，从而使得与Activity产生交互。这节课与下节课都会详细的描述接口中的回调。

举个例子，下面的代码演示了如何定义一个使用了支持库版本的[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)的[FragmentActivity](http://android.xsoftlab.net/reference/android/support/v4/app/FragmentActivity.html)。通过继承[FragmentActivity](http://android.xsoftlab.net/reference/android/support/v4/app/FragmentActivity.html)，你可以获取[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)对Fragment的支持：
```java
public class PhotoThumbnailFragment extends FragmentActivity implements
        LoaderManager.LoaderCallbacks<Cursor> {
...
}
```

##初始化查询
使用[LoaderManager.initLoader()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.html#initLoader(int, android.os.Bundle, android.support.v4.app.LoaderManager.LoaderCallbacks<D>))可以初始化查询。它初始化了后台查询框架。可以将这部分工作放在用户输入了需要查询的数据之后，或者如果不需要用户输入数据，那么也可以将这部分工作放在[onCreate()](http://android.xsoftlab.net/reference/android/support/v4/app/FragmentActivity.html#onCreate(android.os.Bundle))或[onCreateView()](http://android.xsoftlab.net/reference/android/support/v4/app/Fragment.html#onCreateView(android.view.LayoutInflater, android.view.ViewGroup, android.os.Bundle))中执行：
```java
    // Identifies a particular Loader being used in this component
    private static final int URL_LOADER = 0;
    ...
    /* When the system is ready for the Fragment to appear, this displays
     * the Fragment's View
     */
    public View onCreateView(
            LayoutInflater inflater,
            ViewGroup viewGroup,
            Bundle bundle) {
        ...
        /*
         * Initializes the CursorLoader. The URL_LOADER value is eventually passed
         * to onCreateLoader().
         */
        getLoaderManager().initLoader(URL_LOADER, null, this);
        ...
    }
```

> **Note:** [getLoaderManager()](http://android.xsoftlab.net/reference/android/support/v4/app/Fragment.html#getLoaderManager())方法只对Fragment类可用。如果需要在[FragmentActivity](http://android.xsoftlab.net/reference/android/support/v4/app/FragmentActivity.html)中获得[LoaderManager](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.html)，调用[getSupportLoaderManager()](http://android.xsoftlab.net/reference/android/support/v4/app/FragmentActivity.html#getSupportLoaderManager())方法即可。

##开始查询
后台查询框架的初始化一旦完成，紧接着你所实现的[onCreateLoader()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html#onCreateLoader(int, android.os.Bundle))就会被调用。如果要启动查询，需要在该方法内返回一个[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)对象。你可以实例化一个空的[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)，然后再使用它的方法定义查询，或者你也可以在实例化[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)的时候定义查询。
```java
/*
* Callback that's invoked when the system has initialized the Loader and
* is ready to start the query. This usually happens when initLoader() is
* called. The loaderID argument contains the ID value passed to the
* initLoader() call.
*/
@Override
public Loader<Cursor> onCreateLoader(int loaderID, Bundle bundle)
{
    /*
     * Takes action based on the ID of the Loader that's being created
     */
    switch (loaderID) {
        case URL_LOADER:
            // Returns a new CursorLoader
            return new CursorLoader(
                        getActivity(),   // Parent activity context
                        mDataUrl,        // Table to query
                        mProjection,     // Projection to return
                        null,            // No selection clause
                        null,            // No selection arguments
                        null             // Default sort order
        );
        default:
            // An invalid id was passed in
            return null;
    }
}
```

一旦后台查询框架获得了该对象，那么它会马上在后台开始查询。当查询结果完成，后台查询框架会调用[onLoadFinished()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html#onLoadFinished(android.support.v4.content.Loader<D>, D))，该方法的具体内容会在下节课说明。
