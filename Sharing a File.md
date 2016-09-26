原文地址：[http://android.xsoftlab.net/training/secure-file-sharing/share-file.html](http://android.xsoftlab.net/training/secure-file-sharing/share-file.html)

一旦APP设置通过URI的方式共享文件，你需要响应其它APP请求这些文件的请求。响应这些请求的一种方式是，在服务端APP上提供一个文件选择接口，以便其它的程序可以调用。这种方法允许客户端程序的用户从服务端选择一个文件，然后接收被选择文件的URI地址。

这节课展示了如何在Activity中创建文件选择功能，以便响应文件请求。

##接收文件请求
为了接收客户端APP的文件请求，以及以URI的方式作出响应，APP应该在Activity中提供文件选择器。这样的话，客户端APP可以通过调用startActivityForResult()方法启动这个Activity。当客户端调用了startActivityForResult()方法，你的APP可以返回一个结果给客户端APP，这个结果以URI的形式将用户选择的文件返回。

有关学习如何在客户端APP中实现文件的请求，请参见课程： [Requesting a Shared File](http://android.xsoftlab.net/training/secure-file-sharing/request-file.html)。

##创建一个文件选择器Activity
为了设置文件选择器Activity，首选需要在清单文件中指定activity，并在其中附加意图过滤器，这个意图过滤器用来匹配行为[ACTION_PICK](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_PICK)以及类别[CATEGORY_DEFAULT](http://android.xsoftlab.net/reference/android/content/Intent.html#CATEGORY_DEFAULT)和[CATEGORY_OPENABLE](http://android.xsoftlab.net/reference/android/content/Intent.html#CATEGORY_OPENABLE)。还要给服务端给客户端提供的文件添加MIME类型过滤器。下面的代码片段展示了如何指定一个新Activity以及过滤器：
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    ...
        <application>
        ...
            <activity
                android:name=".FileSelectActivity"
                android:label="@"File Selector" >
                <intent-filter>
                    <action
                        android:name="android.intent.action.PICK"/>
                    <category
                        android:name="android.intent.category.DEFAULT"/>
                    <category
                        android:name="android.intent.category.OPENABLE"/>
                    <data android:mimeType="text/plain"/>
                    <data android:mimeType="image/*"/>
                </intent-filter>
            </activity>
```

##在代码中定义文件选择器
接下来，定义一个Activity来展示内部存储器中files/images/目录下的可用文件，并允许用户来选择需要的文件。下面这段代码演示了如何定义一个Activity来响应用户的选择：
```java
public class MainActivity extends Activity {
    // The path to the root of this app's internal storage
    private File mPrivateRootDir;
    // The path to the "images" subdirectory
    private File mImagesDir;
    // Array of files in the images subdirectory
    File[] mImageFiles;
    // Array of filenames corresponding to mImageFiles
    String[] mImageFilenames;
    // Initialize the Activity
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Set up an Intent to send back to apps that request a file
        mResultIntent =
                new Intent("com.example.myapp.ACTION_RETURN_FILE");
        // Get the files/ subdirectory of internal storage
        mPrivateRootDir = getFilesDir();
        // Get the files/images subdirectory;
        mImagesDir = new File(mPrivateRootDir, "images");
        // Get the files in the images subdirectory
        mImageFiles = mImagesDir.listFiles();
        // Set the Activity's result to null to begin with
        setResult(Activity.RESULT_CANCELED, null);
        /*
         * Display the file names in the ListView mFileListView.
         * Back the ListView with the array mImageFilenames, which
         * you can create by iterating through mImageFiles and
         * calling File.getAbsolutePath() for each File
         */
         ...
    }
    ...
}

```

##响应文件选择器
一旦用户选择了被共享的文件，你的程序必须检查哪一个文件被选中，并且生成该文件的URI地址。前面的部分Activity在[ListView](http://android.xsoftlab.net/reference/android/widget/ListView.html)中展示了可用的文件列表，当用户点击了文件的名称，随之系统会调用方法[onItemClick()](http://android.xsoftlab.net/reference/android/widget/AdapterView.OnItemClickListener.html#onItemClick(android.widget.AdapterView%3C?%3E,%20android.view.View,%20int,%20long))，在这个方法中你可以获取到被选中的文件。

在[onItemClick()](http://android.xsoftlab.net/reference/android/widget/AdapterView.OnItemClickListener.html#onItemClick(android.widget.AdapterView%3C?%3E,%20android.view.View,%20int,%20long))方法中，从被选择的文件中获得文件的[File](http://android.xsoftlab.net/reference/java/io/File.html)对象，然后将这个对象作为参数传递给[getUriForFile()](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html#getUriForFile(android.content.Context,%20java.lang.String,%20java.io.File))，它会伴随着权限一并加入，这个权限是由 < provider>元素指定的。结果URI会包含权限、文件在相应目录中的路径段(在XML meta-data中指定的部分)以及文明的名称和其扩展部分。有关FileProvider如何映射meta-data与路径段的目录，请看章节[Specify Sharable Directories](http://android.xsoftlab.net/training/secure-file-sharing/setup-sharing.html#DefineMetaData)。

下面的代码段展示了如何检测被选择的文件以及获取该文件的URI：
```java
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks on a file in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            /*
             * When a filename in the ListView is clicked, get its
             * content URI and send it to the requesting app
             */
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                /*
                 * Get a File for the selected file name.
                 * Assume that the file names are in the
                 * mImageFilename array.
                 */
                File requestFile = new File(mImageFilename[position]);
                /*
                 * Most file-related method calls need to be in
                 * try-catch blocks.
                 */
                // Use the FileProvider to get a content URI
                try {
                    fileUri = FileProvider.getUriForFile(
                            MainActivity.this,
                            "com.example.myapp.fileprovider",
                            requestFile);
                } catch (IllegalArgumentException e) {
                    Log.e("File Selector",
                          "The selected file can't be shared: " +
                          clickedFilename);
                }
                ...
            }
        });
        ...
    }
```

要记住，你只可以对在< paths>元素中指定目录下的文件进行URI编码，就像[Specify Sharable Directories](http://android.xsoftlab.net/training/secure-file-sharing/setup-sharing.html#DefineMetaData)这节课中描述的一样。如果你对getUriForFile()方法中传入的File参数并没有在< path>元素中指定，那么你会收到一个 [IllegalArgumentException](http://android.xsoftlab.net/reference/java/lang/IllegalArgumentException.html)异常。

##对文件授予权限
现在对将要分享的文件有了一个URI，你需要允许客户端APP来访问这个文件。为了允许访问，需要通过添加URI到一个Intent上，并且在这个Intent上设置权限标志。产生的这个权限是个临时权限，并且会在接收端APP任务结束的时候自动终止。

下面这段代码展示了如何对文件设置读取权限：
```java
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                ...
                if (fileUri != null) {
                    // Grant temporary read permission to the content URI
                    mResultIntent.addFlags(
                        Intent.FLAG_GRANT_READ_URI_PERMISSION);
                }
                ...
             }
             ...
        });
    ...
    }
```

> **Caution:**安全的对文件授予临时访问权限，setFlags()是唯一的方式。要避免对文件的URI使用Context.grantUriPermission()方法，因为该方法授予的权限只可以通过Context.revokeUriPermission()方法撤销。

##对客户端APP共享文件
如果要共享文件给客户端APP，需要传递一个Intent给[setResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#setResult(int))，这个Intent包含了文件的URI以及访问权限。当你定义的这个Activity结束时，系统会发送这个Intent给客户端APP，下面这段代码展示了如何实现这部分功能：
```java
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks on a file in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                ...
                if (fileUri != null) {
                    ...
                    // Put the Uri and MIME type in the result Intent
                    mResultIntent.setDataAndType(
                            fileUri,
                            getContentResolver().getType(fileUri));
                    // Set the result
                    MainActivity.this.setResult(Activity.RESULT_OK,
                            mResultIntent);
                    } else {
                        mResultIntent.setDataAndType(null, "");
                        MainActivity.this.setResult(RESULT_CANCELED,
                                mResultIntent);
                    }
                }
        });
```

一旦用户选择了文件，那么就应该立即带用户返回客户端APP。实现这种方式的一种方法就是提供一个钩形符号或者**结束**按钮。使用Button的[android:onClick](http://android.xsoftlab.net/reference/android/view/View.html#attr_android:onClick)属性与对应的Button相关联。在方法中。调用[finish()](http://android.xsoftlab.net/reference/android/app/Activity.html#finish())方法：
```java
    public void onDoneClick(View v) {
        // Associate a method with the Done button
        finish();
    }
```

