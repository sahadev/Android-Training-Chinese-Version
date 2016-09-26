原文地址：[http://android.xsoftlab.net/training/load-data-background/handle-results.html](http://android.xsoftlab.net/training/load-data-background/handle-results.html)

就像上节课所说的，应该在[onCreateLoader()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html#onCreateLoader(int, android.os.Bundle))的实现方法内使用[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)来加载数据。那么加载完毕之后，结果会通过[LoaderCallbacks.onLoadFinished()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html#onLoadFinished(android.support.v4.content.Loader<D>, D))方法传回到实现方法中。该方法的其中一个参数为包含查询结果的[Cursor](http://android.xsoftlab.net/reference/android/database/Cursor.html)对象。你可以通过这个对象来更新UI数据或者用它来做更进一步的操作。

除了[onCreateLoader()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html#onCreateLoader(int, android.os.Bundle))及[onLoadFinished()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html#onLoadFinished(android.support.v4.content.Loader<D>, D))这两个方法之外，还应当实现[onLoaderReset()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html#onLoaderReset(android.support.v4.content.Loader<D>))。这个方法会在上面的[Cursor](http://android.xsoftlab.net/reference/android/database/Cursor.html)对象所关联的数据发生变化时调用。如果数据发生了变化，那么Android框架会重新运行当前的查询。

#处理查询结果
为了展示[Cursor](http://android.xsoftlab.net/reference/android/database/Cursor.html)对象中的数据，这里需要实现[AdapterView](http://android.xsoftlab.net/reference/android/widget/AdapterView.html)的实现类以及[CursorAdapter](http://android.xsoftlab.net/reference/android/support/v4/widget/CursorAdapter.html)的实现类。系统会自动的将[Cursor](http://android.xsoftlab.net/reference/android/database/Cursor.html)中的数据处理到View上。

你可以在显示数据之前将数据与Adapter产生关联，这样的话系统会自动的更新View：
```java
public String[] mFromColumns = {
    DataProviderContract.IMAGE_PICTURENAME_COLUMN
};
public int[] mToFields = {
    R.id.PictureName
};
// Gets a handle to a List View
ListView mListView = (ListView) findViewById(R.id.dataList);
/*
 * Defines a SimpleCursorAdapter for the ListView
 *
 */
SimpleCursorAdapter mAdapter =
    new SimpleCursorAdapter(
            this,                // Current context
            R.layout.list_item,  // Layout for a single row
            null,                // No Cursor yet
            mFromColumns,        // Cursor columns to use
            mToFields,           // Layout fields to use
            0                    // No flags
    );
// Sets the adapter for the view
mListView.setAdapter(mAdapter);
...
/*
 * Defines the callback that CursorLoader calls
 * when it's finished its query
 */
@Override
public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
    ...
    /*
     * Moves the query results into the adapter, causing the
     * ListView fronting this adapter to re-display
     */
    mAdapter.changeCursor(cursor);
}
```

移除旧的Cursor引用
[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)会在[Cursor](http://android.xsoftlab.net/reference/android/database/Cursor.html)处于无效时重置。这经常会发生，因为[Cursor](http://android.xsoftlab.net/reference/android/database/Cursor.html)所关联的数据经常会发生改变。在重新开始查询之前，系统框架会调用你所实现的[onLoaderReset()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html#onLoaderReset(android.support.v4.content.Loader<D>))方法。在该方法内，应该将当前[Cursor](http://android.xsoftlab.net/reference/android/database/Cursor.html)的所有引用移除，以便防止内存泄露。一旦[onLoaderReset()](http://android.xsoftlab.net/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html#onLoaderReset(android.support.v4.content.Loader<D>))执行完成，[CursorLoader](http://android.xsoftlab.net/reference/android/support/v4/content/CursorLoader.html)就会重新开始查询。
```java
/*
 * Invoked when the CursorLoader is being reset. For example, this is
 * called if the data in the provider changes and the Cursor becomes stale.
 */
@Override
public void onLoaderReset(Loader<Cursor> loader) {
    
    /*
     * Clears out the adapter's reference to the Cursor.
     * This prevents memory leaks.
     */
    mAdapter.changeCursor(null);
}
```

