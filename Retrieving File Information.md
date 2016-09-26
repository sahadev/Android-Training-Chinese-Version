原文地址：[http://android.xsoftlab.net/training/secure-file-sharing/retrieve-info.html](http://android.xsoftlab.net/training/secure-file-sharing/retrieve-info.html)

之前的课程讲述了客户端APP试图与含有文件的URI一同运行，APP可以请求服务端APP的文件信息，包括文件的数据类型以及文件的大小。这些数据类型可以帮助客户端APP来判断该文件是否可以处理，文件的大小可以帮助客户端APP对该文件设置相应大小的缓冲区。

这节课演示了如何查询服务端APP返回文件的MIME类型以及大小。

##获取文件的MIME类型
一个文件的数据类型指示了客户端APP应该如何处理这个文件的内容。为了获取URI对应文件的数据类型，客户端APP需要调用方法[ContentResolver.getType()](http://android.xsoftlab.net/reference/android/content/ContentResolver.html#getType(android.net.Uri))。这个方法返回了文件的MIME类型。默认情况下，[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)可以从文件的扩展名来判断文件的MIME类型。

下面这段代码演示了客户端APP如何解析服务端APP返回的URI对应文件的MIME类型：
```java
    ...
    /*
     * Get the file's content URI from the incoming Intent, then
     * get the file's MIME type
     */
    Uri returnUri = returnIntent.getData();
    String mimeType = getContentResolver().getType(returnUri);
    ...
```

##获取文件的名称与大小
[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)类有一个[query()](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html#query(android.net.Uri,%20java.lang.String[],%20java.lang.String,%20java.lang.String[],%20java.lang.String))方法的默认实现，该方法可以返回URI相关文件的名称与大小，不过结果位于一个[Cursor](http://android.xsoftlab.net/reference/android/database/Cursor.html)对象中。默认的实现会返回两列：

**[DISPLAY_NAME](http://android.xsoftlab.net/reference/android/provider/OpenableColumns.html#DISPLAY_NAME)**

- 这是文件的名称，是字符串类型。这个值与[File.getName()](http://android.xsoftlab.net/reference/java/io/File.html#getName())方法返回的值相等。

**[SIZE](http://android.xsoftlab.net/reference/android/provider/OpenableColumns.html#SIZE)**

- 这是文件的大小，以字节形式呈现，是long类型。这个值与[File.length()](http://android.xsoftlab.net/reference/java/io/File.html#length())方法返回的值相等。

客户端APP可以通过对[query()](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html#query(android.net.Uri,%20java.lang.String[],%20java.lang.String,%20java.lang.String[],%20java.lang.String))方法设置null参数的方式来获得文件的名称与大小，当然URI参数除外。举个例子，下面这段代码获取了一个文件的名称与大小，并且在单独的[TextView](http://android.xsoftlab.net/reference/android/widget/TextView.html)中进行了展示：
```java
    ...
    /*
     * Get the file's content URI from the incoming Intent,
     * then query the server app to get the file's display name
     * and size.
     */
    Uri returnUri = returnIntent.getData();
    Cursor returnCursor =
            getContentResolver().query(returnUri, null, null, null, null);
    /*
     * Get the column indexes of the data in the Cursor,
     * move to the first row in the Cursor, get the data,
     * and display it.
     */
    int nameIndex = returnCursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
    int sizeIndex = returnCursor.getColumnIndex(OpenableColumns.SIZE);
    returnCursor.moveToFirst();
    TextView nameView = (TextView) findViewById(R.id.filename_text);
    TextView sizeView = (TextView) findViewById(R.id.filesize_text);
    nameView.setText(returnCursor.getString(nameIndex));
    sizeView.setText(Long.toString(returnCursor.getLong(sizeIndex)));

```


