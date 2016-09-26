原文地址：[http://android.xsoftlab.net/training/sharing/receive.html](http://android.xsoftlab.net/training/sharing/receive.html)

正如你的程序可以发送数据给其它程序，那么你也可以轻松的接收数据。想象一下用户如何与你的程序交互，以及你想从其它应用程序接收的数据类型。举个例子，一个社交网络的程序可能对文本内容更感兴趣，比如一个有意思的Web地址，Google+ APP允许接收文本、单张图片或者多张图片。通过这个APP，用户可以很轻松的从Android图库APP启动一个Google+的发送照片任务。

##更新你的清单文件
意图过滤器告知系统可以接收什么样的意图。举个例子，如果应用程序可以接收并处理文本内容，和任何类型的一张图片，或者任何类型的多张图片，你的清单文件应该声明成这样：
```xml
<activity android:name=".ui.MyActivity" >
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND_MULTIPLE" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
```
>**Note:**有关更多意图过滤器的信息以及意图的分辨，请阅读： [Intents and Intent Filters](http://android.xsoftlab.net/guide/components/intents-filters.html#ifs)

当另一个程序通过构造一个意图并且传递给startActivity()并尝试分享这几种类型的任意一种时，你的程序会在意图选择器中列出各种选项。如果用户选择了你的程序，那么相应的activity会被启动。然后由你的代码和UI来妥善的处理这些内容。

##处理到来的内容
为了处理由Intent传递过来的内容，开始调用[getIntent()](http://android.xsoftlab.net/reference/android/content/Intent.html#getIntent(java.lang.String))方法获得Intent对象。一旦你获得了这个对象，你可以检查其中的内容然后在决定接下来怎么做。记住一点，如果这个activity可以由系统的其它部分启动，比如系统的桌面，然后你需要在执行检查的时候将这种情况考虑在内。
```java
void onCreate (Bundle savedInstanceState) {
    ...
    // Get intent, action and MIME type
    Intent intent = getIntent();
    String action = intent.getAction();
    String type = intent.getType();
    if (Intent.ACTION_SEND.equals(action) && type != null) {
        if ("text/plain".equals(type)) {
            handleSendText(intent); // Handle text being sent
        } else if (type.startsWith("image/")) {
            handleSendImage(intent); // Handle single image being sent
        }
    } else if (Intent.ACTION_SEND_MULTIPLE.equals(action) && type != null) {
        if (type.startsWith("image/")) {
            handleSendMultipleImages(intent); // Handle multiple images being sent
        }
    } else {
        // Handle other intents, such as being started from the home screen
    }
    ...
}
void handleSendText(Intent intent) {
    String sharedText = intent.getStringExtra(Intent.EXTRA_TEXT);
    if (sharedText != null) {
        // Update UI to reflect text being shared
    }
}
void handleSendImage(Intent intent) {
    Uri imageUri = (Uri) intent.getParcelableExtra(Intent.EXTRA_STREAM);
    if (imageUri != null) {
        // Update UI to reflect image being shared
    }
}
void handleSendMultipleImages(Intent intent) {
    ArrayList<Uri> imageUris = intent.getParcelableArrayListExtra(Intent.EXTRA_STREAM);
    if (imageUris != null) {
        // Update UI to reflect multiple images being shared
    }
}
```

> **Caution:**检查到来的附加数据要格外当心，你永远不会知道其它应用程序会发来什么。举个例子，比如可能设置了错误的MIME类型，或者发送来的图像可能非常的大。还要记得要在单独的线程中处理二进制数据，而不是主线程。

更新UI要尽可能的简单的填充EditText，否则它可能会更加复杂，就像将一个有趣的图片过滤到一个图像。这需要指定程序接下来会发生什么。(翻译的不好)