原文地址：[http://android.xsoftlab.net/training/articles/perf-tips.html](http://android.xsoftlab.net/training/articles/perf-tips.html)

本篇文章主要介绍那些可以提升整体性能的微小优化点。它与那些能突然改观性能效果的优化手段并不属于同一类。选择正确的算法与数据结构总是我们的第一法则，但是这不是我们这篇文章要介绍的。你应当将这篇文章所提及的知识点作为日常编码的习惯，这可以提升普通代码的运行效率。

下面是书写代码时需要遵守的基本准则：

- 绝不要做你不需要的工作。
- 如果可以不申请内存就不要申请，要合理复用已有的对象。

另一个较复杂的问题就是被优化过的APP肯定是要运行在各种类型的硬件平台上。不同版本的虚拟机运行在不同的处理器上肯定会有不同的运行速度。需要特别说明的是，在模拟器上测试很少会得知其它设备的性能。在不同设备上还有一个很大的不同点就是有没有JIT(**JIT**的意思是即时编译器):含有JIT的最优代码并不总是没有JIT设备的最优代码。

为了确保APP可以在各种类型的设备上运行良好，要确保代码在各个级别的平台上都是高效的。

##避免创建不必要的对象
创建对象绝不会是没有成本的。虽然分代垃圾收集器可以使临时对象的分配成本变得很低，但是内存分配的成本总是远高于非内存分配的成本。

随着更多对象的生成，你可能就开始关注垃圾收集器了。虽然Android 2.3中出现的并发收集器可能会帮到你，但是不必要的工作总是应该避免的。

因此，要避免创建不需要的对象。下面的示例可能会帮到你：

- 如果你有个返回字符串的方法，该方法所返回的字符串总是被接在一个StringBuffer对象后面。那么就可以更改此方法的实现方式：让该字符串直接跟在StringBuffer的后面返回。这样就可以避免创建那些临时性的变量。
- 当从一系列输入数据中提取字符串时，应该尝试返回原始数据的子串，而不是创建一个副本。子串将会创建一个新的String对象，但是它与char[]共用的是相同的数据。采用这种方式的唯一不足就是：虽然使用了其中的一部分数据，但是剩余的数据还都保留在内存中。

一条更为先进的法则就是，将多维数组转换为平行数组使用：

- int数组的效率要比Integer数组的效率高的多。
- 如果你需要实现一个用于存储(Foo,Bar)对象的数组，要记得两个平行的Foo[]，Bar[]数组要比单一的(Foo,Bar)数组效率好太多。

通常来说，要努力避免创建那些生命周期很短的临时变量。更少的对象创建意味着更低频率的垃圾回收，这会直接反应到用户体验上。

##首选静态
如果不需要访问对象的属性，那么就可以将方法设置为静态方法。这样调用将会增加15%-20%的速度。这还是一个好的习惯，因为这样可以告诉其它方法一个信号：它们更改不了对象的状态。

##使用常量
请先考虑以下声明：

```java
static int intVal = 42;
static String strVal = "Hello, world!";
```

编译器会产生出一个类的实例化方法，名为< clinit>，它会在类首次被用到的时候执行。该方法会将值42存到intVal中，并将字符串常量表中的引用赋给strVal。当这些值被引用之后，其它属性才可以访问它们。

我们可以使用"final"关键字来改进一下：

```java
static final int intVal = 42;
static final String strVal = "Hello, world!";
```

这样的话，类就不需要再调用< clinit>方法，因为常量的初始化工作被移入了dex文件中。代码可以直接引用intVal为42的值，并且访问strVal也会直接得到字符串"string constant" ，这样可以省去了查找字符串的过程。

> **Note:** 这样优化手段仅仅适用于基本数据类型以及字符串常量，不要作用其它类型。

##避免内部的get\set方法
像C++这种本地语言通常都会使用get方法来访问属性。这对C++来说是一个非常好的习惯，并且C#、Java等面向对象语言也广泛使用这种方式，因为编译器通常会进行内联访问，并且如果你需要限制访问或者调试属性的话，只需要添加代码就可以。

不过，这在Android上并不是个好习惯。方法调用的开销是非常大的。虽然为了遵循面向对象语言提供get、set方法是合理的，但是在Android中最好是可以直接访问对象的字段。

在没有JIT的设备中，直接访问对象字段的速度要比通过get方法访问的速度快3倍。在含有JIT的设备中，这个效率会达到7倍之多。

注意：如果你使用了ProGuard，那么就有了一个两全其美的结果，因为ProGuard会直接为你进行内联访问。

##使用增强for循环
增强for循环可用于实现了Iterable接口的集合或数组。在集合内部，迭代器需要实现接口方法：hasNext()以及next()。

有以下几种访问数组的方式：
```java
static class Foo {
    int mSplat;
}
Foo[] mArray = ...
public void zero() {
    int sum = 0;
    for (int i = 0; i < mArray.length; ++i) {
        sum += mArray[i].mSplat;
    }
}
public void one() {
    int sum = 0;
    Foo[] localArray = mArray;
    int len = localArray.length;
    for (int i = 0; i < len; ++i) {
        sum += localArray[i].mSplat;
    }
}
public void two() {
    int sum = 0;
    for (Foo a : mArray) {
        sum += a.mSplat;
    }
}
```

zero()方法是最慢的，因为JIT不能够对每次访问数组长度的开销进行优化。

one()方法是稍快点的。它将一切元素放入了本地变量，这样避免了每一次的查询。只有数组的长度提供了明显的性能提升。

two()方法是最快的。它使用了增强for循环。

所以应当在默认情况下使用增强for循环。

> **Tip:** 也可以查看Josh Bloch 的 Effective Java，第46条。

##考虑使用包内访问
请先思考以下类定义：
```java
public class Foo {
    private class Inner {
        void stuff() {
            Foo.this.doStuff(Foo.this.mValue);
        }
    }
    private int mValue;
    public void run() {
        Inner in = new Inner();
        mValue = 27;
        in.stuff();
    }
    private void doStuff(int value) {
        System.out.println("Value is " + value);
    }
}
```

上面的代码定义了一个内部类，它可以直接访问外部类的私有成员以及私有方法。这是正确的，这段代码将会打印出我们所期望的"Value is 27"。

这里的问题是：**VM**会认为Foo$Inner直接访问Foo对象的私有成员是非法的，因为Foo和Foo$Inner是两个不同的类，虽然Java语言允许内部类可以直接访问外部类的私有成员(**PS:虚拟机与语言是两种互不干扰的存在**)。为了弥补这种差异，编译器专门为此生成了一组方法：
```java
/*package*/ static int Foo.access$100(Foo foo) {
    return foo.mValue;
}
/*package*/ static void Foo.access$200(Foo foo, int value) {
    foo.doStuff(value);
}
```

当内部类代码需要访问属性mValue或者调用doStuff()方法时会调用上面这些静态方法。上面的代码归结为你所访问的成员属性都是通过访问器方法访问的。早期我们我说通过访问器访问要比直接访问慢很多，所以这是一段特定语言形成的隐性性能开销示例。

##避免使用浮点型
一般来说，在Android设备上浮点型要比整型慢大概2倍的速度。

在速度方面，float与double并没有什么区别。在空间方面，double是float的两倍大。所以在桌面级设备上，假设空间不是问题，那么我们应当首选double，而不是float。

还有，在对待整型方面，某些处理器擅长乘法，不擅长除法。在这种情况下，整型的除法与取模运算都是在软件中进行的，如果你正在设计一个哈希表或者做其它大量的数学运算的话，这些东西应该考虑到。

##使用本地方法要当心
使用本地代码开发的APP并不一定比Java语言编写的APP高效多少。首先，它会花费在Java-本地代码的转换过程中，并且JIT也不能优化到这些边界。如果你正在申请本地资源，那么对于这些资源的收集能明显的感觉到困难。除此之外，你还需要对每一种CPU架构进行单独编译。你可能甚至还需要为同一个CPU架构编译多个不同的版本：为G1的ARM处理器编译的代码不能运行在Nexus One的ARM处理上，为Nexus One的ARM处理器编译的代码也同样不能运行在G1的ARM处理器上。

本地代码在这种情况下适宜采用：当你有一个已经存在的本地代码库，你希望将它移植到Android上时，不要为了改善Java语言所编写的代码速度而去使用本地代码。

如果你需要使用本地代码，那么应该读一读[JNI Tips](http://android.xsoftlab.net/guide/practices/jni.html).

> **Tip:** 相关信息也可以查看Josh Bloch 的 Effective Java，第54条。

##性能误区
在没有JIT的设备中，通过具体类型的变量调用方法要比抽象接口的调用要高效，这是事实。所以，举个例子，通过HashMap map调用方法要比Map map调用方法要高效的多，开销也少，虽然这两个实现都是HashMap。事实上速度并不会慢2倍这么多；真实的不同大概有6%的减缓。进一步讲，JIT会使两者的差别进一步缩小。

在没有JIT的设备上，通过缓存访问属性要比反复访问属性要快将近20%的速度。在JIT的设备中，属性访问的花销与本地访问的花销基本一致，所以这不是一项有多少价值的优化手段，除非你觉得这样做的话代码更易读(这对static,final,常量同样适用)。

##经常估测
在开始优化之前，要确保你有个问题需要解决：要确保你可以精准测量现有的性能，否则将不能观察到优化所带来的提升。

基准点由[Caliper](http://code.google.com/p/caliper/)的微型基准点框架创建。基准点很难正确获得，所以Caliper将这份很难处理的工作做了，甚至是在你没有在测量那些你想测量的地方的时候。我们强烈的推荐你使用[Caliper](http://code.google.com/p/caliper/)来运行你自己的微型基准点。

你可能还发现Traceview非常有助于提升性能，不过你应该意识到Traceview工作的时候JIT并没有开启。这会错误的认为JIT会将损失掉的时间弥补回来。这尤其重要：根据Traceview所更改的结果会使实际代码运行的更快。


有关更多提升APP性能的工具及方法，请参见以下文档：

   - [Profiling with Traceview and dmtracedump](http://android.xsoftlab.net/tools/debugging/debugging-tracing.html)
   - [Analyzing UI Performance with Systrace](http://android.xsoftlab.net/tools/debugging/systrace.html)