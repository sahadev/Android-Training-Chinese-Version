原文地址：[http://android.xsoftlab.net/training/secure-file-sharing/request-file.html](http://android.xsoftlab.net/training/secure-file-sharing/request-file.html)

当APP需要访问一个被其它APP所共享的文件时，这个APP通常需要发送一个请求给共享文件的那个APP(服务端)，在大多数的情况下，这个请求会启动一个服务端的Activity，这个Activity会展示可以共享的文件。用户可以选择一个文件，稍后服务端APP会将这个文件以URI的形式返回给客户端APP。

这节课展示了客户端APP如何向服务端APP请求一个共享文件，以及从服务端APP接收这个URI，和通过这个URI打开被选中的文件。

##发送文件请求
如果要请求服务端的文件，客户端APP需要调用startActivityForResult方法并传入一个Intent对象，这个Intent对象包含了一个行为比如ACTION_PICK以及一个MIME类型，这个类型是指客户端APP可以处理的类型。

举个例子，下面这段代码演示了如何发送一个Intent给服务端APP并启动展示共享文件的那个Activity：
```java
public class MainActivity extends Activity {
    private Intent mRequestFileIntent;
    private ParcelFileDescriptor mInputPFD;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mRequestFileIntent = new Intent(Intent.ACTION_PICK);
        mRequestFileIntent.setType("image/jpg");
        ...
    }
    ...
    protected void requestFile() {
        /**
         * When the user requests a file, send an Intent to the
         * server app.
         * files.
         */
            startActivityForResult(mRequestFileIntent, 0);
        ...
    }
    ...
}
```

##访问请求到的文件
服务端给客户端返回了一个带有文件URI的Intent。这个Intent会从客户端中的onActivityResult()方法返回。一旦客户端有了这个文件的URI，那么它就可以通过[FileDescriptor](http://android.xsoftlab.net/reference/java/io/FileDescriptor.html)来访问这个文件。

在这个过程中，文件的安全性一直被保留，因为客户端接收到的URI只是数据的一部分。既然这个URI没有包含目录路径，那么客户端APP不可能发现并打开任何服务端上的任何其它文件。只有客户端APP可以访问文件,且仅仅是由服务器APP授予的权限。这个权限是个临时的权限，所以一旦客户端APP的任务终止，那么这个文件就不可被服务端APP之外的地方所访问。

下面这段代码演示了客户端APP如何处理从服务端返回的Intent，以及如何使用URI来获得[FileDescriptor](http://android.xsoftlab.net/reference/java/io/FileDescriptor.html)对象：
```java
/*
     * When the Activity of the app that hosts files sets a result and calls
     * finish(), this method is invoked. The returned Intent contains the
     * content URI of a selected file. The result code indicates if the
     * selection worked or not.
     */
    @Override
    public void onActivityResult(int requestCode, int resultCode,
            Intent returnIntent) {
        // If the selection didn't work
        if (resultCode != RESULT_OK) {
            // Exit without doing anything else
            return;
        } else {
            // Get the file's content URI from the incoming Intent
            Uri returnUri = returnIntent.getData();
            /*
             * Try to open the file for "read" access using the
             * returned URI. If the file isn't found, write to the
             * error log and return.
             */
            try {
                /*
                 * Get the content resolver instance for this context, and use it
                 * to get a ParcelFileDescriptor for the file.
                 */
                mInputPFD = getContentResolver().openFileDescriptor(returnUri, "r");
            } catch (FileNotFoundException e) {
                e.printStackTrace();
                Log.e("MainActivity", "File not found.");
                return;
            }
            // Get a regular file descriptor for the file
            FileDescriptor fd = mInputPFD.getFileDescriptor();
            ...
        }
    }
```

方法[openFileDescriptor()](http://android.xsoftlab.net/reference/android/content/ContentResolver.html#openFileDescriptor(android.net.Uri,%20java.lang.String))返回了一个文件的[ParcelFileDescriptor](http://android.xsoftlab.net/reference/android/os/ParcelFileDescriptor.html)对象。客户端APP可以根据这个对象得到[FileDescriptor](http://android.xsoftlab.net/reference/java/io/FileDescriptor.html)对象，这个对象便可以用来读取文件了。