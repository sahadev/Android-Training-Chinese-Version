原文地址：[http://android.xsoftlab.net/training/articles/security-tips.html](http://android.xsoftlab.net/training/articles/security-tips.html)

Android系统内置的安全策略可以有效的降低应用程序的安全问题。所以默认创建的应用程序已经包含了一定程度的安全保护措施。

Android所包含的安全策略有：

- 应用程序沙箱，它可以使APP的数据与执行代码与其它APP隔离。
- 应用程序框架对于常见的防护措施的强大实现，比如密码、权限以及安全的IPC机制。
- 一些安全技术的应用，比如ASLR, NX, ProPolice, safe_iop, OpenBSD dlmalloc, OpenBSD calloc, 以及Linux mmap_min_addr，可以降低常见的内存管理错误的风险。
- 加密文件系统可以保证设备在丢失后其中的数据不被盗取。
- 用户权限授予可以限制访问系统特性以及用户数据。
- Android的定义权限可以控制程序的数据只在程序的使用范围之内。

不过，对本文中所提到的Android安全策略最好还是熟悉一下。在日常的编码中使用以下策略可以降低无意中造成的安全漏洞。

##数据存储
应用程序常见的安全问题就是它们的数据是否可被其它应用程序访问到。Android有3种基本的数据存储方式。

###内部存储
默认情况下，由APP自己创建的文件只能由APP本身访问。这种防护措施由Android框架实现，也足以应对大多数应用的安全情况。

通常应当避免为IPC文件使用[MODE_WORLD_WRITEABLE](http://android.xsoftlab.net/reference/android/content/Context.html#MODE_WORLD_WRITEABLE)模式或[MODE_WORLD_READABLE](http://android.xsoftlab.net/reference/android/content/Context.html#MODE_WORLD_READABLE)模式，因为这些模式没有对数据的访问提供限制能力，也没有在数据格式上做任何控制。如果需要在进程间共享数据，你应当考虑使用ContentProvider。ContentProvider对其它APP提供了数据读写权限，也可以按照情况动态的授予权限。

如果需要对一些较为敏感的数据提供额外的保护，你可以选择使用密钥对本地文件进行加密的办法做到。比如，可以将密钥放入[KeyStore](http://android.xsoftlab.net/reference/java/security/KeyStore.html)，并以密码将其保护起来。不过这种方案也不是万能的：某些破解手段可以监视用户所输入的密码，不过它可以对丢失的设备进行保护。
###外部存储
在外部存储器上创建的文件可被全局读写。因为外部存储器可被用户移除，也可以被任何的应用程序修改，所以不应当在外部存储器上存储敏感的用户数据。

正因为可能存在不可信资源，所以从外部存储器中处理数据时应当执行输入验证。我们强烈的建议不要在外部存储器上存储用于动态加载的可执行文件或者类文件。如果APP需要从外部存储器上接收可执行文件，那么这些文件应当是被签过名的，在加载之前也应当对这些签名进行验证。
###内容提供者
ContentProvider提供了一种结构化的存储机制，这种机制可以对所有的应用程序形成一种约束。如果不打算使其它的应用程序访问你的ContentProvider，那么只需再程序的清单文件中标记对应的ContentProvider的属性android:exported=false即可。否则，设置android:exported=true便意味着其它应用程序可以访问其中的数据。

在创建ContentProvider时便默认允许其它应用程序可以访问其中的数据，你可以对读或者写进行单一权限限制，也可以同时管理。

如果使用ContentProvider只是为了在自身的APP之间共享数据，更加合理的方式是将android:protectionLevel属性值设置为"signature"。Signature权限不需要用户确认，所以这样可以提供良好的用户体验，以及更多的控制力。

ContentProvider还可以通过android:grantUriPermissions属性提供更细粒度的访问能力。访问者应当在Intent中使用FLAG_GRANT_READ_URI_PERMISSION标志或FLAG_GRANT_WRITE_URI_PERMISSION标志进行访问。这些权限的范围可被[<grant-uri-permission element>](http://android.xsoftlab.net/guide/topics/manifest/grant-uri-permission-element.html)做更进一步的限制。

当访问ContentProvider时，使用参数化方法比如query(), update(), 及delete()可以避免不可信来源的SQL注入。

不要对写入权限的安全拥有错误的认知。考虑一下，写入权限允许执行SQL语句，这可能会使一些数据被确认。比如，一名攻击者可能会在通话记录中查找一个指定的电话号码是否存在，如果这条数据存在，那么攻击者只需进行修改便可得知结果。如果ContentProvider的数据结构可被猜测出，那么写入权限就相当于也同时提供了读取权限。

##使用权限
因为Android每个应用都处于沙箱之内，所以应用必须显式的共享资源与数据。这需要应用程序单独声明自有权限。

###请求权限
我们推荐应用程序所需的权限越好越好。这样可以降低权限滥用的风险，也更易让用户接受，也可以减少黑客的攻击入口。通常情况下，如果某个权限不是必要的，那就不要去请求它。

如果应用程序可以做到不需要任何权限，那么这是最完美的。比如，如果需要通过访问设备信息的方式来创建唯一标识符的话，我们更推荐[GUID](http://android.xsoftlab.net/reference/java/util/UUID.html)。又比如，相比于将数据存储于外部存储器，我们更推荐内部存储器。

另外在请求权限时，可以使用\<permissions>来保护IPC。IPC对于安全特别敏感、薄弱，并且它会被暴露给其它应用程序，比如ContentProvider。 除了可能需要用户确认的权限之外，我们更推荐使用访问控制，因为这些权限可能会使用户感到困惑、不解。比如，可以考虑对个人开发者开发的应用程序为IPC通讯使用"[signature](http://android.xsoftlab.net/guide/topics/manifest/permission-element.html#plevel)"的权限保护等级。

不要泄露受保护的权限数据。这种情况仅会发生在通过IPC暴露数据时。因为它拥有特殊的权限，并且对于任何的IPC接口的客户端也没有要求提供该权限。

> More details on the potential impacts, and frequency of this type of problem is provided in this research paper published at USENIX:[http://www.cs.berkeley.edu/~afelt/felt_usenixsec2011.pdf](http://www.cs.berkeley.edu/~afelt/felt_usenixsec2011.pdf)

###创建权限
通常情况下应当尽量少的使用权限，少用权限就意味着更安全。创建权限这种事情对于大多数应用来说是用不到的，因为系统定义的权限足以涵盖所有情况，这些权限可以正确的做出访问检查。

如果必须创建权限，考虑是否可以使用"signature"保护等级完成你的所需任务。"signature"会将自身完全暴露给用户，只允许具有相同签名的应用程序访问。

如果要创建"dangerous"保护等级的权限，那么有些东西需要考虑在内：

- 权限必须提供一段简短的描述该权限安全的字符串。
- 描述权限的字符串必须提供不同地区的语言。
- 如果权限的描述含糊不清或者用户认为这会为其带来风险，那么用户可能会选择不安装应用。
- 如果权限的生成器没有安装的话，应用程序可能会请求权限。

上面的每一条对于作为程序员的你都是一项重要的非技术性挑战，这样做可能会使用户感到困惑，这就是为什么我们不鼓励使用"dangerous"权限等级的原因。

##网络安全
网络传输本身就存在安全风险，因为它会包含用户的隐私数据。人们越来越关心移动设备的隐私问题，尤其是执行网络传输时，所以APP需要至始至终以最佳的安全方案保护用户的数据安全。

###使用IP网络
Android所处的网络环境与其它的Linux系统有着很大的不同。主要考虑的就是选用适用于敏感数据的协议。比如[HttpsURLConnection](http://android.xsoftlab.net/reference/javax/net/ssl/HttpsURLConnection.html)用于安全的WEB通信。我们推荐在支持HTTPS协议的服务器上使用HTTP，因为移动设备会频繁的连接到不可信网络，比如公共的Wi-Fi热点。

经认证，Socket等级的加密通讯可以使用[SSLSocket](http://android.xsoftlab.net/reference/javax/net/ssl/SSLSocket.html)类轻易实现。鉴于Android设备会频繁的连接到不安全的无线网络，所以我们强力的建议对所有的应用程序都使用安全的网络实现。

我们也见过一些应用在处理敏感的IPC上使用了localhost网络端口。我们不鼓励使用这种方式，因为这些接口能被其它应用程序访问的到。你应当使用Android的IPC机制。

还有一个常见的问题就是，根证书重复用于验证从HTTP或者其它网络上下载的不可信数据。这也包括WebView以及HTTP请求响应的输入验证。

###使用电话网络
SMS(短消息服务)协议主要用于个人对个人之间的通讯，它并不适用于APP的数据传输。由于SMS的限制，我们强烈的推荐使用Google Cloud Messaging(GCM)及IP网络来传输服务器与设备之间的数据。

要注意，SMS既没有对网络和设备进行加密也没有对其进行相关验证。尤其是，任何的SMS接收器应当考虑到会有一位恶意的用户会发送SMS给你的应用，不要相信没有验证过的SMS数据，并用它们来执行一些敏感操作。还有，你应当意识到SMS可能会在网络上被拦击并被篡改。在Android设备内，SMS消息是由广播意图传送的，所以这些数据可以被拥有[READ_SMS](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_SMS)权限的应用程序读取到。

##输入验证
无论程序运行在哪个平台上，没有执行充分的输入验证都是影响程序安全的主要原因之一。Android提供了平台等级的安全策略，这可以降低应用程序输入验证问题的暴露率，应用程序应当尽可能的使用这些平台特性。还应该注意：安全性语言的选择倾向于减少输入验证问题的可能性。

如果你在使用本地代码，那么从文件中读取的数据、从网络上接收的数据或从IPC接收的数据都可能会引起安全问题。最常见的问题就是缓冲区溢出，使用释放后的对象等等。Android提供了大量的相关技术手段比如ASLR(Address Space Layout Randomization)、DEP(Data Execution Prevention)，这些技术手段可以降低这些错误的出现频率，但是它们不会解决根本的内在问题。你可以谨慎的处理指针或管理缓冲区来防止这些问题的出现。

如果使用SQL数据库或者ContentProvider进行数据查询，那么SQL注入可能是个问题。避免这些问题最好使用参数化的查询方法。将权限降为只读或只写可以降低SQL注入的相关风险。

如果不能够使用以上提及的安全建议，那么我们强烈推荐使用结构良好的数据格式，并在使用时对这些数据格式进行验证。

##处理用户数据
通常来说，对于用户的数据安全最好的方法就是尽量少使用可以访问到敏感数据或者用户数据的API。如果可以避免存储或者传输这些信息，那么就不要传输这些数据。可考虑应用程序是否能够实现这么一种逻辑：使用数据的哈希码或者一种不可逆的数据形态。比如，程序可能会实现这么一种逻辑：将电子邮件地址的哈希码作为一个关键的值，这样可以避免使用本身的电子邮件地址。这可以降低暴露数据的机会，这也可以降低攻击者入侵应用程序的机会。

如果程序需要访问用户的个人信息，比如密码或者用户名等等，要记住一些组件可能会要求你提供相关的隐私政策，来解释这些数据的使用与存储。所以接下来数据访问实践可以简化遵循规则。

你还应当考虑应用程序是否有无意中将用户的个人数据暴露给了第三方。如果你不清楚这些第三方组件或服务为什么要使用用户的信息，那么就不要提供给它们。通常降低个人信息的访问可以降低这块的安全风险。

如果必须要访问敏感数据，那么评估一下这些信息是否有必要必须传输到服务器，或者是否这些操作只在客户端执行。考虑凡是涉及到用户敏感数据的代码都在客户端执行，这样可以避免不必要的网络传输和安全泄漏问题。

还有，要确保没有将用户数据通过IPC暴露给其它应用程序、全局可读写文件或者网络Socket。

如果需要GUID，可以创建一个大的唯一的数据将其保存下来。不要使用与电话有关的数字，比如电话号码、IMEI，这些信息可能会与用户信息有关。

写入设备日志时要当心。在Android中，日志是共享资源，可被具有[READ_LOGS](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_LOGS)权限的应用读取到。尽管日志数据是临时的，但是不恰当的用户信息日志可能会无意中将信息泄露给其它应用程序。

##WebView的使用
因为WebView会解析像HTML和JavaScript这样的web内容，所以不正确的使用会带来常见的安全问题。Android提供了大量机制来收缩这些潜在问题的范围，比如通过限制WebView的能力来达到最低的运行需求。

如果程序没有直接使用到WebView中的JavaScript，那么不要调用[setJavaScriptEnabled()](http://android.xsoftlab.net/reference/android/webkit/WebSettings.html#setJavaScriptEnabled(boolean))。一些示例代码可能会使用该方法，所以如果不需要的话，在你的程序中关闭它。默认情况下，WebView不会执行JavaScript，所以不会发生跨站脚本攻击。

使用[addJavaScriptInterface()](http://android.xsoftlab.net/reference/android/webkit/WebView.html#addJavascriptInterface(java.lang.Object, java.lang.String))需要特别当心，因为它会允许JavaScript调用预留给Android原生应用的方法。如果你使用它，要确认所有的Web的相关内容都是可信的。如果不可信的内容允许进入，那么不可信的 JavaScript 可能会调用App内的相关方法。一般我们推荐在APK内包含JavaScript的时候使用addJavaScriptInterface()。

如果程序通过WebView访问敏感数据，你可能需要使用[clearCache()](http://android.xsoftlab.net/reference/android/webkit/WebView.html#clearCache(boolean))方法来删除存储在本地的所有文件。我们可以使用HTTP头部的某些属性比如no-cache来表示应用程序不应当缓存特别的内容。

Android 4.4之前版本的webkit含有大量的安全问题。如果App运行在这些版本上，应当确认WebView所渲染的内容都是可信的。如果应用必须渲染开放的Web内容，考虑实现一个独有的渲染器，这样可以及时更新相关的安全补丁。

##管理证书
通常来说，我们不推荐频繁的要求用户凭证，推荐使用授权令牌。

用户名及密码通常不应该存在本地。应当使用用户输入的用户名及密码进行初始化验证，然后使用一个权限Token进行通信。

如果一个账户需要被多个程序访问，那么应当使用[AccountManager](http://android.xsoftlab.net/reference/android/accounts/AccountManager.html)。如果可能的话，使用[AccountManager](http://android.xsoftlab.net/reference/android/accounts/AccountManager.html)与服务进行交互，绝不要将密码存在设备上。

如果证书只是用作于你创建的应用程序，那么可以使用[checkSignature()](http://android.xsoftlab.net/reference/android/content/pm/PackageManager.html#checkSignatures(int,%20int))方法进行程序访问验证。如果只有一个程序使用了证书，那么[KeyStore](http://android.xsoftlab.net/reference/java/security/KeyStore.html)可能更适合你。

##使用密码
