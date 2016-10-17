原文地址：[http://android.xsoftlab.net/training/articles/perf-jni.html](http://android.xsoftlab.net/training/articles/perf-jni.html)

JNI的全称为Java Native Interface，中文意思是Java本地接口。它定义了Java代码与C/C++代码之间的管理方式。它是两者的桥梁，支持从动态共享库中加载代码。虽然有时有些复杂，但是它的执行效率还是蛮高的。

如果你对其还不太熟悉，那么可以通过Java Native Interface Specification来了解一下JNI的大致工作流程以及JNI的特性。

##JavaVM与JNIEnv
JNI定义了两个关键的数据结构："JavaVM"与"JNIEnv"。这两个函数本质上都为指向函数指针的指针表。JavaVM提供了"调用接口"功能，该功能允许创建、销毁JavaVM。理论上每个进程可以拥有多个虚拟机，但是在Android中只允许有一个。

JNIEnv提供了大部分的JNI功能。任何本地方法都以JNIEnv对象为第一回调参数。

JNIEnv被用于线程局部存储。正出于这个原因，**不能在线程间共享JNIEnv**。如果不能够通过其它方式获取其对应的JNIEnv，那么应该共享JavaVM，然后通过GetEnv获取该线程的JNIEnv(假设该线程拥有一个JNIEnv，具体请往下看)。

C与C++对JNIEnv和JavaVM的声明方式并不相同。头文件"jni.h"针对C或者C++提供了不同的类型定义。正因为这个原因，在头文件中包含JNIEnv参数并不是个好主意。

##线程
Android中所有的线程都是Linux线程，都由内核执行。通常由受控代码启动(比如Thread.start)，但是也可以由别的地方创建，然后再附到JavaVM上。举个例子，一个线程可以由pthread_create启动，然后通过AttachCurrentThread或AttachCurrentThreadAsDaemon将其依附到JavaVM。

Android并不会挂起正在执行本地代码的线程。如果垃圾收集正在进行，或者调试器发起了挂起请求，Android将会在下次JNI调用时暂停线程。

通过JNI所附加的线程在退出前**必须调用DetachCurrentThread**。

##jclass, jmethodID, 及jfieldID
如果需要在本地代码中访问对象的属性，那么需要执行以下操作：

- 通过FindClass获取类对象的引用
- 通过GetFieldID获得属性的ID
- 通过适当的方法获取对象的内容，比如GetIntField

相应的，如果要调用一个方法，首先获取类对象的引用，其次获取该方法的ID。ID经常只是指向了一个内部的运行时数据结构。查找这些方法通常需要进行若干次字符串比对，但是一旦找到，那么后期的获取属性或者方法调用都会非常的迅速。

如果性能对你很重要，那么在找到这些属性或者方法之后，将其缓存起来就很帮助了。因为Android中只允许每个进程有一个JavaVM的存在，所以将这些数据缓存在一个静态本地结构中就能说的过去了。

类的引用、属性的ID、方法的ID在这个类被卸载之前都可以保证它们有效。一个类只有在这种情况下才会被卸载：该类所关联的ClassLoader也被回收。虽然这几率很低，但是在Android中不是没有可能的。

如果想在类加载的时候将这些ID缓存下来，并在类被卸载之后再重新加载时还能重新缓存，最正确的方法就是像下面的代码一样添加一段代码：

```java
    /*
     * We use a class initializer to allow the native code to cache some
     * field offsets. This native function looks up and caches interesting
     * class/field/method IDs. Throws on failure.
     */
    private static native void nativeInit();
    static {
        nativeInit();
    }
```

在C/C++代码中创建一个名为nativeClassInit的方法，用于ID的查找与缓存。该方法会在类初始化的时候执行一次。就算是该类被卸载后又重新加载了，那么这个方法还是会被再执行一遍。
##局部引用，全局引用
每个被回调到本地方法的参数，以及几乎所有的通过JNI方法返回的对象都是局部变量。这意味着当前线程中该方法内的所有局部变量都是合法的。**在本地方法返回之后，虽然对象仍然存活，但是引用却是无效的。**

这适用于jobject所有的子类：jclass, jstring, 以及jarray。

获取非局部变量的唯一方式就是通过NewGlobalRef及NewWeakGlobalRef方法获得。

如果需要长时间持有一段引用，那么必须使用全局引用。NewGlobalRef方法会将一个局部引用转换为一个全局引用。在调用DeleteGlobalRef方法之前，该全局引用一直有效。

这种模式通常用于缓存一个由FindClass返回的一个jclass对象：

```java
jclass localClass = env->FindClass("MyClass");
jclass globalClass = reinterpret_cast<jclass>(env->NewGlobalRef(localClass));
```

所有的JNI方法都可以以这两种引用作为参数。不过引用相同的对象可能有不同的结果。举个例子，以同一个引用为参数连续调用两次NewGlobalRef可能会得到不同的返回值。**如果要查看两个引用是否指向了同一个对象，必须使用IsSameObject函数。**绝不要在本地代码中使用"=="比较两个引用。

**绝不要认为在本地代码中的对象引用是个常量或者是唯一的。**一个32位的值所代表的对象的方法调用可能与下次调用就有所不同，这可能是因为两个不同的对象拥有相同的32位值。不要将jobject的值当做键使用。

程序员经常被要求不要过度的申请局部变量。这意味着如果你创建了大量的局部变量，那么应当通过DeleteLocalRef函数手动的释放它们，而不是让JNI为你做这些事情。

要注意jfieldIDs、jmethodID并不是对象引用，所以不能够将它们传给NewGlobalRef函数使用。GetStringUTFChars函数与GetByteArrayElements函数所返回的原始数据指针也同样不是对象。

一个不寻常的情况需要单独说明一下：如果通过AttachCurrentThread函数attach到了一个本地线程上，那么在该线程被detache之前，代码中所有的局部变量都不会被自动释放。任何创建的局部变量都需要手动删除。

##UTF-8与UTF-16字符串
Java语言使用的是UTF-16字符串。为了方便起见，JNI所提供的方法工作在Modified UTF-8字符串下。修正后的编码对于C语言代码很有用，因为它将\u0000编码为了0xc0 0x80。

**不要忘记释放你所获得的字符串**。字符串函数会返回jchar* 或 jbyte*，它们是指向原始数据的指针，而不是本地引用。它们在被释放之前一直有效，这意味着在本地方法返回后，它们并没有被释放。

**传给NewStringUTF函数的数据必须是Modified UTF-8格式**。一个常见的错误就是从文件流或者网络流中读取字符串数据，然后没有过滤就直接交给了NewStringUTF函数进行处理。除非你知道这些数据是7位的ASCII，否则你需要剔除高位的ASCII字符串或者将它们转换为正确的Modified UTF-8格式。如果你不这么做，那么转换的结果可能不是你想看到的。额外的JNI检查会扫描字符串并会警告你这是无效的数据，但是它们不会捕获任何事情。

##原始数组
JNI提供了用于访问对象数组的功能。然而，同一时间只能对一个元素进行访问，可以直接对数组今夕读写操作，就好像直接在C中声明的一样。

为了使JNI接口尽可能的高效，也不受虚拟机实现的限制，调用Get<PrimitiveType>ArrayElements的相关方法允许返回一个指向实际值的指针，或者申请一些内存以完成复制。无论哪种方法，所返回的指针可以保证是有效的，直到相应的释放方法被触发。**必须释放你所取得的每个数组**。如果Get方法调取失败，也需要保证不要去释放一个空的指针对象。

你可以通过isCopy参数来检测一个数组是否是由指针所拷贝过来的，这一点很有用。

Release方法需要一个mode参数，这个参数有三种值。运行时执行的操作取决于是否它返回指向实际数据的指针或者指针的副本：

- 0
 - 实际指针：非final修饰的数组对象
 - 指针副本：拷贝后的数组数据，拷贝的缓冲区会被释放
- JNI_COMMIT
 - 实际指针：不做任何事情
 - 指针副本：拷贝后的数组数据，拷贝的缓冲区不会被释放
- JNI_ABORT
 - 实际指针：非final修饰的数组对象。早些写入不会被中止。
 - 指针副本：所拷贝的缓冲区被释放；缓冲区内的任何变更都会丢失。

检查isCopy标志的其中一个原因是需要知道在对数组作出变更之后是否需要调用JNI_COMMIT的相关释放方法，如果正在更改一个正在作出变更以及读取数组内容的操作，那么可以根据该标志跳过这次操作。另一个可能的原因就是用于有效的处理JNI_ABORT。举个例子，你可能想要得到一个数组，然后对其修改之后将其传给一个函数。如果你知道JNI会为你做一个副本的话，那么就不需要创建另外的可编辑副本了。如果JNI传回的是原始数据，那么你自己需要创建一个副本。

一个常见的错误就是如果*isCopy是false，那么可以不调用相关释放方法。但是事实并非如此，如果没有申请拷贝缓冲区，那么原始数据内存必定会被一直占用，也不会被垃圾收集器回收。

还要注意的是，JNI_COMMIT并不会释放数组，你需要在另外的标志执行后再执行一次释放。

##方法调用
JNI在方法使用上有两种方式，一种如下所示：
```java
    jbyte* data = env->GetByteArrayElements(array, NULL);
    if (data != NULL) {
        memcpy(buffer, data, len);
        env->ReleaseByteArrayElements(array, data, JNI_ABORT);
    }
```

上面这段代码首先获取了一个数组，然后拷贝出len个字节的元素，最终将这个数组释放。根据实现的不同，Get调用会返回原始数据或者数据副本。在这个案例中，JNI_ABORT可以确保不出现第三个副本。

另一种实现则要更简单一些：
```java
    env->GetByteArrayRegion(array, 0, len, buffer);
```

对于此有若干建议：
- 减少JNI调用可以节省开销。
- 不要原始数据或者额外的数据拷贝。
- 降低程序员出错的风险--他们会在某些操作失败后忘记调用相关的释放方法。

类似的，你可以使用Set<Type>ArrayRegion函数将数据拷贝到一个数组中，GetStringRegion函数或GetStringUTFRegion可以从String拷贝大小不等的字符。

##异常
**当异常出现时，请不要继续向下执行**。代码应当注意到这些异常并返回，或者处理这些异常。

当异常发生时，只有以下JNI方法允许调用：

- DeleteGlobalRef
- DeleteGlobalRef
- DeleteLocalRef
- DeleteWeakGlobalRef
- ExceptionCheck
- ExceptionClear
- ExceptionDescribe
- ExceptionOccurred
- MonitorExit
- PopLocalFrame
- PushLocalFrame
- Release<PrimitiveType>ArrayElements
- ReleasePrimitiveArrayCritical
- ReleaseStringChars
- ReleaseStringCritical
- ReleaseStringUTFChars

很多JNI函数可以抛出异常，不过只是提供了一种很简单的检查方式。比如，如果NewString函数返回了一个非空的值，那么就不需要检查异常。然而，如果你调用一个方法，比如CallObjectMethod，那么就需要每次都检查一下异常，因为如果异常被抛出后，返回值是无效的。

主要注意的是，由中断所抛出的异常不会释放本地栈帧，Android目前也不支持C++异常。JNI的Throw与ThrowNew结构也只是在当前的线程设置了一个异常指针。当异常发生时也只是返回到代码调用处，异常也不会被正确的注意与处理。

本地代码可以通过ExceptionCheck函数或ExceptionOccurred函数捕获异常，并可以通过ExceptionClear函数清理这些异常。通常情况下，不处理这些异常会导致一些问题的出现。

JNI中并没有与Throwable相对应的映射函数，所以，如果你想获得异常字符串，那么就需要先找到Throwable类，然后查找相关的getMessage "()Ljava/lang/String;"方法ID，然后调用这些方法，如果返回的值是非空的话，调用GetStringUTFChars函数来获得你想得到的异常字符串，可以将这些异常打印出来。

##本地库
你可以通过标准的System.loadLibrary函数加载共享库中的本地代码。推荐的获取本地代码的方法有：

- System.loadLibrary()，该方法唯一的参数是一个简要的库名，所以如果要加载"libfubar.so"，你只需要传"fubar"即可。
- 本地方法：jint JNI_OnLoad(JavaVM* vm, void* reserved);
- 在JNI_OnLoad方法内部，注册所有的本地方法。如果将方法声明为"static"的话，那么方法名称将不会占用符号表的空间。

如果JNI_OnLoad函数是由C++实现的话，那么它看起来应该是这个样子：
```java
jint JNI_OnLoad(JavaVM* vm, void* reserved)
{
    JNIEnv* env;
    if (vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6) != JNI_OK) {
        return -1;
    }
    // Get jclass with env->FindClass.
    // Register methods with env->RegisterNatives.
    return JNI_VERSION_1_6;
}
```

你也可以通过System.load函数外加库的全限定名来加载本地库。

使用JNI_OnLoad另一个需要注意的是：任何FindClass调用都会发生在类加载器的上下文环境中，该类加载器用于加载共享库。通常情况下，FindClass所用到的加载器位于解释栈的顶端，如果还没有加载器，那么它会使用系统的加载器。

##64位的注意事项
Android当前运行于32位的平台上。虽然理论上可以为64位的平台构建系统，但是目前它不是主要的目标。大多数情况下，这不是你需要担心的事情，但是如果要将指针存储于本地结构中的一个对象的Int属性上，那么这就很值得关注了。为了支持64位指针结构，**你需要将本地指针存储于一个Long属性中**。

##不支持特性与向后兼容
支持所有的JNI1.6特性，以及以下异常：
- DefineClass 还没有实现。Android并没有使用Java的字节码以及类文件，所以传入二进制的类数据是不会执行的。

如果需要兼容Android老的版本，那么应该检查以下部分：

- **动态查询本地函数**
 - 在Android 2.0之前，字符'$'在查找方法时不会被正确的转换为"_00024"。所以使用有关方法需要明确注册或者将内部类方法移出。
- **分离线程**
 - 在Android 2.0之前，无法使用pthread_key_create析构函数来避免"在退出之前必须分离线程"这项检查。
- **弱的全局引用**
 - 在Android 2.2之前，弱的全局引用还没有实现。之前的版本会拒绝使用它们。你可以使用Android平台版本来检测是否支持。
 - 在Android 4.0之前，弱的全局引用只能被传入NewLocalRef, NewGlobalRef, 以及 DeleteWeakGlobalRef这几个函数。
 - 从Android 4.0开始，弱的全局引用可以像其它JNI引用一样使用。
- **本地引用**
 - 在Android 4.0之前，本地引用实际上就是指针。在Android 4.0中添加了必要的中间角色，以便更好的支持垃圾回收器的工作，不过这意味着有很多JNI的bug在老版本上无法察觉。查看[JNI Local Reference Changes in ICS](http://android-developers.blogspot.com/2011/11/jni-local-reference-changes-in-ics.html)获取更多信息。
- **通过GetObjectRefType检查引用类型**
 - 在Android 4.0之前，由于直接指针的使用，无法正确的实现GetObjectRefType。我们通过弱的全局表、参数、本地表以及全局表进行查找。首先它会找到你的直接指针，它会返回它所检查的引用类型。这意味着，如果你在全局的jclass上作用GetObjectRefType，而这个jclass以一个隐性参数传给了一个静态本地方法，那么你将会获得JNILocalRefType而不是JNIGlobalRefType。
