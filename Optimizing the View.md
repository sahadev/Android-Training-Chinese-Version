原文地址：[http://android.xsoftlab.net/training/custom-views/optimizing-view.html](http://android.xsoftlab.net/training/custom-views/optimizing-view.html)

现在已经完成了一个拥有良好设计的View，它即可以响应手势，又可以在状态之间过渡。为了避免View在感觉上卡顿，要确保动画始终是每秒60帧的频率。

##尽可能的降低频率
为了使View流畅，要从调用频繁的方法中消除不必要的代码。首先从[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法开始，在该方法中可以带来可观的效果回报。特别的应该移除[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法中的内存分配代码，因为内存分配会导致垃圾回收，这可能会导致程序暂停。应该在程序初始化的时候申请内存，绝不要在动画运行的时候申请内存。

除了精简[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法之外，还应该确保降低这些方法的调用频率。[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法的大部分调用由[invalidate()](http://android.xsoftlab.net/reference/android/view/View.html#invalidate())方法导致，所以要移除不必要的[invalidate()](http://android.xsoftlab.net/reference/android/view/View.html#invalidate())调用代码。

另一项代价非常高昂的操作就是布局的测量。每次调用[requestLayout()](http://android.xsoftlab.net/reference/android/view/View.html#requestLayout())，Android的UI系统都需要测量整个View层级来找出每个View都需要多大的空间。如果找到尺寸有冲突的，还需要进行多次测量。UI设计者有时候需要创建内嵌[ViewGroup](http://android.xsoftlab.net/reference/android/view/ViewGroup.html)的深层级布局来使UI布局正确。这些深层级的布局层级会引起性能问题。要使View层级尽可能的潜。

如果你有一个稍微复杂一点的UI，考虑写一个自定义[ViewGroup来](http://android.xsoftlab.net/reference/android/view/ViewGroup.html)执行这样的布局。与内置的View不同，你的自定义View可以对它的子View的尺寸与形状作个假设，这样就可以不用去测量子View的尺寸了。饼图示例展示了如何继承ViewGroup作为自定义View的一部分。饼图含有一些子View，但是决不会去测量它们。相反的，它在自定义布局的内部直接设置了这些View的尺寸。