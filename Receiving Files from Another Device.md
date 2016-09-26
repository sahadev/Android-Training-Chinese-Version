原文地址：[http://android.xsoftlab.net/training/beam-files/receive-files.html](http://android.xsoftlab.net/training/beam-files/receive-files.html)

Android Beam文件传输系统会将文件拷贝到接收设备的指定目录中。它还会使用Android媒体扫描器扫描被拷贝的文件，并会将媒体文件的入口信息添加到[MediaStore](http://android.xsoftlab.net/reference/android/provider/MediaStore.html)提供者中去。

##响应展示数据的请求
当Android Beam文件传输系统结束了文件拷贝，它会发送通知，这个通知会包含一个Intent对象，这个Intent对象的行为为ACTION_VIEW，以及被传送的第一个文件的MIME类型，以及指向第一个文件的URI地址。当用户点击了通知，这个Intent会被广播到系统中。为了使APP可以响应这个意图，需要在清单文件中的响应Activity的< activity>元素下添加<intent-filter>元素，并在其中添加如下子元素：

```xml
<action android:name="android.intent.action.VIEW" />
```

- 匹配由通知发送来的ACTION_VIEW意图。

```xml
<category android:name="android.intent.category.CATEGORY_DEFAULT" />
```

- 匹配没有明确类别的意图。

```xml
<data android:mimeType="mime-type" />
```

- 匹配一种MIME类型。这里的mime-type类型应为APP可以处理的类型。

举个例子，下面这段代码展示了如何在被触发的Activity com.example.android.nfctransfer.ViewActivity的清单文件中添加意图过滤器：
```xml
    <activity
        android:name="com.example.android.nfctransfer.ViewActivity"
        android:label="Android Beam Viewer" >
        ...
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            ...
        </intent-filter>
    </activity>
```

> **Note:**Android Beam文件传输系统并不是ACTION_VIEW意图的唯一来源，接收设备上的其它APP也可以发送含有这个行为的意图。[Get the directory from a content URI](http://android.xsoftlab.net/training/beam-files/receive-files.html#GetDirectory)这节会讨论如何处理这种情况。

##请求文件权限
为了可以读取Android Beam文件传输系统拷贝到设备的文件，需要请求[READ\_EXTERNAL\_STORAGE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)权限：
```xml
 <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

如果你想要将被拷贝的文件拷贝到自己的存储器上，这里应该使用权限[WRITE_EXTERNAL_STORAGE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)。[WRITE_EXTERNAL_STORAGE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)权限包含了[READ_EXTERNAL_STORAGE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)权限。

> **Note:**在Android 4.2.2上，READ_EXTERNAL_STORAGE权限会在用户需要的时候强制执行。未来的版本可能在各种情况下都需要该权限，所以为了确保向后兼容，请在它要求以前就给它添加上。

因为你的APP对其内部存储器有操作的权利，所以在拷贝文件到内部存储器上时不需要请求写入权限。

##获取被拷贝文件的目录
Android Beam文件传输系统会通过一个单一的传输通道将所有的文件拷贝到接收设备上一个目录中。Android Beam文件传输系统的通知会将内容Intent中的URI指向到第一个已被传送的文件上。然而，APP可能还需要对传输系统上的其它ACTION_VIEW意图来源进行接收。为了判断应该如何处理发送过来的Intent，你需要检查它的计划(scheme)与权力(authority)。

为了可以从URI上获取计划，需要调用[Uri.getScheme()](http://android.xsoftlab.net/reference/android/net/Uri.html#getScheme())方法。下面这段代码展示了如何检查计划以及如何处理URI的访问：
```java
public class MainActivity extends Activity {
    ...
    // A File object containing the path to the transferred files
    private File mParentPath;
    // Incoming Intent
    private Intent mIntent;
    ...
    /*
     * Called from onNewIntent() for a SINGLE_TOP Activity
     * or onCreate() for a new Activity. For onNewIntent(),
     * remember to call setIntent() to store the most
     * current Intent
     *
     */
    private void handleViewIntent() {
        ...
        // Get the Intent action
        mIntent = getIntent();
        String action = mIntent.getAction();
        /*
         * For ACTION_VIEW, the Activity is being asked to display data.
         * Get the URI.
         */
        if (TextUtils.equals(action, Intent.ACTION_VIEW)) {
            // Get the URI from the Intent
            Uri beamUri = mIntent.getData();
            /*
             * Test for the type of URI, by getting its scheme value
             */
            if (TextUtils.equals(beamUri.getScheme(), "file")) {
                mParentPath = handleFileUri(beamUri);
            } else if (TextUtils.equals(
                    beamUri.getScheme(), "content")) {
                mParentPath = handleContentUri(beamUri);
            }
        }
        ...
    }
    ...
}
```

###从文件URI获取目录
如果接收到的Intent包含一个文件URI，那么URI会包含文件的绝对文件名，包括全目录路径以及文件的名字。对Android Beam文件传输系统来说，如果有的话，这个目录路径指向了其它被传送文件的位置。为了获取目录路径，获取URI的路径部分，路径部分包含了除了前缀file:之外的所有URI。从路径部分创建一个File对象，然后该文件对象的上级目录：
```java
    ...
    public String handleFileUri(Uri beamUri) {
        // Get the path part of the URI
        String fileName = beamUri.getPath();
        // Create a File object for this filename
        File copiedFile = new File(fileName);
        // Get a string containing the file's parent directory
        return copiedFile.getParent();
    }
    ...
```

###从内容URI获取目录
如果接收到的Intent包含一个内容URI，那这个URI可能会指向在[MediaStore](http://android.xsoftlab.net/reference/android/provider/MediaStore.html)内容提供者中存储的文件的路径和名称。你可以通过测试URI的权限值来判断是否是[MediaStore](http://android.xsoftlab.net/reference/android/provider/MediaStore.html)的内容URI。[MediaStore](http://android.xsoftlab.net/reference/android/provider/MediaStore.html)的内容URI可能来自Android Beam文件传输系统或者其它APP，但是无论什么情况你都可以从内容URI中接收一个目录以及文件的名称。

你也可以接收包含了不同于[MediaStore](http://android.xsoftlab.net/reference/android/provider/MediaStore.html)的内容URI的ACTION_VIEW意图。在这种情况下，这内容URI不包含 ACTION_VIEW的权限值，以及内容URI通常不会执行一个目录。

> **Note:** 对于Android Beam文件传输系统来说，你可以接收含有内容URI的ACTION_VIEW意图，如果第一个接到的文件含有"audio/\*", "image/\*", 或者"video/\*"之类的MIME类型。这意味着它是和媒体有关的。Android Beam文件传输系统会索引媒体文件，它会通过运行媒体扫描器扫描存储有被传送文件的目录来传送文件。媒体扫描器会将扫描结果写入到MediaStore内容提供者中，然后它会传第一个文件的内容URI给Android Beam文件传输系统。这个内容URI是在通知意图中的接收到的那个。给了获取第一个文件的目录，需要使用内容URI从MediaStore中接收它。

###判断内容提供者
为了判断是否可以从内容URI中接收文件的目录，通过调用[Uri.getAuthority()](http://android.xsoftlab.net/reference/android/net/Uri.html#getAuthority())来获取URI的权限来判断与内容提供者相关联的URI。它的值可能有两种情况：

**[MediaStore.AUTHORITY](http://android.xsoftlab.net/reference/android/provider/MediaStore.html#AUTHORITY)**

这个URI是可以被[MediaStore](http://android.xsoftlab.net/reference/android/provider/MediaStore.html)追踪到的单个文件或者多个文件。[MediaStore](http://android.xsoftlab.net/reference/android/provider/MediaStore.html)中可以接收全路径名称，以及可以从文件名中获取它的目录。

**其它任意的权限值**

从其它内容提供者而来的内容URI。展示了与内容URI相关的数据，但是不能够获取文件的目录。

为了可以从[MediaStore](http://android.xsoftlab.net/reference/android/provider/MediaStore.html)内容URI中获取文件目录，需要进行查询，查询需要指定Uri参数为接收到的内容URI，projection参数为列[MediaColumns.DATA](http://android.xsoftlab.net/reference/android/provider/MediaStore.MediaColumns.html#DATA)，它返回的Cursor对象包含了URI所代表的文件的全路径名称。这个路径还包含Android Beam文件传输系统刚刚拷贝到设备上的其它所有文件。

下面这段代码展示了如何测试内容URI的权限以及从被传送的文件中接收路径以及文件名：
```java
    ...
    public String handleContentUri(Uri beamUri) {
        // Position of the filename in the query Cursor
        int filenameIndex;
        // File object for the filename
        File copiedFile;
        // The filename stored in MediaStore
        String fileName;
        // Test the authority of the URI
        if (!TextUtils.equals(beamUri.getAuthority(), MediaStore.AUTHORITY)) {
            /*
             * Handle content URIs for other content providers
             */
        // For a MediaStore content URI
        } else {
            // Get the column that contains the file name
            String[] projection = { MediaStore.MediaColumns.DATA };
            Cursor pathCursor =
                    getContentResolver().query(beamUri, projection,
                    null, null, null);
            // Check for a valid cursor
            if (pathCursor != null &&
                    pathCursor.moveToFirst()) {
                // Get the column index in the Cursor
                filenameIndex = pathCursor.getColumnIndex(
                        MediaStore.MediaColumns.DATA);
                // Get the full file name including path
                fileName = pathCursor.getString(filenameIndex);
                // Create a File object for the filename
                copiedFile = new File(fileName);
                // Return the parent directory of the file
                return new File(copiedFile.getParent());
             } else {
                // The query didn't work; return null
                return null;
             }
        }
    }
    ...
```

如果要学习更多从内容提供者中接收数据的其它知识，请参见章节：[Retrieving Data from the Provider](http://android.xsoftlab.net/guide/topics/providers/content-provider-basics.html#SimpleQuery).