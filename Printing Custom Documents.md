原文地址：[http://android.xsoftlab.net/training/printing/custom-docs.html](http://android.xsoftlab.net/training/printing/custom-docs.html)

对于一些应用，比如绘图类APP，版面设计类APP以及其它APP，这些APP都关注图形的输出，有一个漂亮的打印页面是它们的关键特性。在这种情况下，就不单单是打印一张图片或者是HTML文档这么简单了。这些程序对于这种类型的打印需要对页面中每样事物的控制都特别的精细，包括字体、文本流、页面间距、页眉、页脚以及图形元素。

创建打印输出对于程序来说是完全自定义的，这需要更多设计上的投入，就像上面讨论的那样。你必须构建一些可以与打印框架交流的组件，并且还可以用来调整打印设置，绘制页面元素及管理多个页面的打印。

这节课展示了如何与打印管理者进行连接、创建打印适配器和构建打印内容。

##连接打印管理者
当程序需要直接管理打印进程时，在收到用户的打印请求之后，第一步就是连接Android的打印框架，以及操作[PrintManager](http://android.xsoftlab.net/reference/android/print/PrintManager.html)类的实例。这个类允许你实例化一个打印工作并开始打印的生命过程。下面的代码展示了如何获得一个打印管理者和启动打印进程。
```java
private void doPrint() {
    // Get a PrintManager instance
    PrintManager printManager = (PrintManager) getActivity()
            .getSystemService(Context.PRINT_SERVICE);
    // Set job name, which will be displayed in the print queue
    String jobName = getActivity().getString(R.string.app_name) + " Document";
    // Start a print job, passing in a PrintDocumentAdapter implementation
    // to handle the generation of a print document
    printManager.print(jobName, new MyPrintDocumentAdapter(getActivity()),
            null); //
}
```

上面的代码演示了如何命名一个打印工作并设置一个[PrintDocumentAdapter](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html)的实例，这个对象可以处理打印过程的每一个步骤。打印适配器的实现会在下面的章节中讨论到。

> **Note:** [print()](http://android.xsoftlab.net/reference/android/print/PrintManager.html#print(java.lang.String,%20android.print.PrintDocumentAdapter,%20android.print.PrintAttributes))方法的最后一个参数需要一个[PrintAttributes](http://android.xsoftlab.net/reference/android/print/PrintAttributes.html)对象。你可以使用这个参数给打印框架提供一些提示及给原先的打印周期预先设置一些选项，这可以改进用户体验。你还可以使用这个参数来设置一些选项，这些选项更适用于内容的打印，比如当打印照片的时候可以设置打印的方向为照片本身的方向。

##创建打印适配器
打印适配器会与Android的打印框架相连接，并会处理打印过程的每一个步骤。这个过程要求用户在创建文档打印之前选择打印机及相关的打印选项。这些过程会影响最终的输出结果，就像用户选择了不同打印能力，不同的页面尺寸，不同的页面方向一样。随着这些选项的设置，打印框架会要求适配器展示并生成一个打印文稿，为最终的打印做准备。一旦用户按下了打印按钮，打印框架会拿到最终的打印文档然后交付给打印提供者以便打印。在打印的过程中，用户可以选择取消打印行为，所以打印适配器必须监听并响应取消请求。

抽象类PrintDocumentAdapter被设计为用来处理打印过程的生命周期，它拥有4个主要的回调方法。你必须在打印适配器中实现这些方法，以便可以与打印框架进行适当的交互：

- [onStart()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html#onStart()) - 会在打印进程开始的时候调用一次，如果应用对任务有任何的单次预处理任务，比如获取要打印的数据段，就可以在这里执行。实现这个方法并不是必须要求的。
- [onLayout()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes,%20android.print.PrintAttributes,%20android.os.CancellationSignal,%20android.print.PrintDocumentAdapter.LayoutResultCallback,%20android.os.Bundle)) - 会在用户每次更改打印设置的时候调用一次，这会影响到最终的输出结果，比如不同的页面尺寸，或者页面方向，提供给程序一个机会来估算页面的版面。在最低限度下，这个方法必须返回将要打印的文档有多少页。
- [onWrite()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html#onWrite(android.print.PageRange[],%20android.os.ParcelFileDescriptor,%20android.os.CancellationSignal,%20android.print.PrintDocumentAdapter.WriteResultCallback)) - 该方法被用来将要打印的页面作用到一个文件中，然后再被打印。这个方法可能会在onLayout()方法每次调用之后被调用一次或者多次，
- [onFinish()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html#onFinish()) - 该方法会在打印过程结束的时候调用一次。如果程序对打印任务需要任何的销毁工作，那可以在这里执行。这个方法不是必须要求被实现的。

下面的章节会描述如何实现layout和write方法，这两个方法实现了打印适配器的决定性功能。

> **Note:** 适配器方法会在程序的主线程中被调用。如果你认为执行这些方法会消耗大量时间的话，那么可以在单独的线程中实现它们。举个例子，你可以将layout或者打印文档的writing工作放入单独的AsyncTask对象中。

###计算文档信息
在[PrintDocumentAdapter](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html)的实现中，程序必须指定文档的类型，它还需要创建并计算打印工作的总页数，获得被打印页的尺寸信息。onLayout()方法应该进行这些计算并且将要输出的文档信息放入一个[PrintDocumentInfo](http://android.xsoftlab.net/reference/android/print/PrintDocumentInfo.html)对象中，包括页面数量以及内容类型。下面的代码展示onLayout()方法的最基础实现：
```java
@Override
public void onLayout(PrintAttributes oldAttributes,
                     PrintAttributes newAttributes,
                     CancellationSignal cancellationSignal,
                     LayoutResultCallback callback,
                     Bundle metadata) {
    // Create a new PdfDocument with the requested page attributes
    mPdfDocument = new PrintedPdfDocument(getActivity(), newAttributes);
    // Respond to cancellation request
    if (cancellationSignal.isCancelled() ) {
        callback.onLayoutCancelled();
        return;
    }
    // Compute the expected number of printed pages
    int pages = computePageCount(newAttributes);
    if (pages > 0) {
        // Return print information to print framework
        PrintDocumentInfo info = new PrintDocumentInfo
                .Builder("print_output.pdf")
                .setContentType(PrintDocumentInfo.CONTENT_TYPE_DOCUMENT)
                .setPageCount(pages);
                .build();
        // Content layout reflow is complete
        callback.onLayoutFinished(info, true);
    } else {
        // Otherwise report an error to the print framework
        callback.onLayoutFailed("Page count calculation failed.");
    }
}
```

[onLayout()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes,%20android.print.PrintAttributes,%20android.os.CancellationSignal,%20android.print.PrintDocumentAdapter.LayoutResultCallback,%20android.os.Bundle))的执行会有三个结果：完成、取消或者失败，失败的情况就是说不能够完成版面的计算。你必须通过调用[PrintDocumentAdapter.LayoutResultCallback](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.LayoutResultCallback.html)对象的适当方法来指定其中一个结果。

> **Note:** [onLayoutFinished()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.LayoutResultCallback.html#onLayoutFinished(android.print.PrintDocumentInfo,%20boolean))方法的布尔参数指示了从最后一次请求开始版面的内容是否有实质上的改变。适当的设置这个参数可以允许打印框架避免对[onWrite()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html#onWrite(android.print.PageRange[],%20android.os.ParcelFileDescriptor,%20android.os.CancellationSignal,%20android.print.PrintDocumentAdapter.WriteResultCallback))方法进行不必要的调用，实质上会缓存原先的书面打印文档并改善性能。

[onLayout()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes,%20android.print.PrintAttributes,%20android.os.CancellationSignal,%20android.print.PrintDocumentAdapter.LayoutResultCallback,%20android.os.Bundle))的主要工作是计算页码，这个页面会作为打印机的输出属性。至于如何计算页码这高度依赖程序如何排版打印页。下面的代码展示了一个实现，这个实现的页码取决于打印的方向：
```java
private int computePageCount(PrintAttributes printAttributes) {
    int itemsPerPage = 4; // default item count for portrait mode
    MediaSize pageSize = printAttributes.getMediaSize();
    if (!pageSize.isPortrait()) {
        // Six items per page in landscape orientation
        itemsPerPage = 6;
    }
    // Determine number of print items
    int printItemCount = getPrintItemCount();
    return (int) Math.ceil(printItemCount / itemsPerPage);
}
```

###写入打印文档文件
当写入打印结果到文件中时，Android打印框架会调用[onWrite()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html#onWrite(android.print.PageRange[],%20android.os.ParcelFileDescriptor,%20android.os.CancellationSignal,%20android.print.PrintDocumentAdapter.WriteResultCallback))方法。这个方法的参数指明了哪一页需要被写入以及被使用到的输出文件。你的实现必须将每个请求内容页渲染到一个多页的PDF文档文件中。这个过程完成以后，你需要调用回调对象的[onWriteFinished()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.WriteResultCallback.html#onWriteFinished(android.print.PageRange[]))方法。

> **Note:** 由于Android打印框架可能会在每次调用onLayout()方法之后调用若干次onWrite()方法，所以在打印页面并没有发生实质上的改变时设置onLayoutFinished()方法的布尔参数为false是很重要的，这样可以避免对打印文档进行不必要的重复写入。


> **Note:** [onLayoutFinished()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.LayoutResultCallback.html#onLayoutFinished(android.print.PrintDocumentInfo,%20boolean))方法的布尔参数指示了从最后一次请求开始版面的内容是否有实质上的改变。适当的设置这个参数可以允许打印框架避免对[onWrite()](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.html#onWrite(android.print.PageRange[],%20android.os.ParcelFileDescriptor,%20android.os.CancellationSignal,%20android.print.PrintDocumentAdapter.WriteResultCallback))方法进行不必要的调用，实质上会缓存原先的书面打印文档并改善性能。

下面简要演示了使用PrintedPdfDocument类创建PDF文件过程的基本技术细节：
```java
@Override
public void onWrite(final PageRange[] pageRanges,
                    final ParcelFileDescriptor destination,
                    final CancellationSignal cancellationSignal,
                    final WriteResultCallback callback) {
    // Iterate over each page of the document,
    // check if it's in the output range.
    for (int i = 0; i < totalPages; i++) {
        // Check to see if this page is in the output range.
        if (containsPage(pageRanges, i)) {
            // If so, add it to writtenPagesArray. writtenPagesArray.size()
            // is used to compute the next output page index.
            writtenPagesArray.append(writtenPagesArray.size(), i);
            PdfDocument.Page page = mPdfDocument.startPage(i);
            // check for cancellation
            if (cancellationSignal.isCancelled()) {
                callback.onWriteCancelled();
                mPdfDocument.close();
                mPdfDocument = null;
                return;
            }
            // Draw page content for printing
            drawPage(page);
            // Rendering is complete, so page can be finalized.
            mPdfDocument.finishPage(page);
        }
    }
    // Write PDF document to file
    try {
        mPdfDocument.writeTo(new FileOutputStream(
                destination.getFileDescriptor()));
    } catch (IOException e) {
        callback.onWriteFailed(e.toString());
        return;
    } finally {
        mPdfDocument.close();
        mPdfDocument = null;
    }
    PageRange[] writtenPages = computeWrittenPages();
    // Signal the print framework the document is complete
    callback.onWriteFinished(writtenPages);
    ...
}
```

这个示例将PDF页的内容委派给了drawPage()方法，这会在下面的章节中讨论。

和layout一样，onWrite()的执行过程也有三个结果：完成、取消或是失败。在失败情况下不能再写入内容。你必须通过[PrintDocumentAdapter.WriteResultCallback](http://android.xsoftlab.net/reference/android/print/PrintDocumentAdapter.WriteResultCallback.html)对象的适当方法指明结果。

> **Note:** 文档打印的过程是个资源密集型的操作。为了避免阻塞UI线程，你应该考虑在单独的线程中执行这些事情，比如在[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)中。有关更多关于比如异步任务的工作执行线程，请参见 [Processes and Threads](http://android.xsoftlab.net/guide/components/processes-and-threads.html)。

##绘制PDF页面内容
当程序要打印时，程序必须先生成一个PDF文档，然后将文档交付给Android打印框架来打印。你可以使用任何的PDF生成库来实现这个目的。这节课展示了如何使用[PrintedPdfDocument](http://android.xsoftlab.net/reference/android/print/pdf/PrintedPdfDocument.html)类来生成PDF页。

[PrintedPdfDocument](http://android.xsoftlab.net/reference/android/print/pdf/PrintedPdfDocument.html)类使用了一个[Canvas](http://android.xsoftlab.net/reference/android/graphics/Canvas.html)对象来绘制元素到PDF页上，这与Activity的布局绘制很类似。你可以使用Canvas的绘制方法来绘制页面元素。下面的代码演示了如何使用这些方法来绘制一些简单的元素到PDF文档页上:
```java
private void drawPage(PdfDocument.Page page) {
    Canvas canvas = page.getCanvas();
    // units are in points (1/72 of an inch)
    int titleBaseLine = 72;
    int leftMargin = 54;
    Paint paint = new Paint();
    paint.setColor(Color.BLACK);
    paint.setTextSize(36);
    canvas.drawText("Test Title", leftMargin, titleBaseLine, paint);
    paint.setTextSize(11);
    canvas.drawText("Test paragraph", leftMargin, titleBaseLine + 25, paint);
    paint.setColor(Color.BLUE);
    canvas.drawRect(100, 100, 172, 172, paint);
}
```

当使用Canvas绘制PDF页面时，元素由一些点来指定位置，这个点的大小是英寸的72分之一。要确保使用这个测量单位来指明元素的尺寸。对于绘制元素的定位，坐标系统会从页面的左上角0,0点开始。

**Tip:** 虽然Canvas对象允许你将打印元素放置到PDF文档的边上，但很多打印机并没有能力可以将边上的元素打印到纸上去。要确保在使用这个类构建打印文档时要保留一定的页面边距。