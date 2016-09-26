原文地址：[http://android.xsoftlab.net/training/custom-views/custom-drawing.html#draw](http://android.xsoftlab.net/training/custom-views/custom-drawing.html#draw)

自定义View最重要的部分就是它的外观了。自定义绘制根据程序的需要或者简单亦或者复杂。这节课的内容涵盖了大多数通用的知识点。

##重写[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法
绘制自定义View很重要的一个步骤就是重写它的[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法。该方法含有一个[Canvas](http://android.xsoftlab.net/reference/android/graphics/Canvas.html)对象作为参数，用来是View绘制它本身。[Canvas](http://android.xsoftlab.net/reference/android/graphics/Canvas.html)类定义了用于绘制文本，线条，位图以及许多其它基础的物理图形。你可以在[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法中使用这些方法来创建属于你自己的UI效果。

在开始任何绘制之前，必须先创建一个[Paint](http://android.xsoftlab.net/reference/android/graphics/Paint.html)对象。后面的部分会讨论[Paint](http://android.xsoftlab.net/reference/android/graphics/Paint.html)的相关知识。

##创建绘制对象
[android.graphics](http://android.xsoftlab.net/reference/android/graphics/package-summary.html)框架将绘制分为了两块区域：

- 要绘制什么，由Canvas控制
- 如何绘制，由Paint控制

举个例子，Canvas提供了用于绘制线条的方法，而Paint则定义了线条的颜色。Canvas拥有绘制矩形的方法，而Paint则定义了是否要使颜色填充这个矩形。简而言之，Canvas定义了你绘制在屏幕上的形状，而Paint则定义了这些形状的颜色、风格以及字体等等。

所以，在开始绘制任何事物之前，你需要创建一个多个Paint对象。示例PieChart将这些工作放在了一个名为init的方法中，它由构造方法调用：
```java
private void init() {
   mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mTextPaint.setColor(mTextColor);
   if (mTextHeight == 0) {
       mTextHeight = mTextPaint.getTextSize();
   } else {
       mTextPaint.setTextSize(mTextHeight);
   }
   mPiePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mPiePaint.setStyle(Paint.Style.FILL);
   mPiePaint.setTextSize(mTextHeight);
   mShadowPaint = new Paint(0);
   mShadowPaint.setColor(0xff101010);
   mShadowPaint.setMaskFilter(new BlurMaskFilter(8, BlurMaskFilter.Blur.NORMAL));
   ...
```

提前创建对象是一项非常重要的优化手段。View会很频繁的绘制，并且很多绘制对象的创建代价非常高昂。在onDraw()方法中创建绘制对象会明显的降低应用性能，并可能会导致UI出现停顿。

##处理布局事件
为了可以正确的绘制View，首先需要知道View的尺寸。复杂一点的View经常需要执行多次布局计算。你永远不要假设View的尺寸，甚至是只有一款应用使用你的View。APP需要处理不同的屏幕尺寸，不同的屏幕密度，以及在垂直模式及水平模式下不同的高宽比。

尽管View拥有很多测量尺寸的方法，但是绝大多数是不需要重写的。如果View不需要特别控制它的尺寸，你只需要重写一个方法：[onSizeChanged()](http://android.xsoftlab.net/reference/android/view/View.html#onSizeChanged(int,%20int,%20int,%20int)).

[onSizeChanged()](http://android.xsoftlab.net/reference/android/view/View.html#onSizeChanged(int,%20int,%20int,%20int))方法会在首次分配尺寸的时候调用，如果尺寸再次变更时则会再次调用。我们在[onSizeChanged()](http://android.xsoftlab.net/reference/android/view/View.html#onSizeChanged(int,%20int,%20int,%20int))方法中计算View的位置、尺寸以及其它任何与尺寸相关的值，而不是在每次绘制的时候重新计算它们。在示例PieChart中，[onSizeChanged()](http://android.xsoftlab.net/reference/android/view/View.html#onSizeChanged(int,%20int,%20int,%20int))方法内部计算了饼图的矩形边框以及文本和其它可视元素的相对位置。

当View被分配了一个尺寸时，布局管理器会假设该尺寸包含了View的所有内边距。所以在计算View的尺寸时要将View的内边距计算在内。下面的代码段展示了PieChart.onSizeChanged()是如何做的：
```java
       // Account for padding
       float xpad = (float)(getPaddingLeft() + getPaddingRight());
       float ypad = (float)(getPaddingTop() + getPaddingBottom());
       // Account for the label
       if (mShowText) xpad += mTextWidth;
       float ww = (float)w - xpad;
       float hh = (float)h - ypad;
       // Figure out how big we can make the pie.
       float diameter = Math.min(ww, hh);
```

如果你想更精细的控制View布局的参数，请实现[onMeasure()](http://android.xsoftlab.net/reference/android/view/View.html#onMeasure(int,%20int))方法。这个方法的两个参数都为[View.MeasureSpec](http://android.xsoftlab.net/reference/android/view/View.MeasureSpec.html)。它们用于告诉View的父布局希望View是多大，这个View的尺寸是最大值还是只是一个建议值等等。随着优化，这些值被存放在一个被包装后的整型值内，你必须使用[View.MeasureSpec](http://android.xsoftlab.net/reference/android/view/View.MeasureSpec.html)的一个静态方法来解压存储在这个整型值内的信息。

下面是[onMeasure()](http://android.xsoftlab.net/reference/android/view/View.html#onMeasure(int,%20int))方法的一个示例。在这个实现中，PieCart尝试将自己的区域扩大到内部标签的大小。
```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   // Try for a width based on our minimum
   int minw = getPaddingLeft() + getPaddingRight() + getSuggestedMinimumWidth();
   int w = resolveSizeAndState(minw, widthMeasureSpec, 1);
   // Whatever the width ends up being, ask for a height that would let the pie
   // get as big as it can
   int minh = MeasureSpec.getSize(w) - (int)mTextWidth + getPaddingBottom() + getPaddingTop();
   int h = resolveSizeAndState(MeasureSpec.getSize(w) - (int)mTextWidth, heightMeasureSpec, 0);
   setMeasuredDimension(w, h);
}
```

这里有三个非常重要的点需要关注：

- 这个公式将View的内边距一并计算在内。正如前面所说的，这是View本身的职责。
- 辅助方法[resolveSizeAndState()](http://android.xsoftlab.net/reference/android/view/View.html#resolveSizeAndState(int,%20int,%20int))用于创建最终的宽度值与高度值。这个辅助方法通过比较View的期望值与[onMeasure()](http://android.xsoftlab.net/reference/android/view/View.html#onMeasure(int,%20int))回调的spec值，返回了一个适当的[View.MeasureSpec](http://android.xsoftlab.net/reference/android/view/View.MeasureSpec.html)值。
- [onMeasure()](http://android.xsoftlab.net/reference/android/view/View.html#onMeasure(int,%20int))没有返回值。相反的，该方法通过调用setMeasuredDimension()方法与外界交流。调用这个方法是强制要求的。如果你忽略了这个调用，那么View类会抛出一个运行时异常。

##绘制
一旦完成对象创建代码与尺寸测量代码之后，接下来就可以实现[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法了。每个View的[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法都不相同，但是它们还是有一些相同特点存在的：

- 使用[drawText()](http://android.xsoftlab.net/reference/android/graphics/Canvas.html#drawText(char[],%20int,%20int,%20float,%20float,%20android.graphics.Paint))方法绘制文本。通过[setTypeface()](http://android.xsoftlab.net/reference/android/graphics/Paint.html#setTypeface(android.graphics.Typeface))方法指定字体类型，通过[setColor()](http://android.xsoftlab.net/reference/android/graphics/Paint.html#setColor(int))方法设置文本颜色。
- 通过[drawRect()](http://android.xsoftlab.net/reference/android/graphics/Canvas.html#drawRect(android.graphics.Rect,%20android.graphics.Paint))、[drawOval()](http://android.xsoftlab.net/reference/android/graphics/Canvas.html#drawOval(android.graphics.RectF,%20android.graphics.Paint))和[drawArc()](http://android.xsoftlab.net/reference/android/graphics/Canvas.html#drawArc(android.graphics.RectF,%20float,%20float,%20boolean,%20android.graphics.Paint))绘制基础图形。通过[setStyle()](http://android.xsoftlab.net/reference/android/graphics/Paint.html#setStyle(android.graphics.Paint.Style))更改图形是填充模式还是绘制轮廓模式，或者两者都不是。
- 通过Path类来绘制更加复杂的图形。通过给[Path](http://android.xsoftlab.net/reference/android/graphics/Path.html)对象添加线条及曲线来定义形状，然后通过[drawPath()](http://android.xsoftlab.net/reference/android/graphics/Canvas.html#drawPath(android.graphics.Path,%20android.graphics.Paint))方法绘制形状。只是这些基础形状可以通过[setStyle()](http://android.xsoftlab.net/reference/android/graphics/Paint.html#setStyle(android.graphics.Paint.Style))方法定义它们的路径风格。
- 通过创建[LinearGradient](http://android.xsoftlab.net/reference/android/graphics/LinearGradient.html)对象定义梯度填充模式。调用[setShader()](http://android.xsoftlab.net/reference/android/graphics/Paint.html#setShader(android.graphics.Shader))方法来填充形状。
- 通过[drawBitmap()](http://android.xsoftlab.net/reference/android/graphics/Canvas.html#drawBitmap(android.graphics.Bitmap,%20android.graphics.Matrix,%20android.graphics.Paint))方法绘制位图。

举个例子，下面的代码用来绘制PieChart。它混合使用了文本、线条以及形状。
```java
protected void onDraw(Canvas canvas) {
   super.onDraw(canvas);
   // Draw the shadow
   canvas.drawOval(
           mShadowBounds,
           mShadowPaint
   );
   // Draw the label text
   canvas.drawText(mData.get(mCurrentItem).mLabel, mTextX, mTextY, mTextPaint);
   // Draw the pie slices
   for (int i = 0; i < mData.size(); ++i) {
       Item it = mData.get(i);
       mPiePaint.setShader(it.mShader);
       canvas.drawArc(mBounds,
               360 - it.mEndAngle,
               it.mEndAngle - it.mStartAngle,
               true, mPiePaint);
   }
   // Draw the pointer
   canvas.drawLine(mTextX, mPointerY, mPointerX, mPointerY, mTextPaint);
   canvas.drawCircle(mPointerX, mPointerY, mPointerSize, mTextPaint);
}
```

