原文地址：[http://android.xsoftlab.net/training/beam-files/index.html](http://android.xsoftlab.net/training/beam-files/index.html)

#导言
Android允许通过Android Beam文件传输特性在两台设备之间传送文件。这个特性拥有一个简单的API，可以通过设备的接触来启动一个文件传输进程。与此同时，在响应端的Android Beam文件传输系统会自动的将文件从一台设备拷贝到另一台设备上，并且会在拷贝结束时通知用户。

虽然Android Beam文件传输API可以处理大量的数据，但在Android 4.0之后，出现了Android Beam NDEF传输API，它可以用来处理轻量级数据，比如URI，或者其它小型消息。Android Beam是Android NFC框架的唯一特性，它允许从NFC标签中读取NDEF消息。有关学习更多Android Beam的相关信息，请参见话题[Beaming NDEF Messages to Other Devices](http://android.xsoftlab.net/guide/topics/connectivity/nfc/nfc.html#p2p)。有关学习更多NFC框架的相关知识，请参见 [Near Field Communication](http://android.xsoftlab.net/guide/topics/connectivity/nfc/index.html) API指南。


#向其它设备发送文件
这节课展示了如何使得APP通过Android Beam文件传输系统来发送文件到另一台设备。如果要发送文件，首先需要声明权限来标明要使用NFC功能以及外部存储器功能，再者检查设备确保支持NFC，最后需要提供文件的URI给Android Beam文件传输系统以便传输。

Android Beam文件传输特性有以下要求：

- 1.对于大文件的传输只在Android 4.1及以上设备适用。
- 2.要传输的文件必须存储于外部存储器上。有关更多使用外部存储器的相关知识，请参见[Using the External Storage](http://android.xsoftlab.net/guide/topics/data/data-storage.html#filesExternal)。
- 3.每一个被传输的文件必须是全局可读的。你可以通过调用[File.setReadable(true,false)](http://android.xsoftlab.net/reference/java/io/File.html#setReadable(boolean))方法来设置这个权限。
- 4.必须提供传输文件的URI。另Android Beam文件传输系统不可以处理由[FileProvider.getUriForFile](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html#getUriForFile(android.content.Context,%20java.lang.String,%20java.io.File))方法所产生的URI。

##在清单文件中声明权限
首先，需要在APP的清单文件中声明APP所需的权限以及特性。
###请求权限
为了允许用户可以使用NFC功能来发送外部存储器上的文件，你必须在清单文件中添加以下权限：

**[NFC](http://android.xsoftlab.net/reference/android/Manifest.permission.html#NFC)**

允许APP通过NFC来发送数据。为了声明该权限，需要在< manifest>元素下添加如下的标签属性：

```xml
<uses-permission android:name="android.permission.NFC" />
```

**[READ\_EXTERNAL\_STORAGE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)**

允许APP读取外部存储器上的文件。为了声明该权限，需要在< manifest>元素下添加如下标签属性：

```xml
<uses-permission
android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

> **Note:**在Android 4.2.2上，该权限不是强制要求的。未来的平台版本可能需要它。为了确保向后兼容的完整性，在要求之前先添上它。

###指明NFC特性
通过在< manifest>元素中添加< uses-feature>来指声明APP将使用NFC功能。需要设置其android:required属性的值为true的方式来指明APP会在使用NFC功能的时候才会请求该权限。

下面的代码展示了如何指定< uses-feature>元素：
```xml
<uses-feature
android:name="android.hardware.nfc"
android:required="true" />
```

注意，如果APP只是将NFC功能作为一个选项，但是在NFC没有出现的时候还仍然可用，你应该设置android:required为false，并且需要在代码中测试NFC是否可用。


###指明Android Beam文件传输系统
Android Beam文件传输系统只是在Android 4.1及以上的版本可用，如果APP的关键部分使用了Android Beam文件传输系统，你必须在< uses-sdk>元素中指明属性[android:minSdkVersion](http://android.xsoftlab.net/guide/topics/manifest/uses-sdk-element.html#min)="16"。
否则，如果有必要的话，你可以设置[android:minSdkVersion](http://android.xsoftlab.net/guide/topics/manifest/uses-sdk-element.html#min)的值为其它值，并需要在代码中测试当前的平台版本，下节将会描述这部分内容。

##测试是否支持Android Beam文件传输
为了在APP的清单文件中声明NFC功能是可选的，你需要使用以下元素：
```xml
<uses-feature android:name="android.hardware.nfc" android:required="false" />
```

如果设置了[android:required](http://android.xsoftlab.net/guide/topics/manifest/uses-feature-element.html#required)="false"，你必须在代码中测试是否支持NFC，以及是否支持Android Beam文件传输系统。

为了在代码中测试Android Beam文件传输系统是否支持，首先需要通过[PackageManager.hasSystemFeature()](http://android.xsoftlab.net/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String))加参数[FEATURE_NFC](http://android.xsoftlab.net/reference/android/content/pm/PackageManager.html#FEATURE_NFC)来判断设备是否支持NFC。接下来，通过测试[SDK_INT](http://android.xsoftlab.net/reference/android/os/Build.VERSION.html#SDK_INT)的值来判断该平台版本是否支持Android Beam文件传输系统。如果支持的话，则可以获取NFC的控制器实例，该实例允许与NFC硬件设备进行通信：
```java
public class MainActivity extends Activity {
    ...
    NfcAdapter mNfcAdapter;
    // Flag to indicate that Android Beam is available
    boolean mAndroidBeamAvailable  = false;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // NFC isn't available on the device
        if (!PackageManager.hasSystemFeature(PackageManager.FEATURE_NFC)) {
            /*
             * Disable NFC features here.
             * For example, disable menu items or buttons that activate
             * NFC-related features
             */
            ...
        // Android Beam file transfer isn't supported
        } else if (Build.VERSION.SDK_INT <
                Build.VERSION_CODES.JELLY_BEAN_MR1) {
            // If Android Beam isn't available, don't continue.
            mAndroidBeamAvailable = false;
            /*
             * Disable Android Beam file transfer features here.
             */
            ...
        // Android Beam file transfer is available, continue
        } else {
        mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
        ...
        }
    }
    ...
}
```

##对传送的文件创建一个回调方法
一旦验证了设备支持Android Beam文件传输系统，那么需要添加一个回调方法，以便当Android Beam文件传输系统检查出用户想要发送文件给另一台NFC设备的时候系统可以调用这个方法。在这个回调方法中，需要返回一组[Uri](http://android.xsoftlab.net/reference/android/net/Uri.html)对象。Android Beam文件传输系统则会拷贝由这些[Uri](http://android.xsoftlab.net/reference/android/net/Uri.html)所指向的文件到接收端设备上。

为了添加这个回调方法，需要实现[NfcAdapter.CreateBeamUrisCallback](http://android.xsoftlab.net/reference/android/nfc/NfcAdapter.CreateBeamUrisCallback.html)接口以及其中的方法[createBeamUris()](http://android.xsoftlab.net/reference/android/nfc/NfcAdapter.CreateBeamUrisCallback.html#createBeamUris(android.nfc.NfcEvent))。下面这段代码展示了如何实现：
```java
public class MainActivity extends Activity {
    ...
    // List of URIs to provide to Android Beam
    private Uri[] mFileUris = new Uri[10];
    ...
    /**
     * Callback that Android Beam file transfer calls to get
     * files to share
     */
    private class FileUriCallback implements
            NfcAdapter.CreateBeamUrisCallback {
        public FileUriCallback() {
        }
        /**
         * Create content URIs as needed to share with another device
         */
        @Override
        public Uri[] createBeamUris(NfcEvent event) {
            return mFileUris;
        }
    }
    ...
}
```

一旦你实现了该接口，通过调用[setBeamPushUrisCallback()](http://android.xsoftlab.net/reference/android/nfc/NfcAdapter.html#setBeamPushUrisCallback(android.nfc.NfcAdapter.CreateBeamUrisCallback,%20android.app.Activity))方法将该回调提供给Android Beam文件传输系统。下面这段代码展示了如何完成：
```java
public class MainActivity extends Activity {
    ...
    // Instance that returns available files from this app
    private FileUriCallback mFileUriCallback;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Android Beam file transfer is available, continue
        ...
        mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
        /*
         * Instantiate a new FileUriCallback to handle requests for
         * URIs
         */
        mFileUriCallback = new FileUriCallback();
        // Set the dynamic callback for URI requests.
        mNfcAdapter.setBeamPushUrisCallback(mFileUriCallback,this);
        ...
    }
    ...
}
```

> **Note:**你也可以通过APP的[NfcAdapter](http://android.xsoftlab.net/reference/android/nfc/NfcAdapter.html)实例直接给NFC框架提供Uri数组。如果在NFC靠近事件发生之前你可以对传输系统定义URI的话，就可以选择这种方式。有关学习更多关于这种方式的相关知识，请参见：[NfcAdapter.setBeamPushUris()](http://android.xsoftlab.net/reference/android/nfc/NfcAdapter.html#setBeamPushUris(android.net.Uri[],%20android.app.Activity))。

##指明发送的文件
为了给其它NFC设备传输文件，需要获取每个文件的URI地址，然后将地址添加到Uri对象数组中去。如果要传输单个文件，还必须拥有文件的永久性读取权限。举个例子，下面这段代码展示了如何根据文件名获取文件的URI地址，并将该URI添加到数组中：
```java
        /*
         * Create a list of URIs, get a File,
         * and set its permissions
         */
        private Uri[] mFileUris = new Uri[10];
        String transferFile = "transferimage.jpg";
        File extDir = getExternalFilesDir(null);
        File requestFile = new File(extDir, transferFile);
        requestFile.setReadable(true, false);
        // Get a URI for the File and add it to the list of URIs
        fileUri = Uri.fromFile(requestFile);
        if (fileUri != null) {
            mFileUris[0] = fileUri;
        } else {
            Log.e("My Activity", "No File URI available for file.");
        }
```
