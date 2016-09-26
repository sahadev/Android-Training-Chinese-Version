原文地址：[http://android.xsoftlab.net/training/basics/data-storage/files.html](http://android.xsoftlab.net/training/basics/data-storage/files.html)

Android使用的文件系统和其它平台的磁碟式文件系统很相似。这节课描述了如何通过[File](http://android.xsoftlab.net/reference/java/io/File.html)API在Android文件系统上进行读取文件和写入文件的操作。

一个File对象适合被用来按照从头到尾的方式读取或写入大量的数据，它不适合被用来跳跃式访问，也就是随机访问。举个例子，这对图片文件或者任何基于网络的数据交换是非常适合的。

这节课展示了如何执行APP中最基本的文件相关任务。这节课假定你有Linux的文件系统基础以及Java标准的输入输出基础。

##选择内部或外部存储器
所有的Android设备有两个文件存储分区："internal"和"external"分区。这些名词来自于较早的Android版本，当大多数的设备提供了内置的固定内存，会添加一个可移除的存储器媒体比如micro SD卡(外部存储器)。一些设备固定的存储器空间划分为"internal"和"external"分区，所以甚至是没有可移除的存储媒体，那么仍然还是会有两个存储器空间。API会表现出相同的行为，无论外部存储器是不是可移除的。下面是对每个存储器空间的事实总结。

**Internal storage:**

- 它总是可用的。
- 默认情况下APP将文件保存在这里，只允许该APP可以访问。
- 如果用户卸载了你的APP，那么系统会将APP的所有文件从这里移除。

如果你不想让用户或者其它APP访问你文件的话，内部存储器是一个合适的地方。

**External storage:**

- 它并不总是可用的，因为用户可能将外部存储挂载为USB存储器，和另外一些可能会被从设备上移除的情况。
- 它是全局可访问的，所以如果文件存储在这里，它可能会你的控制范围之外被访问。
- 当用户卸载了你的APP，系统只会删除你在getExternalFilesDir()返回目录中保存的文件。

如果不需要访问限制的话，外部存储器是一个合适的地方，它可以另其它的APP访问你的文件，用户也可以通过电脑访问这些文件。

> **Note:**尽管APP安装的时候会默认被安装在内部存储器内，不过你可以在清单文件中通过指定android:installLocation属性来指定应用安装的位置。如果APK的尺寸很大，并且外部存储空间比内部存储空间大的话，用户会很欣赏这点。更多相关信息，请参见： [App Install Location](http://android.xsoftlab.net/guide/topics/data/install-location.html)。

##获取外部存储器的权限
如果要在外部存储器写入文件，你必须在清单文件中请求WRITE_EXTERNAL_STORAGE权限。
```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
```

> **警告：**目前所有的APP都有能力访问外部存储器而不需要指定权限。然而，这会在未来的发行版做更改。如果你的APP需要读取外部存储器的数据(不是写入),那么你需要声明READ_EXTERNAL_STORAGE权限，来确保你的APP还可以在未来的版本中继续工作，在更改生效之前，你应该现在就声明这项权限。
> ```xml
> <manifest ...>
>     <uses-permission > android:name="android.permission.READ_EXTERNAL_STORAGE" />
>     ...
> </manifest>
> ```
> 然而，如果你的APP使用了WRITE_EXTERNAL_STORAGE权限，那么它会隐性的含有读取外部存储器的权限。

你在内部存储器中保存文件不要任何权限。你的应用程序总是有权读取和写入文件到内部存储器目录。

##保存文件到内部存储器
当要保存文件到内部存储器时，你可以通过下面两个方法中的一个获得一个合适的目录对象File：

[getFilesDir()](http://android.xsoftlab.net/reference/android/content/Context.html#getFilesDir()) 返回一个代表APP的内部存储器目录的File对象。

[getCacheDir()](http://android.xsoftlab.net/reference/android/content/Context.html#getCacheDir()) 返回一个File对象，这个对象代表了APP在内部存储器中的临时文件缓存目录。为了确保文件不再需要的时候可以删除到它，应该在任何给定时间内对内存的数量实现一个合理的尺寸限制，比如1MB。如果系统开始在存储器上缓慢运行，那么可能会删除你的临时文件，而不会有任何警告。

为了在这些目录中创建一个文件，你可以使用[File()](http://android.xsoftlab.net/reference/java/io/File.html#File(java.io.File,%20java.lang.String))的构造方法，传递一个上面的方法目录到File的构造方法中，来指定内部存储器目录。就像这样：
```java
File file = new File(context.getFilesDir(), filename);
```

或者，你可以调用openFileOutput()获得一个FileOutputStream对象来将文件写入到内部存储器中。这是一个如何写入一些文本到文件中的例子：
```java
String filename = "myfile";
String string = "Hello world!";
FileOutputStream outputStream;
try {
  outputStream = openFileOutput(filename, Context.MODE_PRIVATE);
  outputStream.write(string.getBytes());
  outputStream.close();
} catch (Exception e) {
  e.printStackTrace();
}
```

又或者，如果你需要缓存一些文件，你应该使用[createTempFile()](http://android.xsoftlab.net/reference/java/io/File.html#createTempFile(java.lang.String,%20java.lang.String))。举个例子，下面这个方法从URL中提取了文件的名字，然后使用这个名字在内部缓存目录中创建了一个文件：
```java
public File getTempFile(Context context, String url) {
    File file;
    try {
        String fileName = Uri.parse(url).getLastPathSegment();
        file = File.createTempFile(fileName, null, context.getCacheDir());
    catch (IOException e) {
        // Error while creating file
    }
    return file;
}
```
> **Note:**APP的内部存储路径是通过APP的包名特别指定的，并且被放置在Android文件系统中一个特殊的位置。技术上讲，其它的APP是可以读取你的内部文件的，如果你设置的文件模式是可读的。然而，其它的APP也应该知道你APP的包名和文件名。其它的APP不能浏览你的内部目录，并且没有访问和写入的权限，除非你特别设置了文件是和读写的。所以，一旦你对内部存储器文件使用了MODE_PRIVATE模式，那么其它APP永远不可能访问到这些文件。

##保存文件到外部存储器
因为外部存储器有可能是不可用的，比如当用户将它挂载到了PC上，或者将SD卡移除了，所以你应该在访问它之前检查它是否可用。你可以通过getExternalStorageState()方法查询外部存储器的状态。如果返回的状态等于MEDIA_MOUNTED，那么你可以读取和写入文件。举个例子，下面的方法用于检查存储器是否可用：
```java
/* Checks if external storage is available for read and write */
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}
/* Checks if external storage is available to at least read */
public boolean isExternalStorageReadable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state) ||
        Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
        return true;
    }
    return false;
}
```

尽管外部存储器是可以被用户或者其它APP修改的，这里有两种你可能会用到的文件保存类别。

**Public文件：**
在这种情况下，文件应该会被其它APP或者用户随便访问。当用户卸载了你的APP，那么这些文件还会保留给用户。

举个例子，APP捕获的照片或者下载的其它文件。

**Private文件：**
这些文件会正当的属于你的APP，并且在用户卸载APP的时候会删除它们。尽管这些文件从技术上讲是可以被用户或者其它APP访问的，因为这些文件是在外部存储器上。实际上这些文件不会提供给APP之外的用户。当用户卸载了APP，那么系统会将这些文件从外部存储器中删除。

举个例子，APP下载的附加资源或者临时的媒体文件。

如果你想在外部存储器上保存公开的文件，使用[getExternalStoragePublicDirectory()](http://android.xsoftlab.net/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String))方法获得一个代表合适目录的File对象。这个方法携带的参数指定了你需要存储的文件类型，可以使系统对其它公开文件进行逻辑上的组织，比如[DIRECTORY_MUSIC](http://android.xsoftlab.net/reference/android/os/Environment.html#DIRECTORY_MUSIC)或者[DIRECTORY_PICTURES](http://android.xsoftlab.net/reference/android/os/Environment.html#DIRECTORY_PICTURES)，分别代表的就是公共的音乐和图片：
```java
public File getAlbumStorageDir(String albumName) {
    // Get the directory for the user's public pictures directory. 
    File file = new File(Environment.getExternalStoragePublicDirectory(
            Environment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    return file;
}
```

如果你想保存私有文件，可以调用[getExternalFilesDir()](http://android.xsoftlab.net/reference/android/content/Context.html#getExternalFilesDir(java.lang.String))方法获取合适的目录，然后给它传你喜欢的一个类型的目录名称。每一个目录会通过这种方式创建并被添加到父目录中，以便封装所有在外部存储器上的文件，当用户卸载APP的时候会删除它们。

下面这个方法你可使用它创建一个个人的相册目录：
```java
public File getAlbumStorageDir(Context context, String albumName) {
    // Get the directory for the app's private pictures directory. 
    File file = new File(context.getExternalFilesDir(
            Environment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    return file;
}
```

如果对文件没有适合的预先定义的子目录名称，你可以调用[getExternalFilesDir()](http://android.xsoftlab.net/reference/android/content/Context.html#getExternalFilesDir(java.lang.String))并传入一个Null，这会返回APP在外部存储器上私有目录的根目录。

记住在getExternalFilesDir()目录中创建的目录会随着APP的卸载被删除。如果你想在删除之后仍然保留这些文件，比如你的APP是个摄像机应用，用户希望保留下来这些照片，你应该使用[getExternalStoragePublicDirectory()](http://android.xsoftlab.net/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String))方法。

无论你使用的是getExternalStoragePublicDirectory()要共享文件还是使用getExternalFilesDir()保护你APP内的文件，你使用API提供的常量比如[DIRECTORY_PICTURES](http://android.xsoftlab.net/reference/android/os/Environment.html#DIRECTORY_PICTURES)作为目录的名称来说是非常重要的。这些目录的名称可以为系统合适的对待，举个例子，如果文件存储在DIRECTORY_RINGTONES的目录中，系统媒体扫描器会将它识别为铃声文件，而不是音乐文件。

##查询剩余空间
如果你可以提前知道你保存的数据有多少，你可以查询是否有足够的空间可以使用，通过调用[getFreeSpace()](http://android.xsoftlab.net/reference/java/io/File.html#getFreeSpace())或者[getTotalSpace()](http://android.xsoftlab.net/reference/java/io/File.html#getTotalSpace())达到这一点。这些方法分别提供了当前可用空间和磁盘容量上的总空间，这些信息有助于避免填充的磁盘容量超过固定的阈值。

然而，系统不会保证你可以写入与getFreeSpace()返回的剩余空间容量相当的数据。如果返回的数字是几MB，大于你想存储的数据大小，又或者系统当前使用容量少于总容量的90%,那么现在处理可能是安全的。否则，你可能不适合写入到存储器中。

>**Note:**如果你在保存文件之前不检查剩余可用空间，你可以试着以正确的方式写入文件，如果事件发生了，那么可能会引起[IOException](http://android.xsoftlab.net/reference/java/io/IOException.html)异常。如果你不知道你需要的精确空间，那么你可能就需要这么去做。举个例子，如果你在存储文件之前从PNG编码改为了JPEG编码，你可能就不会提前知道文件的大小。

##删除文件
你应该在文件不再需要的时候将它们删除。最直接了当的方式就是拥有一个已经打开了的文件的引用，然后调用它的[delete()](http://android.xsoftlab.net/reference/java/io/File.html#delete())方法。
```java
myFile.delete();
```

如果文件被存在了内部存储器上，你也可以要求[Context](http://android.xsoftlab.net/reference/android/content/Context.html)定位，然后调用[deleteFile()](http://android.xsoftlab.net/reference/android/content/Context.html#deleteFile(java.lang.String))方法删除文件。

```java
myContext.deleteFile(fileName);
```

> **Note:**当用户卸载了你的APP，Android系统会删除以下文件：
> 
> - 所有保存在内部存储器中的文件
> 
> - 所有保存getExternalFilesDir()目录下的外部存储器中的文件
> 
> 无论如何，你应该定期手动删除所有的缓存文件，即便是通过getCacheDir()方法创建的，还应该定期删除你不再需要的其它文件。
