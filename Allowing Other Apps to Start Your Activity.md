原文地址：[http://android.xsoftlab.net/training/basics/intents/filters.html](http://android.xsoftlab.net/training/basics/intents/filters.html)

在前两节课程中我们只关注了事情的一面：从你的APP启动其它APP。但是如果你的APP可以执行一些功能，并且这些功能可以被其它APP所利用，那么你可以做一个功能来响应其它APP的请求。举个例子，如果你构建了一个社交APP并且可以给用户的朋友分享消息或者照片，那么这是支持[ACTION_SEND](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_SEND)意图的最佳兴趣点，所以用户可以从其它APP中启动一个"share"行为，然后启动你的APP来完成这个功能。

为了允许其它APP可以启动你的Activity，你需要添加一个[< intent-filter>](http://android.xsoftlab.net/guide/topics/manifest/intent-filter-element.html)元素标签到清单文件中的对应的[< activity>](http://android.xsoftlab.net/guide/topics/manifest/activity-element.html)标签元素下。

当APP被安装到设备上之后，系统会识别你的意图过滤器，并且添加该信息到所有支持的内部意图目录中。当APP使用了隐式意图调用了[startActivity()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivity(android.content.Intent))或[startActivityForResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivityForResult(android.content.Intent,%20int))，那么系统会寻找那些activity可以响应这个意图。

##添加意图过滤器
为了可以适当的定义Activity可以处理哪一种意图，你添加每一个意图过滤器应该尽可能的指明activity可以接受的行为类型和数据类型。

系统可能通过给定的Intent发送到activity，如果这个activity有一个意图过滤器正好可以完全匹配以下的Intent对象标准：
 
**Action:**
这个行为可以执行的名称。通常平台上定义的值比如是[ACTION_SEND](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_SEND)或[ACTION_VIEW](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_VIEW)。

在意图过滤器中使用[< action>](http://android.xsoftlab.net/guide/topics/manifest/action-element.html)元素标签来指定该值。该值必须是这个行为的全称，而不是API常量(请看下面的例子)。

**Data:**
有关这个意图的数据描述。

在意图过滤器中使用[< data>](http://android.xsoftlab.net/guide/topics/manifest/data-element.html)元素标签指定该值。使用该元素的更多属性，你可以仅仅指定MIME类型，URI前缀类型，URI计划类型，或者这些类型的组合，以及可以接收的其它数据类型。

>**Note:**如果你不需要声明Uri数据的指定类型(比如你的activity处理其它种类的附加数据，而不是URI)，你应该通过android:mimeType属性声明activity可以处理的数据类型，比如text/plain或者image/jpeg。

**Category:**
提供了一种附加方式来描述activity处理intent，通常与启动的用户手势或者位置有关。这里有几个系统支持的不同的分类，但是大多数是很少使用的。然而，所有的隐式意图会在默认情况下使用[CATEGORY_DEFAULT](http://android.xsoftlab.net/reference/android/content/Intent.html#CATEGORY_DEFAULT)定义。

可以在意图过滤器中使用[< category>](http://android.xsoftlab.net/guide/topics/manifest/category-element.html)元素标签指定该值。

在意图过滤器中，你可以通过在意图过滤器中使用相应的XML元素来声明activity可以接收的意图标准。

举个例子，这里的activity可以处理[ACTION_SEND](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_SEND)意图，当数据类型为文本或者图像时。
```java
<activity android:name="ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
        <data android:mimeType="image/*"/>
    </intent-filter>
</activity>
```

每个到来的Intent都只会指定一个行为和一个数据类型，但是在每一个过滤器中声明多个< action>,< category>,和< data>元素的话，这就没问题了。

如果任意的两对行为和数据会在他们的行为中相互排斥的话，你应该创建单独的意图过滤器来指明哪个行为可以接收，当也匹配到相应的数据类型时。

举个例子，假设activity在[ACTION_SEND](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_SEND)行为下或者[ACTION_SENDTO](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_SENDTO)行为下都可以处理文本和图像。在这种情况下，你必须定义两个单独的意图过滤器，因为[ACTION_SENDTO](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_SENDTO)意图必须使用数据Uri指定接收地址通过使用send或sendto URI计划：
```xml
<activity android:name="ShareActivity">
    <!-- filter for sending text; accepts SENDTO action with sms URI schemes -->
    <intent-filter>
        <action android:name="android.intent.action.SENDTO"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:scheme="sms" />
        <data android:scheme="smsto" />
    </intent-filter>
    <!-- filter for sending text or images; accepts SEND action and text or image data -->
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="image/*"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
```

>**Note:**为了可以接收隐式意图，你必须在过滤器中包含类别[CATEGORY_DEFAULT](http://android.xsoftlab.net/reference/android/content/Intent.html#CATEGORY_DEFAULT)。startActivity()和startActivityForResult()方法会对待所有的Intent，仿佛他们都声明了[CATEGORY_DEFAULT](http://android.xsoftlab.net/reference/android/content/Intent.html#CATEGORY_DEFAULT)类别。如果你没有在意图过滤器中声明它，那么将不会有隐式意图传送到该activity中。

有关更多关于发送和接收[ACTION_SEND](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_SEND)意图来执行社会分享行为的相关信息，请参见课程： [Receiving Simple Data from Other Apps](http://android.xsoftlab.net/training/sharing/receive.html).

##在Activity中处理意图
为了决定要在activity中采取什么行为，你可以从启动activity的Intent中读取信息。

随着activity被启动，你可以调用[getIntent()](http://android.xsoftlab.net/reference/android/app/Activity.html#getIntent())方法获得启动该Activity的Intent。你可以在activity的任意生命周期内做这样的事情，但是你通常应该在较早的回调方法中做这些事情，比如onCreate()或onStart()。
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);
    // Get the intent that started this activity
    Intent intent = getIntent();
    Uri data = intent.getData();
    // Figure out what to do based on the intent type
    if (intent.getType().indexOf("image/") != -1) {
        // Handle intents with image data ...
    } else if (intent.getType().equals("text/plain")) {
        // Handle intents with text ...
    }
}
```

##返回结果
如果需要将结果返回给调用该activity的那个activity，那么只需要简单的调用[setResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#setResult(int,%20android.content.Intent))方法然后设置一个结果码以及结果Intent。当你的操作完成并且用户应该返回原来的那个activity是，调用[finish()](finish())方法来关闭你的activity，像下面这样：
```java
// Create intent to deliver some kind of result data
Intent result = new Intent("com.example.RESULT_ACTION", Uri.parse("content://result_uri");
setResult(Activity.RESULT_OK, result);
finish();
```

你需要每一次都给结果指定一个结果码。通常情况下，要不然是[RESULT_OK](http://android.xsoftlab.net/reference/android/app/Activity.html#RESULT_OK)或者是[RESULT_CANCELED](http://android.xsoftlab.net/reference/android/app/Activity.html#RESULT_CANCELED)。必要的话，可以在Intent中提供附加数据。

> **Note:**默认情况下，结果会设置[RESULT_CANCELED](http://android.xsoftlab.net/reference/android/app/Activity.html#RESULT_CANCELED)。所以，如果用户在完成功能之前或者在你设置结果之前按下了返回按钮，那么原来的activity会接收到"canceled"结果。

如果你简单的需要返回一个整型，这个整型指明了若干个结果选项，那么你可以设置结果码为比0大的任何值。如果你使用了结果码来传递一个整型值，那么你不再需要包含Intent对象，你可以调用[setResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#setResult(int))方法，然后只传一个结果码进去：
```java
setResult(RESULT_COLOR_RED);
finish();
```

在这个例子中，这可能对结果有损伤，所以结果码是一个本地变量。这种情况在你自己的APP中运行的话，效果会很好，因为原来的activity可以接收到结果码，然后引用一个公开的常量来判断结果码的值。

>**Note:**这里不需要检查你的activity是否是通过startActivity()或者startActivityForResult()启动的。那个启动你activity的activity可能希望会有个结果返回。如果原来的activity调用了[startActivityForResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivityForResult(android.content.Intent,%20int))，那么系统会传递[setResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#setResult(int))结果，否则，这个结果会被忽略。
