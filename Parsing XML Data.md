原文地址：[http://android.xsoftlab.net/training/basics/network-ops/xml.html](http://android.xsoftlab.net/training/basics/network-ops/xml.html)

扩展标记语言(XML)是一系列有序编码的文档。它是一种很受欢迎的互联网数据传输格式。像需要频繁更新内容的网站，比如新闻站点或者博客，需要经常更新它们的XML源，以使外部程序可以保持内容的同步变化。对于含有网络连接态的APP应用来说，上传及解析XML数据是一个通用的任务。这节课将会学习如何解析XML文档及如何使用XML中的数据。

##选择解析器
我们推荐[XmlPullParser](http://android.xsoftlab.net/reference/org/xmlpull/v1/XmlPullParser.html)解析器，在Android上它是一种高效的可维护的解析方式。Android中含有该接口的两个实现：

 - [KXmlParser](http://kxml.sourceforge.net/)由[XmlPullParserFactory.newPullParser()](http://android.xsoftlab.net/reference/org/xmlpull/v1/XmlPullParserFactory.html#newPullParser())获得。
 - ExpatPullParser由[Xml.newPullParser()](http://android.xsoftlab.net/reference/android/util/Xml.html#newPullParser())获得。

每个选择都可以。这里所使用的是ExpatPullParser。

##分析源
解析源的第一步就是判断哪种属性是你所关注的。解析器会将这些属性所对应的数据提取出，将剩余的部分忽略。

下面是示例APP中的部分摘录。每个实例都是以entry的形式出现在源里，并且entry会包含若干个嵌套标签。
```xml
<?xml version="1.0" encoding="utf-8"?> 
<feed xmlns="http://www.w3.org/2005/Atom" xmlns:creativeCommons="http://backend.userland.com/creativeCommonsRssModule" ...">     
<title type="text">newest questions tagged android - Stack Overflow</title>
...
    <entry>
    ...
    </entry>
    <entry>
        <id>http://stackoverflow.com/q/9439999</id>
        <re:rank scheme="http://stackoverflow.com">0</re:rank>
        <title type="text">Where is my data file?</title>
        <category scheme="http://stackoverflow.com/feeds/tag?tagnames=android&sort=newest/tags" term="android"/>
        <category scheme="http://stackoverflow.com/feeds/tag?tagnames=android&sort=newest/tags" term="file"/>
        <author>
            <name>cliff2310</name>
            <uri>http://stackoverflow.com/users/1128925</uri>
        </author>
        <link rel="alternate" href="http://stackoverflow.com/questions/9439999/where-is-my-data-file" />
        <published>2012-02-25T00:30:54Z</published>
        <updated>2012-02-25T00:30:54Z</updated>
        <summary type="html">
            <p>I have an Application that requires a data file...</p>
        </summary>
    </entry>
    <entry>
    ...
    </entry>
...
</feed>
```

示例APP中从entry中提出的数据包含title, link, summary等标签。

##实例化解析器
接下来这一步就是实例化解析器并启动解析过程。在下面的代码段中，解析器被实例化为不处理命名空间，并且将[InputStream](http://android.xsoftlab.net/reference/java/io/InputStream.html)作为其输入来源。它通过调用nextTag()方法来启动解析过程，调用readFeed()方法来提取并处理APP所关心的数据。
```java
public class StackOverflowXmlParser {
    // We don't use namespaces
    private static final String ns = null;
   
    public List parse(InputStream in) throws XmlPullParserException, IOException {
        try {
            XmlPullParser parser = Xml.newPullParser();
            parser.setFeature(XmlPullParser.FEATURE_PROCESS_NAMESPACES, false);
            parser.setInput(in, null);
            parser.nextTag();
            return readFeed(parser);
        } finally {
            in.close();
        }
    }
 ... 
}
```

##读取源
readFeed()方法开始对源进行实际操作。它会寻找元素标签"entry"并将其作为递归处理的起点。如果标签不是entry，则跳过。一旦整个源完成了递归处理，那么readFeed()则会返回一个List，它内部包含了entry对应的数据及内嵌的数据成员。
```java
private List readFeed(XmlPullParser parser) throws XmlPullParserException, IOException {
    List entries = new ArrayList();
    parser.require(XmlPullParser.START_TAG, ns, "feed");
    while (parser.next() != XmlPullParser.END_TAG) {
        if (parser.getEventType() != XmlPullParser.START_TAG) {
            continue;
        }
        String name = parser.getName();
        // Starts by looking for the entry tag
        if (name.equals("entry")) {
            entries.add(readEntry(parser));
        } else {
            skip(parser);
        }
    }  
    return entries;
}
```

##解析XML
XML的解析包含以下步骤：

1.像Analyze the Feed中所描述的那样，需要确定你所需要的标签。这个示例从entry及其内嵌标签title, link, 和 summary中提取数据。

2.创建以下方法：

 - 对每个标签创建一个read方法，用于读取你所关注的标签。 比如readEntry(), readTitle(), 等等等等。解析器会从输入流中读取标签。当它探测到标签名为entry, title, link或summary时，它就会调用该标签所对应的方法。否则会跳过该标签。
 - 每个提取数据的方法会将解析器移动至下一个标签。比如：
 
   -  对于title及summary标签，解析器会调用readText()方法。该方法通过parser.getText()方法提取相应标签所对应的数据。
   -  对于link标签，解析器首先检查该link是否是我们所关注的，然后才会通过parser.getAttributeValue()方法将link的值提取出。
   -  对于entry标签，解析器会调用readEntry()方法。这个方法会解析entry的内部标签，并返回一个含有数据成员title, link,及summary的对象Entry。

 - 辅助skip()方法是个递归方法。有关更多它的信息，请继续往下。

下面的代码展示了如何解析部分entry,title,link及summary标签。
```java
public static class Entry {
    public final String title;
    public final String link;
    public final String summary;
    private Entry(String title, String summary, String link) {
        this.title = title;
        this.summary = summary;
        this.link = link;
    }
}
  
// Parses the contents of an entry. If it encounters a title, summary, or link tag, hands them off
// to their respective "read" methods for processing. Otherwise, skips the tag.
private Entry readEntry(XmlPullParser parser) throws XmlPullParserException, IOException {
    parser.require(XmlPullParser.START_TAG, ns, "entry");
    String title = null;
    String summary = null;
    String link = null;
    while (parser.next() != XmlPullParser.END_TAG) {
        if (parser.getEventType() != XmlPullParser.START_TAG) {
            continue;
        }
        String name = parser.getName();
        if (name.equals("title")) {
            title = readTitle(parser);
        } else if (name.equals("summary")) {
            summary = readSummary(parser);
        } else if (name.equals("link")) {
            link = readLink(parser);
        } else {
            skip(parser);
        }
    }
    return new Entry(title, summary, link);
}
// Processes title tags in the feed.
private String readTitle(XmlPullParser parser) throws IOException, XmlPullParserException {
    parser.require(XmlPullParser.START_TAG, ns, "title");
    String title = readText(parser);
    parser.require(XmlPullParser.END_TAG, ns, "title");
    return title;
}
  
// Processes link tags in the feed.
private String readLink(XmlPullParser parser) throws IOException, XmlPullParserException {
    String link = "";
    parser.require(XmlPullParser.START_TAG, ns, "link");
    String tag = parser.getName();
    String relType = parser.getAttributeValue(null, "rel");  
    if (tag.equals("link")) {
        if (relType.equals("alternate")){
            link = parser.getAttributeValue(null, "href");
            parser.nextTag();
        } 
    }
    parser.require(XmlPullParser.END_TAG, ns, "link");
    return link;
}
// Processes summary tags in the feed.
private String readSummary(XmlPullParser parser) throws IOException, XmlPullParserException {
    parser.require(XmlPullParser.START_TAG, ns, "summary");
    String summary = readText(parser);
    parser.require(XmlPullParser.END_TAG, ns, "summary");
    return summary;
}
// For the tags title and summary, extracts their text values.
private String readText(XmlPullParser parser) throws IOException, XmlPullParserException {
    String result = "";
    if (parser.next() == XmlPullParser.TEXT) {
        result = parser.getText();
        parser.nextTag();
    }
    return result;
}
  ...
}
```

##跳过不关心的标签
解析XML数据的其中一个步骤就是需要忽略那些不需要关注的标签。下面是解析器的skip()方法。
```java
private void skip(XmlPullParser parser) throws XmlPullParserException, IOException {
    if (parser.getEventType() != XmlPullParser.START_TAG) {
        throw new IllegalStateException();
    }
    int depth = 1;
    while (depth != 0) {
        switch (parser.next()) {
        case XmlPullParser.END_TAG:
            depth--;
            break;
        case XmlPullParser.START_TAG:
            depth++;
            break;
        }
    }
 }
```
它的工作过程如下：

- 如果当前的项目不是START_TAG的话则抛出异常。
- 它会消费掉START_TAG标签，直到遇到与之相匹配的END_TAG时结束。
- 为了确保在正确的END_TAG时结束，这里使用了深度追踪的方式。

所以如果当前的标签含有内嵌标签，depth的值就不会是0，直到解析器消费完了改标签内的所有项目。试想一下解析器如何跳过&lt;author>标签，它含有两个内嵌标签&lt;name>和&lt;uri>:

- 第一次while循环，解析器所遇到的&lt;author>标签的下一个标签是&lt;name>的START_TAG项目，这时depth的值被自增到2.
- 第二次while循环，解析器所遇到的下一个标签是&lt;/name>的END_TAG项目。这时depth的值被自减到1.
- 第三次while循环，解析器所遇到的下一个标签是&lt;uri>的START_TAG项目。这时depth的值被自增到2.
- 第四次while循环，解析器所遇到的下一个标签是&lt;/uri>的END_TAG项目。这时depth的值被自减到1.
- 第四次while循环，解析器所遇到的下一个标签是&lt;/author>的END_TAG项目。这时depth的值被自减到0.这表示&lt;author>标签被成功跳过。

##消费XML数据
示例应用中将获取解析XML源的过程放在了[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)中。

loadPage()方法执行了以下过程：

- 以字符串的形式实例化了XML源的URL地址。
- 如果用户的设置以及网络状况良好，则调用new DownloadXmlTask().execute(url)。这实例化了一个新的 DownloadXmlTask 对象，并运行了其 execute() 方法，该方法会下载并解析源，最后将返回一个用于显示在UI界面上的字符串形式的结果。

```java
public class NetworkActivity extends Activity {
    public static final String WIFI = "Wi-Fi";
    public static final String ANY = "Any";
    private static final String URL = "http://stackoverflow.com/feeds/tag?tagnames=android&sort=newest";
   
    // Whether there is a Wi-Fi connection.
    private static boolean wifiConnected = false; 
    // Whether there is a mobile connection.
    private static boolean mobileConnected = false;
    // Whether the display should be refreshed.
    public static boolean refreshDisplay = true; 
    public static String sPref = null;
    ...
      
    // Uses AsyncTask to download the XML feed from stackoverflow.com.
    public void loadPage() {  
      
        if((sPref.equals(ANY)) && (wifiConnected || mobileConnected)) {
            new DownloadXmlTask().execute(URL);
        }
        else if ((sPref.equals(WIFI)) && (wifiConnected)) {
            new DownloadXmlTask().execute(URL);
        } else {
            // show error
        }  
    }
```

DownloadXmlTask实现了AsyncTask的下面一些方法：

- doInBackground() 执行了loadXmlFromNetwork()方法，它将源的URL地址作为参数传递给loadXmlFromNetwork()，当loadXmlFromNetwork()处理完成之后，将处理后的字符串返回。
- onPostExecute() 获得刚刚返回的字符串将其显示在UI界面上。

```java
// Implementation of AsyncTask used to download XML feed from stackoverflow.com.
private class DownloadXmlTask extends AsyncTask<String, Void, String> {
    @Override
    protected String doInBackground(String... urls) {
        try {
            return loadXmlFromNetwork(urls[0]);
        } catch (IOException e) {
            return getResources().getString(R.string.connection_error);
        } catch (XmlPullParserException e) {
            return getResources().getString(R.string.xml_error);
        }
    }
    @Override
    protected void onPostExecute(String result) {  
        setContentView(R.layout.main);
        // Displays the HTML string in the UI via a WebView
        WebView myWebView = (WebView) findViewById(R.id.webview);
        myWebView.loadData(result, "text/html", null);
    }
}
```

下面是loadXmlFromNetwork()方法的实现，它执行了以下过程：

1.实例化了StackOverflowXmlParser。它还创建了一个List的变量及title, url, summary变量，这些变量用于持有从XML中读取的值。
2.调用downloadUrl()，它用于获取源，并返回一个InputStream对象。
3.使用StackOverflowXmlParser 来解析刚刚的InputStream对象。 StackOverflowXmlParser将提取出的数据放入到List中。
4.处理List，将其中的数据与HTML标签整合。
5.最后返回一个HTML形式的字符串，字符串用于显示在主界面上。
```java
// Uploads XML from stackoverflow.com, parses it, and combines it with
// HTML markup. Returns HTML string.
private String loadXmlFromNetwork(String urlString) throws XmlPullParserException, IOException {
    InputStream stream = null;
    // Instantiate the parser
    StackOverflowXmlParser stackOverflowXmlParser = new StackOverflowXmlParser();
    List<Entry> entries = null;
    String title = null;
    String url = null;
    String summary = null;
    Calendar rightNow = Calendar.getInstance(); 
    DateFormat formatter = new SimpleDateFormat("MMM dd h:mmaa");
        
    // Checks whether the user set the preference to include summary text
    SharedPreferences sharedPrefs = PreferenceManager.getDefaultSharedPreferences(this);
    boolean pref = sharedPrefs.getBoolean("summaryPref", false);
        
    StringBuilder htmlString = new StringBuilder();
    htmlString.append("<h3>" + getResources().getString(R.string.page_title) + "</h3>");
    htmlString.append("<em>" + getResources().getString(R.string.updated) + " " + 
            formatter.format(rightNow.getTime()) + "</em>");
        
    try {
        stream = downloadUrl(urlString);        
        entries = stackOverflowXmlParser.parse(stream);
    // Makes sure that the InputStream is closed after the app is
    // finished using it.
    } finally {
        if (stream != null) {
            stream.close();
        } 
     }
    
    // StackOverflowXmlParser returns a List (called "entries") of Entry objects.
    // Each Entry object represents a single post in the XML feed.
    // This section processes the entries list to combine each entry with HTML markup.
    // Each entry is displayed in the UI as a link that optionally includes
    // a text summary.
    for (Entry entry : entries) {       
        htmlString.append("<p><a href='");
        htmlString.append(entry.link);
        htmlString.append("'>" + entry.title + "</a></p>");
        // If the user set the preference to include summary text,
        // adds it to the display.
        if (pref) {
            htmlString.append(entry.summary);
        }
    }
    return htmlString.toString();
}
// Given a string representation of a URL, sets up a connection and gets
// an input stream.
private InputStream downloadUrl(String urlString) throws IOException {
    URL url = new URL(urlString);
    HttpURLConnection conn = (HttpURLConnection) url.openConnection();
    conn.setReadTimeout(10000 /* milliseconds */);
    conn.setConnectTimeout(15000 /* milliseconds */);
    conn.setRequestMethod("GET");
    conn.setDoInput(true);
    // Starts the query
    conn.connect();
    return conn.getInputStream();
}
```
