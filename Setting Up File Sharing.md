原文地址：[http://android.xsoftlab.net/training/secure-file-sharing/index.html](http://android.xsoftlab.net/training/secure-file-sharing/index.html)

#导言
APP经常需要给其它的APP提供一个或多个文件。举个例子，相册APP可能需要提供文件以供编辑，或者一个文件管理的APP可能希望用户在外部存储器中的两个区域之间复制粘贴文件。其中一种方式就是发送端的APP可以分享文件来响应接收端APP的请求。

在所有的例子中，使一个文件从你的APP到另一个APP的唯一安全方式就是发送这个文件内容的URI地址到接收端APP，并且授予一个临时的访问权限给这个URI。带有临时URI访问权限的内容URI是安全的，因为它只会被应用于接收这个URI的那个APP，并且会这个权限会自动终止。Android的[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)组件提供了[getUriForFile()](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html#getUriForFile(android.content.Context,%20java.lang.String,%20java.io.File))方法来生成该文件的内容URI地址。

如果你想分享少量的文本或者数字型数据，你应该发送一个Intent，使这个Intent携带这些数据给其它APP。有关学习如何使用Intent来发送简单的数据，参见训练课程[Sharing Simple Data](http://android.xsoftlab.net/training/sharing/index.html)。

这节课解释了如何使用Android [FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)组件生成的内容URI安全的在APP之间共享文件。

#设置文件共享
为了从你的APP安全的提供文件给其它APP，你需要配置你的APP以便对文件提供安全的防护。Android的[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)组件会基于在XML中提供的参数对文件的相应URI地址，。这节课展示了如何给APP添加默认的[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)实现，以及展示如何指定你要分享给其它APP的文件。

> **Note:**[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)是v4支持包的一部分。有关程序中包含该库的更多信息，请参见：[Support Library Setup](http://android.xsoftlab.net/tools/support-library/setup.html)。

##指定FileProvider
给APP定义FileProvider需要在清单文件中登记。在被登记的条目需要指定URI的权限，和指定XML文件的文件名一样，这里也需要指定被分享文件的目录。

下面这一小段代码展示了如何在清单文件中添加< provider>元素，在这个元素中指明了[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)类，权限和XML文件名：
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    <application
        ...>
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.example.myapp.fileprovider"
            android:grantUriPermissions="true"
            android:exported="false">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/filepaths" />
        </provider>
        ...
    </application>
</manifest>
```

在这个例子中，属性android:authorities指明了由[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)生成的URI的权限，这里的权限值是com.example.myapp.fileprovider。在自己的APP中，权限是由APP的android:package值以及跟随的"fileprovider"字符串组成。有关学习更多权限值的课程，请参见话题：[Content URIs](http://android.xsoftlab.net/guide/topics/providers/content-provider-basics.html#ContentURIs)以及[android:authorities](http://android.xsoftlab.net/guide/topics/manifest/provider-element.html#auth)的属性文档。

< provider>的子元素< meta-data>指向了一个XML文件，这个XML文件指定了你想要分享出去的目录。属性android:resource的值是要分享文件的路径与名称，只是这个文件名不带.xml后缀。文件的内容会在下节课描述。

##指定可搜索目录
一旦在清单文件中添加了FileProvider,那么你还需要指定将要分享的文件目录。如果要指定，首先需要在工程的res/xml/子目录中创建一个名为filepaths.xml的文件。在这个文件中，通过给每个目录添加相应的XML元素来指定它们的目录。下面代码展示了res/xml/filepaths.xml文件中的样例，这段代码中还演示了如何分享内部存储器中files/目录下的子目录：
```xml
<paths>
    <files-path path="images/" name="myimages" />
</paths>
```

在这个例子中，<files-path>标签分享了一些目录，这个目录位于APP在内部存储器中的files/目录下。属性path分享了files/目录的子目录images/目录，属性name则用于告诉[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)给文件的URI添加files/images/子目录下的路径段 myimages。


< paths>元素可以拥有多个子元素，每一个元素分别指向相应的分享目录。在附加的<files-path>元素中，你可以使用<external-path>元素来分享外部存储器上的目录，使用<cache-path> 元素分享内部存储器上的目录。学习更多有关分享指定目录的子元素，请参见[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)的引用文档。

>**Note:**使用XML文件是分享指定目录的唯一方式，你不可以动态的添加目录。

你现在有了使用[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)对文件生成相关URI的完整说明。当你的APP需要为文件产生URI的时候，它包含了< provider>元素指定的权限，以及文件的路径myimages/，还有文件的名称。

举个例子，如果你通过这节课中的所有片段定义了一个[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)，以及你要请求一个default_image.jpg文件的URI地址，那么[FileProvider](http://android.xsoftlab.net/reference/android/support/v4/content/FileProvider.html)会返回如下的URI：
```xml
content://com.example.myapp.fileprovider/myimages/default_image.jpg
```

