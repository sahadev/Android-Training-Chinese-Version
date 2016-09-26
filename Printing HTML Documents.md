原文地址：[http://android.xsoftlab.net/training/printing/html-docs.html](http://android.xsoftlab.net/training/printing/html-docs.html)

在Android中打印内容要比打印照片要复杂一些，它要求将文本与图像整合到一个文档中。Android框架提供了一种实现方式，这种方式需要使用HTML来整合文档并打印，实现这个过程仅需要少量的代码。

在Android 4.4及以上的版本中，类[WebView](http://android.xsoftlab.net/reference/android/webkit/WebView.html)也开始可以打印HTML内容。这个类允许你加载本地的HTML资源或者从web上下载一个页面，并可以创建一个打印工作然后将工作传递给Android的打印服务。

这节课展示了如何快速构建一个包含了文本及图形的HTML文档，然后通过[WebView](http://android.xsoftlab.net/reference/android/webkit/WebView.html)将它打印出来。

##加载HTML文档
通过[WebView](http://android.xsoftlab.net/reference/android/webkit/WebView.html)打印HTML文档会涉及到加载HTML资源或者构建一个字符串形式的HTML文档。这一小节描述了如何构建HTML的字符串并且通过[WebView](http://android.xsoftlab.net/reference/android/webkit/WebView.html)将其加载并打印出来。

这个View对象可以作为activity布局的典型用法。然而，如果你的程序没有使用[WebView](http://android.xsoftlab.net/reference/android/webkit/WebView.html)，那么你可以创建一个这个类的实例，然后专门用于打印。创建自定义打印的主要步骤有：

- 1 .创建一个[WebViewClient](http://android.xsoftlab.net/reference/android/webkit/WebViewClient.html)，它用于在HTML资源加载完毕之后启动打印工作。
- 2 .加载HTML资源到[WebView](http://android.xsoftlab.net/reference/android/webkit/WebView.html)对象中。

下面的代码演示了如何创建一个简要的[WebViewClient](http://android.xsoftlab.net/reference/android/webkit/WebViewClient.html)，以及如何加载动态创建的HTML文档：
```java
private WebView mWebView;
private void doWebViewPrint() {
    // Create a WebView object specifically for printing
    WebView webView = new WebView(getActivity());
    webView.setWebViewClient(new WebViewClient() {
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                return false;
            }
            @Override
            public void onPageFinished(WebView view, String url) {
                Log.i(TAG, "page finished loading " + url);
                createWebPrintJob(view);
                mWebView = null;
            }
    });
    // Generate an HTML document on the fly:
    String htmlDocument = "<html><body><h1>Test Content</h1><p>Testing, " +
            "testing, testing...</p></body></html>";
    webView.loadDataWithBaseURL(null, htmlDocument, "text/HTML", "UTF-8", null);
    // Keep a reference to WebView object until you pass the PrintDocumentAdapter
    // to the PrintManager
    mWebView = webView;
}
```

> **Note:** 要确保所产生的打印工作是在[WebViewClient](http://android.xsoftlab.net/reference/android/webkit/WebViewClient.html)的[onPageFinished()](http://android.xsoftlab.net/reference/android/webkit/WebViewClient.html#onPageFinished(android.webkit.WebView,%20java.lang.String))方法中被调用的。如果没有等到页面加载完毕，那么所打印出的内容可能是不完整或者空白的，甚至可能会完全失败。

> **Note:** 上面的示例代码保持了一个WebView对象的引用，所以在打印工作创建之前不会被垃圾回收器回收。要确保在你自己的实现中做了相同的工作，否则打印进程可能会失败。

如果你想在这个页面中包含图像，请将图像文件放置到工程的assets/目录下，然后在[loadDataWithBaseURL()](http://android.xsoftlab.net/reference/android/webkit/WebView.html#loadDataWithBaseURL(java.lang.String,%20java.lang.String,%20java.lang.String,%20java.lang.String,%20java.lang.String))方法的第一个参数中指定这个图像的URL，就像下面的代码展示的那样：
```java
webView.loadDataWithBaseURL("file:///android_asset/images/", htmlBody,
        "text/HTML", "UTF-8", null);
```

你也可以加载一个Web页来打印，不过这里不是使用[loadDataWithBaseURL()](http://android.xsoftlab.net/reference/android/webkit/WebView.html#loadDataWithBaseURL(java.lang.String,%20java.lang.String,%20java.lang.String,%20java.lang.String,%20java.lang.String))，而是使用[loadUrl()](http://android.xsoftlab.net/reference/android/webkit/WebView.html#loadUrl(java.lang.String))，就像下面这样：
```java
// Print an existing web page (remember to request INTERNET permission!):
webView.loadUrl("http://developer.android.com/about/index.html");
```

当使用WebView来创建打印文档时，你应该意识到以下这些限制：

- 不能为文档添加页眉、页脚，包括页码。
- HTML文档的打印选项不包括打印页码范围的能力，举个例子：要打印10页文档的第2页到第4页，这时就不支持了。
- 在同一时间只能有一个WebView的实例处理打印工作。
- 一个HTML文档会包含CSS打印属性，比如横向打印属性，这并不支持。
- 不能使用HTML文档内部的JavaScript来启动打印。

> **Note:** 布局中所包含的WebView的内容在加载完文档之后可以被打印一次。

如果你想创建一个有更多选项的打印及打印页上完整的控制功能，请参见下节课程： [Printing a Custom Document](http://android.xsoftlab.net/training/printing/custom-docs.html).

##创建打印工作
在创建WebView并加载完毕HTML文档之后，你的程序几乎完成了打印所要做的所有工作。下面的步骤就需要访问[PrintManager](http://android.xsoftlab.net/reference/android/print/PrintManager.html)，创建一个打印适配器，到最后创建一个打印工作。下面的代码演示了如何执行这些步骤：
```java
private void createWebPrintJob(WebView webView) {
    // Get a PrintManager instance
    PrintManager printManager = (PrintManager) getActivity()
            .getSystemService(Context.PRINT_SERVICE);
    // Get a print adapter instance
    PrintDocumentAdapter printAdapter = webView.createPrintDocumentAdapter();
    // Create a print job with name and adapter instance
    String jobName = getString(R.string.app_name) + " Document";
    PrintJob printJob = printManager.print(jobName, printAdapter,
            new PrintAttributes.Builder().build());
    // Save the job object for later status checking
    mPrintJobs.add(printJob);
}
```

这个例子在程序的内部保存了一个[PrintJob](http://android.xsoftlab.net/reference/android/print/PrintJob.html)对象的实例，不过这不是必须的。你的程序可能会在工作开始之后需要使用这个对象来追踪打印工作的进度。这适用于在程序内部监视打印工作的状态，比如完成、失败或者是用户取消。创建内置的通知并不是必须的，因为打印框架会为打印工作自动的创建一个系统通知。