原文地址：[http://android.xsoftlab.net/training/basics/data-storage/index.html](http://android.xsoftlab.net/training/basics/data-storage/index.html)

#引言
大多数的安卓APP需要保存数据，即使仅仅存储在onPause状态下的信息，这样的话，用户的进度信息就不会被丢失。大多数重量级的APP还需要保存用户的设置信息，还有一些APP必须管理在文件或者数据库中存储的大量信息。这节课会介绍Android中的数据的主要存储方式，包括以下几点：

- 在共享参数文件中存储简单的键值对信息
- 在Android文件系统中存储任意的文件
- 使用SQLite数据库管理系统

#存储键值对序列
原文地址：[http://android.xsoftlab.net/training/basics/data-storage/shared-preferences.html](http://android.xsoftlab.net/training/basics/data-storage/shared-preferences.html)

如果你有个相对简单的键值对序列需要保存，你应该使用SharedPreferences API。一个SharedPreferences指向的是一个文件，这个文件包含了键值对。并且SharedPreferences提供了简单的方法可以读取或者写入数据。每一个SharedPreferences文件都是被Framework框架所管理的，它可以是私有的或者是公开的。

这节课向你展示了如何使用SharedPreferences API来存储和接收简单的值。

> **Note:**SharedPreferences API仅仅可以用来读取和写入键值对，你不应该将它与Preference API搞混，Preference API可以帮助你构建APP设置的用户界面(尽管Preference内部使用的是SharedPreferences来保存APP的设置信息的)。更多有关使用[Preference](http://android.xsoftlab.net/reference/android/preference/Preference.html) API的相关信息，请参见[Settings](http://android.xsoftlab.net/guide/topics/ui/settings.html)向导。


##获取SharedPreferences的句柄
你可以创建一个共享参数文件或者访问一个已经存在的共享参数文件，通过调用以下两者之一的方法：

- [getSharedPreferences()](http://android.xsoftlab.net/reference/android/content/Context.html#getSharedPreferences(java.lang.String,%20int)) 如果需要多个共享参数文件的话可以使用这个方法，每个文件都拥有一个标识符，可以通过这个标识符通过该方法的第一个参数获得共享参数对象。你可以通过APP中的任意Context对象调用它。
- [getPreferences()](http://android.xsoftlab.net/reference/android/app/Activity.html#getPreferences(int)) 只可以在Activity中使用该方法。该方法适用于你只需要一个与该Activity有关的共享参数文件。因为这个方法会返回一个属于这个Activity的默认的共享参数文件，你不必要去指定共享参数的名称。

举个例子，下面这段代码会在Fragment中执行。这里使用了一个字符串资源来作为共享参数的标识符，并通过该标识符获得对应的共享参数对象，并且以私有的模式打开它，所以这个文件只仅限在你的APP内访问。
```java
Context context = getActivity();
SharedPreferences sharedPref = context.getSharedPreferences(
        getString(R.string.preference_file_key), Context.MODE_PRIVATE);
```

当需要命名你的共享参数文件时，你应该使用APP的唯一标识符来命名，比如"com.example.myapp.PREFERENCE_FILE_KEY"

或者，如果你只需要一个与Activity关联的共享参数文件，你可以使用getPreferences()方法：
```java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
```

> **警告:**如果你使用了MODE_WORLD_READABLE或者MODE_WORLD_WRITEABLE模式创建了一个共享参数文件，那么知道文件标识符的其它APP也可以访问你的数据。


##写入数据到共享参数中
为了可以写入数据到共享参数文件中，需要通过调用SharedPreferences的edit()方法创建一个SharedPreferences.Editor对象。

通过putInt()或putString()方法传入你想写入的键值对数据，然后调用commit()存储更改：
```java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
SharedPreferences.Editor editor = sharedPref.edit();
editor.putInt(getString(R.string.saved_high_score), newHighScore);
editor.commit();
```

##从共享参数中读取数据
如果要从共享参数文件中读取数据，调用比如getInt()或getString()方法，然后传入你想获取值的键，如果键不存在，则会返回一个默认值：
```java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
int defaultValue = getResources().getInteger(R.string.saved_high_score_default);
long highScore = sharedPref.getInt(getString(R.string.saved_high_score), defaultValue);
```