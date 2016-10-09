原文地址：http://android.xsoftlab.net/training/articles/perf-jni.html

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


##UTF-8与UTF-16字符串