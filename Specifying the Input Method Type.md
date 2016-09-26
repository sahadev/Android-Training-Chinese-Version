原文地址：[http://android.xsoftlab.net/training/keyboard-input/index.html](http://android.xsoftlab.net/training/keyboard-input/index.html)

#引言
文本框接收到焦点时，Android系统会在屏幕上显示一个软键盘。为了提供最佳的用户体验，你可以指定相关输入类型的特性，以及输入法应当如何展现。

除了屏幕上的软键盘之外，Android还支持实体键盘，所以APP如何与各种类型的键盘交互这件事情，就变得很重要了。
#指定输入法的类型
每一个文本框必定只有一种输入类型，比如一个电子邮件地址，一个电话号码或者是常规文本。所以为每一个文本框指定输入类型就变得很重要，这样的话系统才会显示正确的输入法。

你可以指定比如输入方法所提供的拼写建议、首字母大写、以及输入法右下角按钮的行为(**Done**或者**Next**)。这节课主要介绍如何指定这些特性。

##指定键盘类型
你应该总是为文本框声明输入类型，通过[android:inputType](http://android.xsoftlab.net/reference/android/widget/TextView.html#attr_android:inputType)属性可以为文本框添加输入类型。

比如，如果你希望文本框的输入类型为电话号码，可以使用"phone":
```xml
<EditText
    android:id="@+id/phone"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:hint="@string/phone_hint"
    android:inputType="phone" />
```
![](http://android.xsoftlab.net/images/ui/edittext-phone.png)

或者如果文本框主要是用于输入密码的，可以使用"textPassword"隐藏用户的输入文本：
```xml
<EditText
    android:id="@+id/password"
    android:hint="@string/password_hint"
    android:inputType="textPassword"
    ... />    
```
![](http://android.xsoftlab.net/images/training/input/ime_password.png)

[android:inputType](http://android.xsoftlab.net/reference/android/widget/TextView.html#attr_android:inputType)含有多种指定的输入类型，并且一些值可以组合使用。

##开启拼写检查与其它功能
[android:inputType](http://android.xsoftlab.net/reference/android/widget/TextView.html#attr_android:inputType)属性允许你可以为输入类型指定多种行为。更重要的一点是，如果文本框的重点在基础文本输入上(如文本消息)，你应当使用"textAutoCorrect"开启拼写检查。

你还可以为[android:inputType](http://android.xsoftlab.net/reference/android/widget/TextView.html#attr_android:inputType)属性指定多种不同的行为以及输入类型。比如，下面的例子就展示了如何同时开启首字母大写以及拼写检查的功能:
```xml
<EditText
    android:id="@+id/message"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:inputType=
        "textCapSentences|textAutoCorrect"
    ... />
```

##指定输入方法的行为
大多数的输入法都在右下角提供了一个用户功能按钮，这对于当前的文本框来说是极为恰当的。在默认情况下，系统使用这个按钮来实现**Next**或者**Done**功能。除非你的文本框允许多行情况的出现(比如使用了android:inputType="textMultiLine")。在这种情况下，该功能按钮是一个回车按钮。然而，你可以指定一些更加符合你文本框的特别功能，比如**Send**或**Go**。

为了指定键盘的功能按钮，需要使用属性[android:imeOptions](http://android.xsoftlab.net/reference/android/widget/TextView.html#attr_android:imeOptions)，并需要执行比如"actionSend"或"actionSearch"之类的值：
```xml
<EditText
    android:id="@+id/search"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:hint="@string/search_hint"
    android:inputType="text"
    android:imeOptions="actionSend" />
```
![](http://android.xsoftlab.net/images/ui/edittext-actionsend.png)

接下来可以通过[TextView.OnEditorActionListener](http://android.xsoftlab.net/reference/android/widget/TextView.OnEditorActionListener.html)来监听功能按钮的按下事件,并需要在该监听器内响应正确的IME功能ID，该ID定义与[EditorInfo](http://android.xsoftlab.net/reference/android/view/inputmethod/EditorInfo.html)中，比如下面使用的就是[IME_ACTION_SEND](http://android.xsoftlab.net/reference/android/view/inputmethod/EditorInfo.html#IME_ACTION_SEND)：
```java
EditText editText = (EditText) findViewById(R.id.search);
editText.setOnEditorActionListener(new OnEditorActionListener() {
    @Override
    public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
        boolean handled = false;
        if (actionId == EditorInfo.IME_ACTION_SEND) {
            sendMessage();
            handled = true;
        }
        return handled;
    }
});
```

