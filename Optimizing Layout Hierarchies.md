原文地址：[http://android.xsoftlab.net/training/improving-layouts/index.html](http://android.xsoftlab.net/training/improving-layouts/index.html)

#引言
布局是直接影响用户体验的关键部分。如果实现的不好，那么布局很有可能会导致内存紧张。Android的SDK包含的一些工具可以用来检查布局性能上的问题。结合本章的课程学习，你将有能力以最小的内存开销实现更为顺畅的UI体验。

#优化布局层级
有一个常见的误解就是使用基本的布局结构会使布局更高效。然而却不是这样的，每一个控件、布局容器都需要执行初始化、排布、绘制等过程。举个例子，使用内嵌的LinearLayout会使布局层级过度加深。进一步讲，内嵌多个使用了layout_weight参数的控件所花费的代价尤其高昂，因为每个子View都需要被测量两次。这当布局被重复加载时尤其重要，比如当使用在ListView或GridView中的时候。

在这节课我们将会学习如何使用[Hierarchy Viewer](http://android.xsoftlab.net/tools/help/hierarchy-viewer.html)工具及[Layoutopt](http://android.xsoftlab.net/tools/help/layoutopt.html)工具来检查、优化布局。

##布局检查
Android的SDK包含了一个名为[Hierarchy Viewer](http://android.xsoftlab.net/tools/help/hierarchy-viewer.html)的工具。使用该工具可以帮助你发现影响布局性能的瓶颈。

[Hierarchy Viewer](http://android.xsoftlab.net/tools/help/hierarchy-viewer.html)工作于你所选择的进程上，它会显示一个布局树。每个View节点上的信号灯代表了该View在测量、排布、绘制上的性能优劣，这可以帮助你发现潜在的问题。

举个例子说明：下图是ListView的一个Item。该Item左边用于显示图片，而右边则显示两行文本。因为该Item会被进行多次加载，所以对其优化的话，那么UI性能会有显著的提升。
![](http://android.xsoftlab.net/images/training/layout-listitem.png)

[Hierarchy Viewer](http://android.xsoftlab.net/tools/help/hierarchy-viewer.html)工具位于< sdk>/tools/目录下。打开后，[Hierarchy Viewer](http://android.xsoftlab.net/tools/help/hierarchy-viewer.html)会列出当前的可用设备以及设备上运行的组件。点击Load View Hierarchy来浏览所选组件的布局层级。下图是上图位于ListView中的运行效果演示：
![](http://android.xsoftlab.net/images/training/hierarchy-linearlayout.png)
在上图中，我们可以看到View的层级为3，并在文本的排布上发现一些问题。点击每个节点我们可以看到每个阶段所花费的时间(如下图所示)。那么我们就可以很清晰的知道哪个Item在测量、排布、渲染上花费的时间最长，所以我们就需要花点时间专门对其优化。
![](http://android.xsoftlab.net/images/training/hierarchy-layouttimes.png)

这里我们可以看到每个阶段所花费的时间：

- Measure: 0.977ms
- Layout: 0.167ms
- Draw: 2.717ms

##调整布局
因为上面的示例说布局的性能慢是由于内嵌了一个LinearLayout，所以改进这部分性能只能通过扁平化来处理。要尽量使布局变浅变宽，杜绝变窄变深。RelativeLayout可以实现这样的布局。所以当使用RelativeLayout实现这样的布局的话，那么我们可以看到布局的层级变为了2。我们所看到的布局图就是这个样子：
![这里写图片描述](http://android.xsoftlab.net/images/training/hierarchy-relativelayout.png)

现在是优化后的时间：

- Measure: 0.598ms
- Layout: 0.110ms
- Draw: 2.146ms

我们可能会看到很微小的改进。

在改进时间上的大部分差别是由于LinearLayout的权重造成的，它会降低测量的速度。这里的示例仅仅是个优化手段的演示，在开发过程中应当认真考虑是否有必要使用权重。

##使用Lint
开发者应该使用lint工具来检查布局层级是否有可优化的地方。Lint 与Layoutopt 相比有更加强大的功能。一些Lint的检查规则如下：

- 使用组合图形 - 一个包含了ImageView和TextView的LinearLayout作为组合图形处理起来更加高效。
- 合并根帧布局 - 如果一个FrameLayout是根布局，并且它没有提供背景色或内边距什么的，那么可以使用合并标签将其替换，这可以稍微的改进性能。
- 无用的叶子节点 - 如果一个布局没有子View，没有背景色，那么通常可以将其移除。
- 无用的中间节点 - 如果一个布局内部只含有一个子View，并且不是ScrollView或者根布局，并且也没有背景色，那么可以将它移除，并将其子View移动到它的父容器内。
- 非常深的布局嵌套 - 一个被嵌套很深的布局通常不利于性能。考虑使用RelativeLayout或者GridLayout这种扁平化布局来改进性能。默认的最大深度为10。

Lint的另一个好处就是它被集成进了Android Studio。Lint会在程序编译时自动运行。

你也可以管理检查Lint的配置，在Android Studio内通过**File>Settings>Project Settings**路径可以找到。

![](http://android.xsoftlab.net/images/tools/studio-inspections-config.png)

Lint可以自动的修复一些问题，并且可以对剩下的问题提供建议以供开发者手动修复。