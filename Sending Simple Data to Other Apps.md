原文地址：[http://android.xsoftlab.net/training/building-content-sharing.html](http://android.xsoftlab.net/training/building-content-sharing.html)

#引言
Android应用程序有一项伟大的事情就是它们有可以与其它应用程序交流及整合。为什么不重新使用已经存在于其它APP中的非核心功能呢？

这节课覆盖了一些共同的方式，你可以使用这些方式在两个程序之间使用[Intent](http://android.xsoftlab.net/reference/android/content/Intent.html)API以及[ActionProvider](http://android.xsoftlab.net/reference/android/view/ActionProvider.html)对象发送和接收一些简单的数据。

##发送简单的数据给其它APP
当在构造Intent时，必须指定intent要触发的功能。Android定义了包括ACTION_SEND在内的若干功能。你可以猜到，ACTION_SEND表明这个intent可以发送数据从一个activity到另一个activity，甚至是跨进程。如果要发送数据到另一个activity，你需要做的就是指定数据与类型，系统会识别适合接收的activity列表并展示给用户选择，如果有多个的话，或者立即启动activity。相似的，你可以公布你的activity支持接收的数据类型。

在两个应用之间发送接收数据在社会化分享中非常常见。Intent使用户可以更快捷更方便的使用他们喜欢的应用分享信息。

**Note:**在ActionBar上添加分享按钮的最好方式是使用[ShareActionProvider](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html)，ShareActionProvider从API 14开始可用。我们会在课程[Adding an Easy Share Action](http://android.xsoftlab.net/training/sharing/shareaction.html)中讨论ShareActionProvider。

##发送文本内容
![](http://android.xsoftlab.net/images/training/sharing/share-text-screenshot.png)

**上图：**在手持设备上ACTION_SEND意图选择器的对话框。

ACTION_SEND的大多数功能是发送文本从一个activity到另一个activity。举个例子，系统内置的浏览器可以将当前页面的URL作为文本分享给任何程序。这对通过email或者社交网络分享一篇文章或者一个网站给朋友来说是非常有用的。这里的代码实现了这种类型的分享：
```java
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, "This is my text to send.");
sendIntent.setType("text/plain");
startActivity(sendIntent);
```

如果有应用程序的过滤器可以匹配到ACTION_SEND以及MIME类型text/plain，那么Android系统会运行它；如果有多个应用程序匹配到，系统会展示一个选择对话框，来允许用户选在一个APP。

然而，如果你调用的是Intent.createChooser()，那么它返回的Intent版本将总是会展示一个选择器对话框。这里是它的一些优势：

- 虽然用户原先已经选择过这个Intent的默认应用，但是对话框还是需要每次都出现。
- 如果没有程序匹配到，那么Android系统会展示一条系统消息。
- 你可以指定选择对话框的标题。

这里升级后的代码：
```java
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, "This is my text to send.");
sendIntent.setType("text/plain");
startActivity(Intent.createChooser(sendIntent, getResources().getText(R.string.send_to)));
```

它的结果会向上图显示的那样。

你可以给Intent设置一些附加标准：[EXTRA_EMAIL](http://android.xsoftlab.net/reference/android/content/Intent.html#EXTRA_EMAIL), EXTRA_CC, EXTRA_BCC, EXTRA_SUBJECT。如果接收的应用程序不是被设计为使用它们的话，程序会忽略这些附加标准。

> **Note:**一些e-mail的应用程序，比如Gmail，会期望接收附加的[字符串数组](http://android.xsoftlab.net/reference/java/lang/String.html)，类似EXTRA_EMAIL和EXTRA_CC,使用[putExtra(String, String[])](putExtra(String, String[]))方法来将这些信息添加到Intent。

##发送二进制内容
分享二进制内容需要通过ACTION_SEND行为结合合适的MIME类型然后将数据放入到URI以 EXTRA_STREAM命名的附加值中。下面是分享一张图片的例子，不过，它适用于分享任何类型的二进制内容：
```java
Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND);
shareIntent.putExtra(Intent.EXTRA_STREAM, uriToImage);
shareIntent.setType("image/jpeg");
startActivity(Intent.createChooser(shareIntent, getResources().getText(R.string.send_to)));
```

注意以下事项：

- 你可以使用"\*/\*"的MIME类型，但是这只是会匹配有能力处理通用数据流的Activity。
- 匹配到的应用程序需要有权限来访问Uri所指向的资源。下面是推荐的方式：
 - 将数据存储到你自己的[ContentProvider](http://android.xsoftlab.net/reference/android/content/ContentProvider.html)中，确保其他APP有正确的权限访问你的提供者。提供访问的首选机制是使用[per-URI permissions](http://android.xsoftlab.net/guide/topics/security/permissions.html#uri)，它是一个临时的只授权接收到的应用程序访问的权限。可以像使用[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)帮助类那样简单的创建一个[ContentProvider](http://android.xsoftlab.net/reference/android/content/ContentProvider.html)。
 - 使用系统的[MediaStore](http://android.xsoftlab.net/reference/android/provider/MediaStore.html)，MediaStore会首先瞄准视频，音频，以及图像MIME类型，然而从Android 3.0之后，它还可以存储非媒体类型。文件可以通过scanFile()被插入到MediaStore之后，scanFile()所提供的[onScanCompleted()](http://android.xsoftlab.net/reference/android/media/MediaScannerConnection.OnScanCompletedListener.html#onScanCompleted(java.lang.String,%20android.net.Uri))回调方法会传递一个适用于分享的content://风格的Uri。注意，一旦将内容被添加到MediaStore中，那么设备上的任何APP都可以访问它。

##发送多个内容片段
如果要分享内容的多个片段的话，使用ACTION_SEND_MULTIPLE行为可以将Uri分别指向的内容整合成为一个列表。MIME类型取决于你分享的内容。举个例子，如果要分享3张JPEG图片，使用的类型仍然是"image/jpeg"。如果混合了多个类型的话，应该使用"image/*"来匹配一个可以处理任何类型的Activity。如果你分享出一个类型很多样的内容的话，你应该使用"\*/\*"。就像原先陈述的，这取决于接收的应用程序解析并处理你的数据：
```java
ArrayList<Uri> imageUris = new ArrayList<Uri>();
imageUris.add(imageUri1); // Add your image URIs here
imageUris.add(imageUri2);
Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, imageUris);
shareIntent.setType("image/*");
startActivity(Intent.createChooser(shareIntent, "Share images to.."));
```

和以前需要注意的一样，请确保提供的URI所指向的数据，那些接收的应用程序可以访问。