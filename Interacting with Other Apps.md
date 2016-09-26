原文地址：[http://android.xsoftlab.net/training/basics/intents/index.html](http://android.xsoftlab.net/training/basics/intents/index.html)

#导言
一个Android APP应用通常会有若干个Activity。每一个Activity所展示的用户界面用于允许用户执行特定的任务(比如浏览地图或者是拍照)。为了把用户从一个activity带到另一个activity，APP必须使用一个Intent对象定义APP的意图是想要做什么事情，系统会使用这个Intent对象来识别并启动合适的APP组件。使用意图事件允许你的APP启动另一个程序中的Activity。

一个Intent可以明确的启动一个具体的组件(一个具体的Activity实例)或者模糊的启动任何组件，不过这个组件需要有能力处理预期的行为(比如拍照)>

这节课展示了如何使用Intent与其它APP执行一些基础交互，比如启动另一个APP，从另一个APP接收结果，或者使你的APP可以响应其它APP的请求。

#将用户带到另一个Activity
Android主要特征之一就是一个APP有能力带领用户到另一个APP。举个例子，如果你的APP有一条商业地址，并且你想在地图上将这个地址展示出来，你不需要不得不在APP内创建一个可以展示地图的Activity。相反，你可以使用Intent创建一个请求，来请求展示这个地址。Android系统然后会启动一个有能力在地图上展示地址的APP。

就像第一节课所描述的，[Building Your First App](http://android.xsoftlab.net/training/basics/firstapp/index.html)，你必须使用Intent来引导APP中的两个activity。你通常对这样的情况使用显式意图，意图中明确定义了你想启动的组件的类名。然而，当你想要通过其它不知类名的APP执行这个行为的时候，比如查看地图，这时，你就必须使用隐式意图了。

这节课展示了如何对特定的行为创建隐式意图，以及如何使用它来启动另一个APP中可以执行这个行为的Activity。

##构建一个隐式意图
隐式意图不会声明将要启动的组件的类名，不过相反的，它会声明要执行的行为。这个行为指明了你将要做的事情，比如查看、编辑、发送或者获取一些东西。Intent经常还会包含一些与行为有关的数据，比如你想浏览的地址，或者你想发送的email消息。依赖于你想创建的intent，这里的数据可能是一个[Uri](http://android.xsoftlab.net/reference/android/net/Uri.html),一种其它的数据类型，或者intent一点数据都不需要。

如果你的数据是一个[Uri](http://android.xsoftlab.net/reference/android/net/Uri.html)，这里有一个简单的Intent类的构造方法，你可以使用这个构造行为和数据。

举个例子，这里展示了如何使用[Uri](http://android.xsoftlab.net/reference/android/net/Uri.html)创建一个Intent来拨打电话，并且在[Uri](http://android.xsoftlab.net/reference/android/net/Uri.html)中指明了电话号码：
```java
Uri number = Uri.parse("tel:5551234");
Intent callIntent = new Intent(Intent.ACTION_DIAL, number);
```

当你的APP通过[startActivity()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivity(android.content.Intent))调用了这个intent，电话APP会拨打刚才给定的电话号码。

这里有一些其他意图以及它们的行为和[Uri](http://android.xsoftlab.net/reference/android/net/Uri.html)数据对：

查看地图：
```java
/ Map point based on address
Uri location = Uri.parse("geo:0,0?q=1600+Amphitheatre+Parkway,+Mountain+View,+California");
// Or map point based on latitude/longitude
// Uri location = Uri.parse("geo:37.422219,-122.08364?z=14"); // z param is zoom level
Intent mapIntent = new Intent(Intent.ACTION_VIEW, location);
```

浏览一个Web页面：
```java
Uri webpage = Uri.parse("http://www.android.com");
Intent webIntent = new Intent(Intent.ACTION_VIEW, webpage);
```

其它的隐式意图要求附加数据，附加数据提供了不同的数据类型，比如一个字符串，你可以各种的[putExtra()](http://android.xsoftlab.net/reference/android/content/Intent.html#putExtra(java.lang.String,%20java.lang.String))方法添加一个或者多个额外的数据。

默认情况下，系统确定要求[Uri](http://android.xsoftlab.net/reference/android/net/Uri.html)中包含合适的MIME类型，如果没有在intent中包含[Uri](http://android.xsoftlab.net/reference/android/net/Uri.html)，你应该使用[setType()](http://android.xsoftlab.net/reference/android/content/Intent.html#setType(java.lang.String))方法指定与intent关联的数据类型。设置了MIME类型更进一步的指明了哪一种类型的activity应该接收这个intent。

这里有一些添加了附加数据来指定期望行为的intent：

使用附加数据发送email:
```java
Intent emailIntent = new Intent(Intent.ACTION_SEND);
// The intent does not have a URI, so declare the "text/plain" MIME type
emailIntent.setType(HTTP.PLAIN_TEXT_TYPE);
emailIntent.putExtra(Intent.EXTRA_EMAIL, new String[] {"jon@example.com"}); // recipients
emailIntent.putExtra(Intent.EXTRA_SUBJECT, "Email subject");
emailIntent.putExtra(Intent.EXTRA_TEXT, "Email message text");
emailIntent.putExtra(Intent.EXTRA_STREAM, Uri.parse("content://path/to/email/attachment"));
// You can also attach multiple items by passing an ArrayList of Uris
```

创建一个日历事件：
```java
Intent calendarIntent = new Intent(Intent.ACTION_INSERT, Events.CONTENT_URI);
Calendar beginTime = Calendar.getInstance().set(2012, 0, 19, 7, 30);
Calendar endTime = Calendar.getInstance().set(2012, 0, 19, 10, 30);
calendarIntent.putExtra(CalendarContract.EXTRA_EVENT_BEGIN_TIME, beginTime.getTimeInMillis());
calendarIntent.putExtra(CalendarContract.EXTRA_EVENT_END_TIME, endTime.getTimeInMillis());
calendarIntent.putExtra(Events.TITLE, "Ninja class");
calendarIntent.putExtra(Events.EVENT_LOCATION, "Secret dojo");
```
> **Note:**这里的日历事件仅仅支持API 14及以上的版本

> **Note:**尽可能的明确Intent是非常重要的。举个例子，如果你希望使用[ACTION_VIEW](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_VIEW)意图来展示一张图片，你应该指定MIME类型为'image/*'。这可以预防由intent引起的APP查看其它类型的数据(比如地图APP)。


##确认有APP可以接收Intent
尽管Android平台保证intent可以被内置的应用(比如电话、邮件或者日历)解析，但是你还是应该在启动一个intent之前执行验证这一步。

>**警告：**如果你启动了一个intent，但是没有APP可以处理该intent，你的APP就会崩溃。

为了验证系统中有Activity可以响应Intent的请求，需要调用[queryIntentActivities()](http://android.xsoftlab.net/reference/android/content/pm/PackageManager.html#queryIntentActivities(android.content.Intent,%20int))方法来获取有能力处理Intent的activity列表。如果返回的列表不是空的，那么你就可以安全的使用intent了：
```java
PackageManager packageManager = getPackageManager();
List activities = packageManager.queryIntentActivities(intent,
        PackageManager.MATCH_DEFAULT_ONLY);
boolean isIntentSafe = activities.size() > 0;
```

如果isIntentSafe的值为true，那么至少有一个APP可以响应intent。如果为false，那么没有任何APP可以处理intent。

>**Note：**当在activity第一次启动的情况下，你应该执行这项检查。如果你知道指定的APP可以处理这个Intent，你也可以提供一个连接以供用户去下载这个APP(有关请看[link to your product on Google Play](http://android.xsoftlab.net/distribute/tools/promote/linking.html)).

##使用Intent启动一个Activity
![](http://android.xsoftlab.net/images/training/basics/intents-choice.png)

**上图中：**当多个APP可以处理intent的时候，选择对话框会显示出来。

曾经你创建了Intent，并且设置了附加信息，调用[startActivity()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivity(android.content.Intent))将intent发送给系统。如果系统识别了多个activity可以处理这个intent，它会显示一个对话框来让用户选择要使用哪个APP，就像上图所示。如果只有一个activity可以处理，那么系统会立即启动它。
```java
startActivity(intent);
```

这里有个完整的例子来展示如何创建一个intent来浏览地图，验证存在可以处理intent的APP，然后启动它：
```java
// Build the intent
Uri location = Uri.parse("geo:0,0?q=1600+Amphitheatre+Parkway,+Mountain+View,+California");
Intent mapIntent = new Intent(Intent.ACTION_VIEW, location);
// Verify it resolves
PackageManager packageManager = getPackageManager();
List<ResolveInfo> activities = packageManager.queryIntentActivities(mapIntent, 0);
boolean isIntentSafe = activities.size() > 0;
// Start an activity if it's safe
if (isIntentSafe) {
    startActivity(mapIntent);
}
```

##显示一个APP选择器
![](http://android.xsoftlab.net/images/training/basics/intent-chooser.png)

**上图中：**一个选择器对话框。

注意当你在启动Activity的时候有多个APP可以响应Intent，那么用户可以选择默认情况下要启动哪个APP。这非常好，在每一次要执行相同动作的时候，比如打开一个Web页面(用户可能只喜欢一个web浏览器)或者拍照。

然而，在这种情况下，可能用户需要每次都需要选择不同的APP，比如分享这个行为，这种情况下用户可能要分享到多个APP上，你应该明确显示一个选择器对话框，这个选择器对话框会每次出现在用户选择的时候(这种情况下没有默认选择选项)。

如果要显示选择器对话框，需要使用[createChooser()](http://android.xsoftlab.net/reference/android/content/Intent.html#createChooser(android.content.Intent,%20java.lang.CharSequence))方法创建一个[Intent](http://android.xsoftlab.net/reference/android/content/Intent.html)对象，然后传给[startActivity()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivity(android.content.Intent))：
```java
Intent intent = new Intent(Intent.ACTION_SEND);
...
// Always use string resources for UI text.
// This says something like "Share this photo with"
String title = getResources().getString(R.string.chooser_title);
// Create intent to show chooser
Intent chooser = Intent.createChooser(intent, title);
// Verify the intent will resolve to at least one activity
if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}
```

这样就会显示一个选择器对话框了，这个对话框会像上图显示的那样，将符合条件的APP显示出来，并且支持自定义的标题文本。