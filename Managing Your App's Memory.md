原文地址：[http://android.xsoftlab.net/training/articles/memory.html](http://android.xsoftlab.net/training/articles/memory.html)

随机存储器(RAM)在任何软件运行环境中都是一块非常重要的区域，尤其是在内存受限的移动操作系统上。尽管Android的Dalvik虚拟机会进行垃圾回收，但是这不意味着APP就可以忽略所申请及所释放的内存。

为了可以使垃圾回收器能清理APP所使用的内存空间，你需要防止内存泄漏的事情发生，并需要在适当的时间将[Reference](http://android.xsoftlab.net/reference/java/lang/ref/Reference.html)对象释放。对于大多数APP来说，垃圾回收器会在正确的对象使用完毕之后将其所占用的内存回收释放。

这节课将会学习Android如何管理APP的进程以及内存空间、以及如何正确的减少内存的使用。

##Android如何管理内存
Android并没有提供专门的内存交换空间，但是它使用了[paging](http://en.wikipedia.org/wiki/Paging)及[memory-mapping](http://en.wikipedia.org/wiki/Memory-mapped_files)来管理内存。这意味着任何你所修改的内存——是否被对象分配所使用，或者是否是被映射的内存页面——这些仍然遗留在内存中，不能被交换出去。所以完全释放APP内存的唯一方式就是释放任何你可能所持有的对象引用，这样才可以使垃圾回收器对其进行回收。不过这里有一个例外：任何没有被修改的文件映射，比如代码，在系统需要的时候会被移出RAM。

###共享内存
为了可以在RAM中满足一切要求，Android试着在进程间分享RAM页。可以通过以下几种方式做到：

- 每个APP进程都是由一个名为Zygote的进程fork出来的。Zygote进程在系统启动加载通用框架代码及资源(比如Activity的主题)时启动。为了启动新的APP进程，系统会先fork出Zygote进程，然后再在新的进程中加载、运行APP的代码。这允许大多数的RAM页面为Android框架的代码以及资源分配内存以便使所有的APP进程都可以使用。
- 大多数的静态数据都是被映射到进程中的。这种方式不仅可以在进程间共享数据，还可以在需要的时候将其移除页面。静态数据包含：Dalvik代码(放置在预链接的.odex文件中)，APP资源以及在.so文件中的本地代码。
- 在很多地方，Android通过显式的内存分配区域在不同的进程间共享同一块RAM。比如，WindowSurface就在APP与屏幕合成器间使用了共享内存，CursorBuffer在内容提供者与客户端之间使用了共享内存。

由于大量使用了共享内存，所以检查APP占用的内存量就很有必要了。

##分配与回收应用内存
以下是Android回收与再分配内存的一些情况：
- Dalvik中每个进程的堆都有虚拟内存范围限制。这个范围取决于逻辑堆尺寸的定义，它可以随着APP的需要随之增长(不过只是会增长到系统为每个app所分配的内存大小)。
- 堆栈的逻辑尺寸并不等同于堆栈所使用的物理内存大小。当系统检查APP的堆时，会计算一个名为Proportional Set Size(PSS)的值，PSS的意思是，与其他进程共享的，需要清理的页面列表。有关更多PSS的相关信息，请阅读指南：[Investigating Your RAM Usage](http://android.xsoftlab.net/tools/debugging/debugging-memory.html#ViewingAllocations)。
- Dalvik堆栈对堆栈的逻辑空间并不是连续排布的，这句话的意思是Android并不会对堆空间进行碎片整理。Android只有在未使用空间到达堆栈的末端时才会收缩堆栈的逻辑空间。不过这不意味着堆所使用的物理空间不能被收缩。在垃圾收集之后，Dalvik会扫描堆并找出无用页面，然后使用madvise将这些页面返回到kernel。所以，成对的分配回收大段的内存可以使大量的内存能够重复使用。然而,回收小段的内存的效率可能会很低，因为小段内存的页面可能正在被使用，并没有被释放。

##侦测应用内存
为了维持一个多任务执行环境，Android为每个APP的堆大小都设置了硬性限制。具体的堆大小都不相同，这取决于RAM的大小。如果APP已经将所分配的堆容量用完，并还要继续申请更多的内存，那么该APP会收到[OutOfMemoryError](http://android.xsoftlab.net/reference/java/lang/OutOfMemoryError.html)的错误。

在一些情况中，你可能需要知道当前的设备中还有多少堆内存可用。比如，检查有多少数据缓存在内存中是安全的。你可以通过getMemoryClass()方法进行这样查询。它会返回一个整型数值，这个数值以兆字节为单位，代表了APP堆内存的可用值。这项内容将会在下面进行详细讨论。

###APP的切换
用户在切换APP时并没有使用交换空间，Android将切换到后台的进程放置在一个LRU(最近最少使用)缓存中。这么说吧，用户先开启了一个APP，那么会专门有个进程为它启动，后来用户离开了该APP，但这个APP的进程并没有退出，那么这时系统会将这个APP的进程缓存下来，所以如果用户再次返回了该APP，那么刚刚缓存的进程会被再次利用，以便完成快速切换。

如果APP含有一个缓存进程，并且占用了当前系统并不那么需要的内存，那么在用户不再使用它时，它会影响到系统的整体性能。所以，随着系统的可用内存减少，系统可能会杀死LRU缓存中最近最少使用到的进程。为了使APP尽可能缓存的时间长，下面的章节会介绍何时应当释放引用。

有关更多进程在后台如何缓存以及Android是如何决定哪个进程应当被杀死的相关信息，请参见：[Processes and Threads](http://android.xsoftlab.net/guide/components/processes-and-threads.html)。

##APP应当如何管理内存
APP应当在每个开发阶段考虑RAM的限制，包括APP的设计阶段。下面将会列出几种有效的解决方案：

在开发时应当采用以下的方式来增进内存的使用效率。

###尽可能少的使用服务
如果APP需要使用服务在后台做一些工作，绝不要在服务内做不必要的工作。还要注意在工作完成之后，如果服务停止失败，则要当心服务的泄露。

当启动服务时，系统宁愿专门为该服务搭配一个进程。这会使得系统的开销非常高昂，因为服务所使用的内存不能作为它用。这会减少系统保持在LRU缓存中的进程数量，并会使得APP的转换效率低下。当内存非常紧张或者系统不能够保证有足够的进程来维持当前的服务数量时它甚至会引起系统的卡死。

对于以上问题最佳的解决方案就是使用IntentService来限制本地服务的数量。


当服务不再需要的时候，留下服务继续运行是APP常见的**一种非常糟糕的内存管理错误**。所以不要贪图使服务保持长时运行。不及时停止服务不但会增加APP RAM容量不够用的风险，而且还会使用户觉得该APP做的非常的烂，并顺便将其卸载。

###在UI不可见时释放内存
当用户切换到其它APP时，这时你的APP UI会变得不可见，所以应该释放与UI相关的所有资源。及时释放UI资源可以显著的增长系统缓存进程的能力，这会直接影响到用户的体验。

为了可以在用户离开UI后还能收到系统通知，应当在Activity内实现[onTrimMemory()](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#onTrimMemory(int))方法。应当使用该方法来监听[TRIM_MEMORY_UI_HIDDEN](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_UI_HIDDEN)标志，这个标志代表了UI目前是隐藏态，应当释放UI所用到的所有资源。

这里要注意，[TRIM_MEMORY_UI_HIDDEN](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_UI_HIDDEN)标志代表的是APP内所有的UI组件对于用户隐藏。这要与[onStop()](http://android.xsoftlab.net/reference/android/app/Activity.html#onStop())区分开，该方法是在Activity的实例变的不可见时调用，它是在APP内部Activity之间的切换时调用的。所以尽管在[onStop()](http://android.xsoftlab.net/reference/android/app/Activity.html#onStop())中释放了Activity的资源比如网络连接，注销广播接收器等等，但是一般不要在该方法内释放UI资源。这可以使得在用户返回该Activity时，UI资源可以迅速恢复。

###在内存紧张时释放内存
在APP生命周期的任何阶段，[onTrimMemory()](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#onTrimMemory(int))方法会告知当前设备内存很紧张。你应当在收到以下标志时进一步的释放资源：

- [TRIM_MEMORY_RUNNING_MODERATE](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_RUNNING_MODERATE) APP目前处于运行态，暂时不会被杀死，但是设备目前处于低内存运行态，并且系统正在杀死LRU缓存中的进程。
- [TRIM_MEMORY_RUNNING_LOW](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_RUNNING_LOW) APP目前处于运行态，暂时不会被杀死，但是设备目前处于极低内存运行态，所以你应当释放无用的资源来增进系统的性能。
- [TRIM_MEMORY_RUNNING_CRITICAL](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_RUNNING_CRITICAL) APP还处于运行态，但是系统已经准备将LRU缓存中的大部分进程杀死，所以APP应当立即释放所有不必要的资源。如果系统没有获得足够数量的RAM空间，那么系统会清除LRU中的所有进程，并会杀死一些主机正在进行的服务。

还有，在APP处于缓存状态时，你可能会收到以下标志：

- [TRIM_MEMORY_BACKGROUND](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_BACKGROUND) APP处于低内存运行态，APP的进程处于LRU列表的前端。尽管APP所面临被杀死的风险还比较低，但是系统可能已经做好了杀死LRU进程中的准备。APP应当释放那些易于恢复的资源，这样的话，进程会继续保留在缓存列表中，并且会在用户返回到APP时迅速恢复。
- [TRIM_MEMORY_MODERATE](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_MODERATE) APP处于低内存运行态，APP的进程处于LRU列表的中部。如果系统的内存进一步的降低，那么APP的进程可能就会被杀死。
- [TRIM_MEMORY_COMPLETE](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_COMPLETE) APP处于低内存运行态。如果系统没有足够内存的话，APP的进程首当其冲会被杀死。APP应当释放在恢复APP时一切不重要的事物。

因为[onTrimMemory()](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#onTrimMemory(int))方法添加于API 14，所以可以使用[onLowMemory()](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks.html#onLowMemory())来兼容老版本，它大致与[TRIM_MEMORY_COMPLETE](http://android.xsoftlab.net/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_COMPLETE)标志是等价的。

> **Note:** 当系统开始杀死LRU中的进程时，尽管它是自下而上工作的，但是系统还是会考虑这么一种情况：哪个进程消耗的内存比较多，所以如果将该进程杀死后，将会获得更多的内存。所以在APP处于LRU缓存时，尽可能的消耗少量的内存，这样一直维持在缓存列表中的机会才大，才可以在切换回APP时迅速恢复状态。

###检查应该使用多少内存
就像我们早期提到的，运行Android系统的设备的RAM空间各有不同，所以提供给每个APP的堆空间也是不同的。你可以通过[getMemoryClass()](http://android.xsoftlab.net/reference/android/app/ActivityManager.html#getMemoryClass())方法获得APP的可用空间。如果APP试图向系统申请比该方法返回值大的内存空间的话，那么它会收到一个[OutOfMemoryError](http://android.xsoftlab.net/reference/java/lang/OutOfMemoryError.html)错误。

在一些特别特殊的环境中，你可以申请更大的堆空间，可以通过在清单文件的< application>标签中添加largeHeap="true"属性的方式来设置。在设置之后，可以通过getLargeMemoryClass()来查询大尺寸的堆栈空间量。

然而，申请大堆空间的APP只有正常用途才应该申请，比如大照片编辑类APP。决不要是因为经常出现了OutOfMemory错误才这么去做，你应该做的是解决那个OutOfMemory的问题。只有在你明确知道正常的堆空间不足以支撑APP的运行时才应该这么做。使用额外的内存空间会严重损害整体的用户体验，因为垃圾收集器会在此消耗更长的时间，并且在任务切换或者执行其它并发操作时系统性能会明显减慢。

此外，大堆空间的尺寸在所有的设备上并不是相等的。当运行在某些RAM限制的设备上，大堆空间的尺寸可能与常规的堆空间尺寸相等。所以，就算是申请了大堆空间，那么还是应该使用[getMemoryClass()](http://android.xsoftlab.net/reference/android/app/ActivityManager.html#getMemoryClass())来检查一下常规堆空间大小，并尽量将内存的使用量控制在这个范围以下。

###避免浪费位图的内存
当加载一张图片到内存时，最好是将该图片适配到当前屏幕分辨率大小之后再做内存缓存，如果原图本身分辨率很高的话，最好将其缩小到适合屏幕分辨率大小。要注意，位图的分辨率增加，所占的相应内存也一并增加。

> **Note:** 在Android 2.3.x之前，位图对象无论分辨率是多大都是以相同的大小出现在堆中的，因为位图的实际像素数据被单独的存放在了本地内存中。这使得位图内存分配的调试变得很困难，因为大多数的堆栈分析工具并不能探测到本地内存的分配。然而，自Android 3.0之后，位图的像素数据被分配与APP的Dalvik堆栈中，这样增进了垃圾回收的效率以及调试能力。

###使用优化过的数据容器
我们建议使用Android框架优化过的数据容器，比如[SparseArray](http://android.xsoftlab.net/reference/android/util/SparseArray.html),[SparseBooleanArray](http://android.xsoftlab.net/reference/android/util/SparseBooleanArray.html),以及[LongSparseArray](http://android.xsoftlab.net/reference/android/support/v4/util/LongSparseArray.html).常规的[HashMap](http://android.xsoftlab.net/reference/java/util/HashMap.html)其实效率是很低的，因为它需要为每个映射创建单独的实体。另外[SparseArray](http://android.xsoftlab.net/reference/android/util/SparseArray.html)的工作效率更高，因为它可以避免系统对键或值的自动装箱功能。

###要对内存的消耗有一定的意识
要充分了解你所使用的语言以及库的内存开销，并要一直保持有这种意识，包括在APP的设计阶段。经常表面的事物看起来无伤大雅，但实际上它们所消耗的内存是很高的。比如：

- Java的枚举类型需要占用静态常量的两倍内存。你应该坚决制止在Android中使用枚举。
- Java中的每个类，包含匿名内部类的代码需要占用500个字节。
- 每个类的实例需要占用12-16个字节的RAM空间。
- 将每个实例放入HashMap需要格外花费32个字节的空间。 

虽然以上的内容只是会消耗几个字节的空间，但是它们会在程序的内部迅速的积累增加，成为一个巨无霸级的开销。它会使你在分析内存问题的时候让你处于一个非常尴尬的境地，因为这些众多的小对象消耗了大量的内存。

###当心抽象代码
经常开发者会使用抽象代码实现一种良好的程序设计结构，因为抽象代码可以增进代码的灵活性与可维护性。不过，抽象代码会有不菲的开销：通常它们需要更多的执行代码，需要花费更多的时间以及更多的RAM空间来将这些抽象代码映射到内存。所以，如果不是必须的话，最好远离它们。

###为序列化数据使用纳米级缓冲协议(这段翻译的有问题)
Protocol buffers是一种由Google设计的序列化结构的数据。它与语言无关、与平台无关的、可扩展。与XML类似，但是体积更小，速度更快、也更简单。如果你决定要使用该协议，那么就应当在客户端代码中一直使用它。常规的protobufs会生成非常冗长的代码，这会引出相当多的问题：增加内存的消耗，增长APK的体积，减缓执行效率并会迅速接近DEX标志的限制。

###避免依赖注解框架
使用[Guice](https://code.google.com/p/google-guice/)、[RoboGuice](https://github.com/roboguice/roboguice)这类注解依赖框架是相当方便的，因为这些框架可以简化代码的书写，以及提供了相应的测试环境。然而，这些框架在扫描代码的注解时会执行大量的初始化工作，这会使得大量的代码映射到RAM中，尽管你不需要降这些代码载入内存。这些被映射的页面会一直驻留在内存中，虽然系统可以将它们清除，但是只有在这些页面长时间驻留在内存中才会执行清理。

###使用第三方库要当心
第三方代码通常不是专门为移动设备而写。当这些代码运行在移动客户端时往往执行效率很低。在决定使用第三方库之前，应该假设正在执行一项很重要的移植工作，并将要负担为移动设备的维护、优化工作。在决定使用之前要分析该库的大小以及RAM的占用。

就算是某些库是专门为Android所设计的，但是它们还是存在隐患的，因为每个库所做的事不同。举个栗子，一个库可能使用了纳米级的protobufs，而另一个库则使用了毫米级的protobufs。那么现在在APP中使用了两个级别的protobufs。这两种差异可能会发生在日志、解析、图像加载框架、缓存以及其它任何你不期望的事情上。

还要当心掉入共享库的陷阱，这种共享库有一个共同的特点就是，你只使用了该库所提供的很小的功能，你并不希望将其它用不到的大量代码也一并放入你的工程内。在最后，如果你不是特别的需要这个第三方库的话，那么最好的方式就是自己实现一个。
###优化整体性能
有关APP整体性能优化的建议都列在了[Best Practices for Performance](http://android.xsoftlab.net/training/best-performance.html)中。这些建议还包括了CPU的性能优化，除此之外还包括了内存的优化，比如减少布局对象的数量。

你还应该读一读有关[optimizing your UI](http://android.xsoftlab.net/tools/debugging/debugging-ui.html)的文章，文章内包含了布局调试工具以及[lint tool](http://android.xsoftlab.net/tools/debugging/improving-w-lint.html)中所提示的一些布局优化建议。

###使用ProGuard筛除无用代码
[ProGuard](http://android.xsoftlab.net/tools/help/proguard.html)工具可以通过移除无用代码以及以一种无意义的名称重命名类名，属性，方法的方式来达到一种精简、优化、模糊的效果。接下来还必须使用zipalign工具对重命名后的代码进行调整。如果不做这一步将会大大增加RAM的使用量，因为类似于资源这些事物不会再由APK映射到内存。

> **作者PS:** 这段话摘自于zipalign的介绍，相当于是说Zipalign的原理与优势: Specifically, it causes all uncompressed data within the .apk, such as images or raw files, to be aligned on 4-byte boundaries. This allows all portions to be accessed directly with mmap() even if they contain binary data with alignment restrictions. The benefit is a reduction in the amount of RAM consumed when running the application.

###分析RAM的使用状况
一旦APP达到一个相对稳定的程度，那么接下来就需要分析APP在各个生命周期的RAM使用情况了。有关如何分析APP的RAM使用情况，请参见： [Investigating Your RAM Usage](http://android.xsoftlab.net/tools/debugging/debugging-memory.html)。

###使用多进程
如果它适用于你的APP，那么另一项可能帮助你管理APP内存的升级建议就是将组件部署到不同的进程中。使用这项建议必须总是特别的小心，并且大部分APP不应该使用这项技术，如果处理不当的话它会迅速的增加RAM的消耗。这项技术对于那些运行在后台的工作与前台的工作一样重要的APP极为有用，并且可以单独管理这些操作。

使用多进程最适合的场景就是音乐播放器。如果整个APP运行在单一的进程中，那么Activity UI所执行的大部分内存分配都会和音乐的播放保持相同的时间，甚至是用户切换到了其它APP。那么像这样的APP就应该拥有两个进程：一个进程负责UI，而另一个的工作就是持续不断的运行后台服务。

你可以在清单文件中需要执行单独进程的组件里添加android:process属性来实现独立进程。比如，你可以在需要执行单独进程的服务中添加该属性，并声明该进程的名称"background"(你可以命名任何你想命名的名称)：

```xml
<service android:name=".PlaybackService"
         android:process=":background" />
```

进程的名称应该以冒号':'开头，以便确保该进程属于你APP的私有进程。

在决定创建一个新进程之前，你应该了解一下内存的影响。为了演示每个进程的执行效果，首先要考虑到一个不做任何事情的进程需要占用大约1.4MB的内存空间，下面显示了空态下的内存信息堆：

<pre class="no-pretty-print">
adb shell dumpsys meminfo com.example.android.apis:empty
** MEMINFO in pid 10172 [com.example.android.apis:empty] **
                Pss     Pss  Shared Private  Shared Private    Heap    Heap    Heap
              Total   Clean   Dirty   Dirty   Clean   Clean    Size   Alloc    Free
             ------  ------  ------  ------  ------  ------  ------  ------  ------
  Native Heap     0       0       0       0       0       0    1864    1800      63
  Dalvik Heap   764       0    5228     316       0       0    5584    5499      85
 Dalvik Other   619       0    3784     448       0       0
        Stack    28       0       8      28       0       0
    Other dev     4       0      12       0       0       4
     .so mmap   287       0    2840     212     972       0
    .apk mmap    54       0       0       0     136       0
    .dex mmap   250     148       0       0    3704     148
   Other mmap     8       0       8       8      20       0
      Unknown   403       0     600     380       0       0
        TOTAL  2417     148   12480    1392    4832     152    7448    7299     148
</pre>

> **Note:** 如何阅读这些信息请参见[Investigating Your RAM Usage](http://android.xsoftlab.net/tools/debugging/debugging-memory.html#ViewingAllocations)。这里的关键数据是Private Dirty及Private Clean所指示的内存。它们分别说明了这个进程使用了大概1.4MB左右的非交换页内存，而另外150K RAM则是被映射到内存之后将要执行的代码所占用的空间。

了解空进程状态下的内存占用是相当重要的，它会随着工作的开始迅速增长。比如，下面是一个显示了一些文本的Activity的内存占用情况：

<pre class="no-pretty-print">
** MEMINFO in pid 10226 [com.example.android.helloactivity] **
                Pss     Pss  Shared Private  Shared Private    Heap    Heap    Heap
              Total   Clean   Dirty   Dirty   Clean   Clean    Size   Alloc    Free
             ------  ------  ------  ------  ------  ------  ------  ------  ------
  Native Heap     0       0       0       0       0       0    3000    2951      48
  Dalvik Heap  1074       0    4928     776       0       0    5744    5658      86
 Dalvik Other   802       0    3612     664       0       0
        Stack    28       0       8      28       0       0
       Ashmem     6       0      16       0       0       0
    Other dev   108       0      24     104       0       4
     .so mmap  2166       0    2824    1828    3756       0
    .apk mmap    48       0       0       0     632       0
    .ttf mmap     3       0       0       0      24       0
    .dex mmap   292       4       0       0    5672       4
   Other mmap    10       0       8       8      68       0
      Unknown   632       0     412     624       0       0
        TOTAL  5169       4   11832    4032   10152       8    8744    8609     134
</pre>

现在进程使用了刚刚的三倍内存，将近4MB，只是在UI中展示了一段文本而已。这可以推出一个非常重要的结论：如果你将APP的功能放在多个进程中执行，只有一个进程用于响应UI，而另外的进程则应当避免与UI接触，因为这会迅速的增加RAM的消耗。一旦UI被绘制，那么几乎就很难将内存的用量降下来。

另外，当运行超过一个进程时，非常重要的一点是，应当使代码尽可能的精简，因为任何不必要的开销都是因为相同的实现被复制到了每个进程中。比如，如果你正在使用枚举([尽管不应该使用枚举](http://android.xsoftlab.net/training/articles/memory.html#Overhead))，所有进程的RAM都需要创建并且初始化这些复制到每个进程中的常量，其它任何的抽象适配器、常量或者其它占用内存的都会被复制。

使用多进程的另外一个担忧就是它们之间的依赖关系。比如，如果APP内含有ContentProvider，并且该ContentProvider运行于显示UI的进程，那么另一个后台进程的代码需要使用这个ContentProvider时，这就需要该UI进程也驻留在RAM中。如果你的目的是一个与UI进程相当权重的后台服务，那么该服务就不可以依赖UI进程中的ContentProvider或者服务。
