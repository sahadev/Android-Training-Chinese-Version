原文地址：[http://android.xsoftlab.net/training/sharing/shareaction.html](http://android.xsoftlab.net/training/sharing/shareaction.html)

从Android 4.0开始，使用[ActionProvider](http://android.xsoftlab.net/reference/android/view/ActionProvider.html)可以更方便的在ActionBar上实现一个有效的、用户友好的分享按钮。一个[ActionProvider](http://android.xsoftlab.net/reference/android/view/ActionProvider.html)一旦依附到了ActionBar的菜单条目上，它会处理这个菜单条目的外观和行为。在[ShareActionProvider](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html)这种情况中，你只需提供一个分享意图，它会处理剩下的事情。

> **Note:**[ShareActionProvider](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html)从API 14开始可用。

![](http://android.xsoftlab.net/images/ui/actionbar-shareaction.png)

**上图：**[ShareActionProvider](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html)在相册APP中的应用。

##更新菜单声明
如果要开始使用[ShareActionProvider](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html)，需要在[菜单资源](http://android.xsoftlab.net/guide/topics/resources/menu-resource.html)文件中相应的< item>中定义android:actionProviderClass属性：
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
            android:id="@+id/menu_item_share"
            android:showAsAction="ifRoom"
            android:title="Share"
            android:actionProviderClass=
                "android.widget.ShareActionProvider" />
    ...
</menu>
```

这样就定义了[ShareActionProvider](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html)对菜单外观与行为的代理职责。不过，你还是需要告诉提供者Provider你要分享的东西。

##设置分享意图
为了可以使[ShareActionProvider](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html)运行，你必须给它提供一个分享意图。这个分享意图应该与课程[Sending Simple Data to Other Apps](http://android.xsoftlab.net/training/sharing/send.html)中描述的一致，需要有行为[ACTION_SEND](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_SEND)以及附加数据集比如EXTRA_TEXT或EXTRA_STREAM。为了分配一个共享意图，首先要找到在Activity或者Fragment中填充的对应的菜单条目[MenuItem](http://android.xsoftlab.net/reference/android/view/MenuItem.html)。接下来，调用[MenuItem.getActionProvider()](http://android.xsoftlab.net/reference/android/view/MenuItem.html#getActionProvider())方法接收一个[ShareActionProvider](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html)的实例。使用[setShareIntent()](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html#setShareIntent(android.content.Intent))来更新与共享意图相关的那个行为条目：
```java
private ShareActionProvider mShareActionProvider;
...
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate menu resource file.
    getMenuInflater().inflate(R.menu.share_menu, menu);
    // Locate MenuItem with ShareActionProvider
    MenuItem item = menu.findItem(R.id.menu_item_share);
    // Fetch and store ShareActionProvider
    mShareActionProvider = (ShareActionProvider) item.getActionProvider();
    // Return true to display menu
    return true;
}
// Call to update the share intent
private void setShareIntent(Intent shareIntent) {
    if (mShareActionProvider != null) {
        mShareActionProvider.setShareIntent(shareIntent);
    }
}
```

在创建菜单期间，你可能只需要设置一次分享意图，或者你可能想在UI改变的时候通过设置它来更新它。举个例子，当你在全屏状态下浏览照片的时候，分享意图会随着照片的滑动而改变。

有关[ShareActionProvider](http://android.xsoftlab.net/reference/android/widget/ShareActionProvider.html)的进一步讨论，请参见[Action Bar](http://android.xsoftlab.net/guide/topics/ui/actionbar.html#ActionProvider)指南。