原文地址：[http://android.xsoftlab.net/training/articles/security-tips.html](http://android.xsoftlab.net/training/articles/security-tips.html)

Android系统内置的安全策略可以有效的降低应用程序的安全问题。所以默认创建的应用程序已经包含了一定程度的安全保护措施。

Android所包含的安全策略有：

- 应用程序沙箱，它可以使APP的数据、代码与其它APP相互隔离。
- 应用程序框架对于常见防护措施的强大实现，比如密码、权限以及安全的IPC机制。
- 一些安全技术的应用，比如ASLR, NX, ProPolice, safe_iop, OpenBSD dlmalloc, OpenBSD calloc, 以及Linux mmap_min_addr，可以降低常见的内存管理错误风险。
- 文件加密系统可以保证设备在丢失后其中的数据不被盗取。
- 用户权限授予可以限制访问系统特性以及用户数据。
- Android定义的权限可以控制程序的数据只能在程序的控制范围之内。

本文中所提及的Android安全策略对于日常开发非常重要，应养成良好的编码习惯。在日常的编码行为中使用以下策略可以有效降低无意中造成的安全风险。

##数据存储
应用程序常见的安全问题就是它们的数据是否可被其它应用程序访问到。Android有3种基本的数据存储方式。

###内部存储
默认情况下，由APP自己创建的文件只能由APP本身访问。这种防护措施由Android框架实现，也足以应对大多数应用的安全情况。

通常应当避免为IPC文件使用[MODE_WORLD_WRITEABLE](http://android.xsoftlab.net/reference/android/content/Context.html#MODE_WORLD_WRITEABLE)模式或[MODE_WORLD_READABLE](http://android.xsoftlab.net/reference/android/content/Context.html#MODE_WORLD_READABLE)模式，因为这些模式没有对数据的访问提供限制能力，也没有在数据格式上做任何控制。如果需要在进程间共享数据，你应当考虑使用[ContentProvider](http://android.xsoftlab.net/guide/topics/providers/content-providers.html)。[ContentProvider](http://android.xsoftlab.net/guide/topics/providers/content-providers.html)对APP提供了数据读写权限，也可以按照实际情况动态的授予权限。

如果需要对一些较为敏感的数据提供额外的保护，你可以使用密钥对本地文件进行加密。比如，可以将密钥放入[KeyStore](http://android.xsoftlab.net/reference/java/security/KeyStore.html)，并以用户密码将其保护起来。不过这种方案也不是万能的：某些破解手段可以监视用户所输入的密码，不过这种方式可以对丢失的设备进行保护。
###外部存储
在外部存储器上创建的文件可被全局读写。因为外部存储器可被用户移除，也可以被任何的应用程序修改，所以不应当在外部存储器上存储敏感的用户数据。

正因为可能存在不可信资源，所以从外部存储器中读取数据时应当执行输入验证。我们强烈的建议不要在外部存储器上存储用于动态加载的可执行文件或者类文件。如果APP需要从外部存储器上接收可执行文件，那么这些文件应当是被签过名的，在加载之前也应当对这些签名进行校验。
###内容提供者
[ContentProvider](http://android.xsoftlab.net/guide/topics/providers/content-providers.html)提供了一种结构化的存储机制，这种机制可以对所有的应用程序形成一种约束。如果不打算使其它的应用程序访问你的[ContentProvider](http://android.xsoftlab.net/guide/topics/providers/content-providers.html)，那么只需在程序的清单文件中标记[ContentProvider](http://android.xsoftlab.net/guide/topics/providers/content-providers.html)的属性[android:exported=false](http://android.xsoftlab.net/guide/topics/manifest/provider-element.html#exported)即可。否则，设置[android:exported=true](http://android.xsoftlab.net/guide/topics/manifest/provider-element.html#exported)便意味着其它应用程序可以访问其中的数据。

在创建ContentProvider时默认允许其它应用程序可以访问其中的数据，你可以对读或者写进行单一权限限制，也可以同时管理。

如果使用ContentProvider只是为了在自身的APP之间共享数据，更加合理的方式是将[android:protectionLevel](http://android.xsoftlab.net/guide/topics/manifest/permission-element.html#plevel)属性值设置为"signature"。Signature权限不需要用户确认，所以这样可以提供良好的用户体验，以及更多的控制力。

ContentProvider还可以通过[android:grantUriPermissions](http://android.xsoftlab.net/guide/topics/manifest/provider-element.html#gprmsn)属性提供更细粒度的访问能力。访问者应当在Intent中使用[FLAG_GRANT_READ_URI_PERMISSION](http://android.xsoftlab.net/reference/android/content/Intent.html#FLAG_GRANT_READ_URI_PERMISSION)标志或[FLAG_GRANT_WRITE_URI_PERMISSION](http://android.xsoftlab.net/reference/android/content/Intent.html#FLAG_GRANT_READ_URI_PERMISSION)标志进行访问。这些权限的范围可被[<grant-uri-permission element>](http://android.xsoftlab.net/guide/topics/manifest/grant-uri-permission-element.html)做更进一步的限制。

当访问ContentProvider时，使用参数化方法比如[query()](http://android.xsoftlab.net/reference/android/content/ContentProvider.html#query(android.net.Uri,%20java.lang.String[],%20java.lang.String,%20java.lang.String[],%20java.lang.String)), [update()](http://android.xsoftlab.net/reference/android/content/ContentProvider.html#update(android.net.Uri,%20android.content.ContentValues,%20java.lang.String,%20java.lang.String[])), 及[delete()](http://android.xsoftlab.net/reference/android/content/ContentProvider.html#delete(android.net.Uri,%20java.lang.String,%20java.lang.String[]))可以避免不可信来源的SQL注入。

不要对写入权限的安全拥有错误的认知。考虑一下，写入权限允许执行SQL语句，这可能会使一些数据被确认。比如，一名攻击者可能会在通话记录中查找一个指定的电话号码是否存在，如果这条数据存在，那么攻击者只需进行修改便可得知结果。如果ContentProvider的数据结构可被猜测出，那么写入权限就相当于也同时提供了读取权限。

##使用权限
因为Android每个应用都处于沙箱之内，所以如果需要共享资源与数据的话，应用必须显式的声明自有权限。

###请求权限
我们推荐应用程序所需的权限越少越好。这样可以降低权限滥用的风险，也更易让用户接受，也可以减少黑客的攻击入口。通常情况下，如果某个权限不是必要的，那就不要去请求它。

如果应用程序可以做到不需要任何权限，那么这是最完美的。比如，如果需要通过访问设备信息的方式来创建唯一标识符的话，我们更推荐[GUID](http://android.xsoftlab.net/reference/java/util/UUID.html)。又比如，相比于将数据存储于外部存储器，我们更推荐内部存储器。

另外在请求权限时，可以使用< permissions>来保护IPC。IPC对于安全特别敏感、薄弱，并且它会被暴露给其它应用程序，比如ContentProvider。 除了可能需要用户确认的权限之外，我们更推荐使用访问控制，因为这些权限可能会使用户感到困惑、不解。比如，可以考虑对个人开发者的应用程序使用"[signature](http://android.xsoftlab.net/guide/topics/manifest/permission-element.html#plevel)"的IPC权限保护等级。

不要泄露受保护的权限数据。这种情况仅会发生在通过IPC共享数据时：因为它拥有特殊的权限，并且对于任何的IPC接口的客户端也没有要求出示该权限。

> More details on the potential impacts, and frequency of this type of problem is provided in this research paper published at USENIX:[http://www.cs.berkeley.edu/~afelt/felt_usenixsec2011.pdf](http://www.cs.berkeley.edu/~afelt/felt_usenixsec2011.pdf)

###创建权限
通常情况下应当尽量少的使用权限，少用权限就意味着更安全。创建权限这种事情对于大多数应用来说是用不到的，因为系统定义的权限足以涵盖所有情况，这些权限会提供访问检查。

如果必须创建权限，考虑是否可以使用"signature"保护等级完成你的所需任务。"signature"会将自身完全暴露给用户，只允许具有相同签名的应用程序访问。

如果要创建"dangerous"保护等级的权限，那么有些东西需要考虑在内：

- 权限必须提供一段简短的描述该权限安全的字符串。
- 描述权限的字符串必须提供不同地区的语言。
- 如果权限的描述含糊不清或者用户认为这会为其带来风险，那么用户可能会选择不安装应用。
- 如果权限的生成器没有安装的话，应用程序可能会请求权限。

上面的每一条对于作为程序员的你都是一项重要的非技术性挑战，这样做可能会使用户感到困惑，这就是为什么我们不鼓励使用"dangerous"权限等级的原因。

##网络安全
网络传输本身就存在安全风险，因为它会包括用户的隐私数据。人们越来越关心移动设备的隐私问题，尤其是执行网络传输时，所以APP需要至始至终以最佳的安全方案保护用户的数据安全。

###使用IP网络
Android所处的网络环境与其它的Linux系统有着很大的不同。主要考虑的就是选用适用于敏感数据传输的协议。比如[HttpsURLConnection](http://android.xsoftlab.net/reference/javax/net/ssl/HttpsURLConnection.html)用于安全的WEB通信。我们推荐在支持HTTPS，因为移动设备会频繁的连接到不可信网络，比如公共的Wi-Fi热点。

经认证，Socket等级的加密通讯可以使用[SSLSocket](http://android.xsoftlab.net/reference/javax/net/ssl/SSLSocket.html)类简单实现。鉴于Android设备会频繁的连接到不安全的无线网络，所以我们强力的建议对所有的应用程序都使用安全的网络实现。

我们也见过一些应用在处理敏感的IPC上使用了localhost网络端口。我们不鼓励使用这种方式，因为这些接口能被其它应用程序访问到，应当使用Android的IPC机制。

还有一个常见的问题就是，根证书重复用于验证从HTTP或者其它网络上下载的不可信数据。这也包括WebView以及HTTP请求响应的输入验证。

###使用电话网络
SMS(短消息服务)协议主要用于个人对个人之间的通讯，它并不适用于APP的数据传输。由于SMS的限制，我们强烈的推荐使用Google Cloud Messaging(GCM)及IP网络来传输服务器与设备之间的数据。

要注意，SMS既没有对网络和设备进行加密也没有对其进行相关验证。尤其是，任何的SMS接收器应当考虑到会有一位恶意的用户会发送SMS给你的应用，不要相信没有验证过的SMS数据，并用它们来执行一些敏感操作。还有，你应当意识到SMS可能会在网络上被拦截并被篡改。在Android设备内，SMS消息是由广播意图传送的，所以这些数据可以被拥有[READ_SMS](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_SMS)权限的应用程序读取到。

##输入验证
无论程序运行在哪个平台上，没有执行充分的输入验证都是影响程序安全的主要原因之一。Android提供了平台等级的安全策略，这可以降低应用程序输入验证问题的暴露率，应用程序应当尽可能的使用这些平台特性。还应该注意：安全性语言的选择倾向于减少输入验证问题的可能性。

如果你在使用本地代码，那么从文件中读取的数据、从网络上接收的数据或从IPC接收的数据都可能会引起安全问题。最常见的问题就是缓冲区溢出，使用释放后的对象等等。Android提供了大量的相关技术手段比如ASLR(Address Space Layout Randomization)、DEP(Data Execution Prevention)，这些技术手段可以降低这些错误的出现频率，但是它们不会解决根本的内在问题。你可以谨慎的处理指针或管理缓冲区来防止这些问题的出现。

如果使用SQL数据库或者ContentProvider进行数据查询，那么SQL注入可能是个问题。避免这些问题最好使用参数化的查询方法。将权限降为只读或只写可以降低SQL注入带来的相关风险。

如果不能够使用以上提及的安全建议，那么我们强烈推荐使用结构良好的数据格式，并在使用时对这些数据格式进行验证。

##处理用户数据
通常来说，对于用户的数据安全最好的方法就是尽量少使用可以访问到敏感数据或者用户数据的API。如果可以避免存储或者传输这些信息，那么就不要传输这些数据。可考虑应用程序是否能够实现这么一种逻辑：使用数据的哈希码或者一种不可逆的数据形态。比如，程序可能会实现这么一种逻辑：将电子邮件地址的哈希码作为一个关键的值，这样可以避免使用原有的电子邮件地址，这可以降低暴露数据的机会，也可以降低攻击者入侵应用程序的机会。

如果程序需要访问用户的个人信息，比如密码或者用户名等等，要记住一些组件可能会要求你提供相关的隐私政策，来解释这些数据的使用与存储。所以接下来数据访问实践可以简化遵循规则。

你还应当考虑应用程序是否有无意中将用户的个人数据暴露给了第三方。如果你不清楚这些第三方组件或服务为什么要使用用户的信息，那么就不要提供给它们。通常降低个人信息的访问可以降低这块的安全风险。

如果必须要访问敏感数据，那么评估一下这些信息是否有必要传输到服务器，或者是否这些操作只需在客户端执行就可以。考虑凡是涉及到用户敏感数据的代码都在客户端执行，这样可以避免不必要的网络传输和安全泄漏问题。

还有，要确保没有将用户数据通过IPC、全局可读写文件或者网络Socket暴露给其它应用程序。

如果需要GUID，可以创建一个大的唯一的数据将其保存下来。不要使用与电话有关的数字，比如电话号码、IMEI，这些信息可能会与用户信息有关。

写入设备日志时要当心。在Android中，日志是共享资源，可被具有[READ_LOGS](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_LOGS)权限的应用读取到。尽管日志数据是临时的，但是不恰当的用户信息日志可能会无意中将信息泄露给其它应用程序。

##WebView的使用
因为WebView会解析像HTML和JavaScript这样的web内容，所以不正确的使用会带来常见的安全隐患。Android提供了大量机制来收缩这些潜在问题的范围，比如通过限制WebView的能力来达到最低的运行需求。

如果程序没有直接使用到WebView中的JavaScript，那么不要调用[setJavaScriptEnabled()](http://android.xsoftlab.net/reference/android/webkit/WebSettings.html#setJavaScriptEnabled(boolean))。一些示例代码可能会使用该方法，所以如果不需要的话，在你的程序中关闭它。默认情况下，WebView不会执行JavaScript，所以不会发生跨站脚本攻击。

使用[addJavaScriptInterface()](http://android.xsoftlab.net/reference/android/webkit/WebView.html#addJavascriptInterface(java.lang.Object, java.lang.String))需要特别当心，因为它会允许JavaScript调用预留给Android原生应用的方法。如果你使用它，要确认所有的Web的相关内容都是可信的。如果不可信的内容允许进入，那么不可信的 JavaScript 可能会调用App内的相关方法。一般我们推荐在APK内包含JavaScript代码的时候使用addJavaScriptInterface()。

如果程序通过WebView访问敏感数据，你可能需要使用[clearCache()](http://android.xsoftlab.net/reference/android/webkit/WebView.html#clearCache(boolean))方法来删除存储在本地的所有文件。我们可以使用HTTP头部的某些属性比如no-cache来表示应用程序不应当缓存某些特殊的内容。

Android 4.4之前版本的webkit含有大量的安全问题。如果App运行在这些版本上，应当确认WebView所渲染的内容都是可信的。如果应用必须渲染开放的Web内容，考虑实现一个独有的渲染器，这样可以及时更新相关的安全补丁。

##管理证书
通常来说，我们不推荐频繁的要求用户凭证，推荐使用授权令牌。

用户名及密码通常不应该存在本地。应当使用用户输入的用户名及密码进行初始化验证，然后使用一个权限Token进行通信。

如果一个账户需要被多个程序访问，那么应当使用[AccountManager](http://android.xsoftlab.net/reference/android/accounts/AccountManager.html)。如果可能的话，使用[AccountManager](http://android.xsoftlab.net/reference/android/accounts/AccountManager.html)与服务进行交互，绝不要将密码存在设备上。

如果证书只是用作于你创建的应用程序，那么可以使用[checkSignature()](http://android.xsoftlab.net/reference/android/content/pm/PackageManager.html#checkSignatures(int,%20int))方法进行程序访问验证。如果只有一个程序使用了证书，那么[KeyStore](http://android.xsoftlab.net/reference/java/security/KeyStore.html)可能更适合你。

##使用密码
除了数据隔离、文件系统加密、安全通信通道等保护手段之外，Android还提供了一系列通过算法加密的安全保护措施。

通常情况下，Android内置的最高等级的安全实现已经可以支持各种安全情况。如果需要从一个已知的位置接受一个文件，那么HTTPS URI已经足够。如果需要一条安全通道，考虑使用[HttpsURLConnection](http://android.xsoftlab.net/reference/javax/net/ssl/HttpsURLConnection.html)或[SSLSocket](http://android.xsoftlab.net/reference/javax/net/ssl/SSLSocket.html)。

如果发现确实需要实现自己的安全协议，不建议自己实现加密算法。而应当使用已有的加密算法比如在[Cipher](http://android.xsoftlab.net/reference/javax/crypto/Cipher.html)中提供的AES加密算法或RSA加密算法。

使用安全随机数生成器[SecureRandom](http://android.xsoftlab.net/reference/java/security/SecureRandom.html)来初始化密钥[KeyGenerator](http://android.xsoftlab.net/reference/javax/crypto/KeyGenerator.html)。如果不使用由[SecureRandom](http://android.xsoftlab.net/reference/java/security/SecureRandom.html)生成的密钥的话，则会大大减弱加密算法的健壮性，也更容易遭受线下攻击。

如果需要将密钥存在本地以便后续使用，那么可以使用[KeyStore](http://android.xsoftlab.net/reference/java/security/KeyStore.html)类似的加密机制。

##使用进程间通信
一些App尝试使用传统的Linux技术比如网络Socket和共享文件来实现IPC。我们对此推荐使用Android为IPC实现的系统功能，例如[Intent](http://android.xsoftlab.net/reference/android/content/Intent.html)，[Binder](http://android.xsoftlab.net/reference/android/os/Binder.html)，[Messenger](http://android.xsoftlab.net/reference/android/os/Messenger.html)和[BroadcastReceiver](http://android.xsoftlab.net/reference/android/content/BroadcastReceiver.html)。Android的IPC机制允许验证连接到你IPC的程序的身份，并且可以对每个IPC机制设置安全策略。

许多安全元素通过IPC机制共享。如果你的IPC机制的目的不是为了供其他应用程序使用，那么请将对应元素的android:exported的属性设置为false。这对于由相同UID的多进程组成的应用很有帮助，或者对后期再开启该功能也有一定的帮助。

如果IPC的目的是为了可被其他应用访问，你可以通过使用[< permission>](http://android.xsoftlab.net/guide/topics/manifest/permission-element.html)元素指定安全策略。如果IPC只是为了在持有相同key的两个自有应用中使用，那么android:protectionLevel="signature"则更为适合。

###使用Intent
Intent是异步IPC的首选机制。这取决于程序的需求，你可能会使用[sendBroadcast()](http://android.xsoftlab.net/reference/android/content/Context.html#sendBroadcast(android.content.Intent)), [sendOrderedBroadcast()](http://android.xsoftlab.net/reference/android/content/Context.html#sendOrderedBroadcast(android.content.Intent,%20java.lang.String))或者显式Intent来指定应用程序组件。

要注意，由于有序广播可以被接收方消耗掉，所以这些广播可能不会被分发给所有的应用程序。如果你发送了一个广播，而该广播必须被指定接收器接收的，那么必须使用显式Intent，并且该Intent还需指明广播接收器的名称。

发送Intent的一方可以验证接收方是否允许非空的权限方法调用。只有含有该权限的应用才会接收到该Inetnt。如果广播中的Intent的数据是敏感的，那么应当考虑这里所使用的权限不会被非法的应用程序拦截。在这种情况下，你应当考虑直接调用接收器，而不是使用广播。

> **Note:**Intent filters不应当被视作为一种安全特性：因为组件可能会由显式的Intent调起。你应当在收到Intent的地方执行输入验证来确认该Intent是否被格式化为适用于调用广播接收器，服务或者Activity的格式。

###使用服务
[Service](http://android.xsoftlab.net/reference/android/app/Service.html)经常被用作支持其他程序的功能。每个[Service](http://android.xsoftlab.net/reference/android/app/Service.html)必须在它的清单文件中有相应的声明。

默认情况下，Service不是开放的，也不能够被其它应用程序调起。然而，如果在[Service](http://android.xsoftlab.net/reference/android/app/Service.html)的声明中添加了IntentFilter，那么默认就会被暴露出去。最好的办法就是显式的声明[android:exported](http://android.xsoftlab.net/guide/topics/manifest/service-element.html#exported)属性为你需要的属性。[Service](http://android.xsoftlab.net/reference/android/app/Service.html)还可以由android:permission属性保护。如果这么做了，那么其它的程序则需要在它们的清单文件中声明对应的权限才可以启动、停止或者绑定该服务。

[Service](http://android.xsoftlab.net/reference/android/app/Service.html)还可以保护在其权限内的IPC调用，在执行这个调用的实现之前调用[checkCallingPermission()](http://android.xsoftlab.net/reference/android/content/Context.html#checkCallingPermission(java.lang.String))进行必要检查。我们通常推荐使用在清单文件中声明的权限，因为有很多漏洞会被忽略。

###使用Binder及Messenger接口
[Binder](http://android.xsoftlab.net/reference/android/os/Binder.html)、[Messenger](http://android.xsoftlab.net/reference/android/os/Messenger.html)是远程过程调用的首要IPC机制。它们提供了一种定义良好的接口：可以进行端对端相互认证。

我们强烈推荐以不需要特定的权限检查的方式设计接口。由于[Binder](http://android.xsoftlab.net/reference/android/os/Binder.html)、[Messenger](http://android.xsoftlab.net/reference/android/os/Messenger.html)并不是在应用的清单文件中声明过的，因此不能对其采用特定权限。它们的权限通常来自于对应的Service或Activity所声明的权限。如果你创建了一个用于请求验证或者访问控制器的接口，那么这些控制器必须显式的添加在[Binder](http://android.xsoftlab.net/reference/android/os/Binder.html)、[Messenger](http://android.xsoftlab.net/reference/android/os/Messenger.html)的接口中。

如果创建了一个需要访问控制器的接口，使用[checkCallingPermission()](http://android.xsoftlab.net/reference/android/content/Context.html#checkCallingPermission(java.lang.String))验证调用者是否含有所需的权限。这在访问服务的远端代理之前尤其重要，正如你的应用的身份需要传给其它接口一样。如果调用一个由Service提供的接口，那么如果你没有访问给定服务所需的权限，那么[bindService()](http://android.xsoftlab.net/reference/android/content/Context.html#bindService(android.content.Intent, android.content.ServiceConnection, int))的调用可能会失败。如果调用一个由自身APP提供的一个本地的接口，那么[clearCallingIdentity()](http://android.xsoftlab.net/reference/android/os/Binder.html#clearCallingIdentity())则可以满足内部安全检查的需求。

有关更多执行与服务有关的IPC的相关信息，请参见[Bound Services](http://android.xsoftlab.net/guide/components/bound-services.html)。

###使用广播接收器
[BroadcastReceiver](http://android.xsoftlab.net/reference/android/content/BroadcastReceiver.html)用于处理由Intent发起的异步请求。

默认情况下，接收器可以被任何应用调起。如果你的广播接收器的作用是给其它程序使用，那么可能需要对接收器采取一些安全措施：在清单文件中添加相应的安全权限。这可以防止没有正确权限的应用程序发送Intent给广播接收器。

##动态加载代码
Dalvik是Android的运行时虚拟机。虽然Dalvik专用于Android，但是其它虚拟机上的相关安全问题也同样适用于Android。一般不需要关心与虚拟机相关的安全问题，因为Android的应用程序运行在安全的沙箱环境中，所以系统上的其它进程访问不到程序的代码或者私有数据。

如果你对更深的虚拟机安全课题感兴趣，那么推荐熟悉一些现有的有关这一课题的相关文献。其中最受欢迎的两个资源如下：
- [http://www.securingjava.com/toc.html](http://www.securingjava.com/toc.html)
- [https://www.owasp.org/index.php/Java_Security_Resources](https://www.owasp.org/index.php/Java_Security_Resources)

这篇文档主要关注于Android的特殊之处和与其它虚拟机环境有什么不同。对于对其它虚拟机很有经验的开发者来说，这里有两个很主要的Android的不同之处：

- 一些虚拟机，比如JVM或.net运行时，扮演了一个安全边界的角色，从底层操作系统将代码、功能隔离。然而在Android中Dalvik虚拟机并没有这样的功能——应用程序沙箱实现与操作系统层面，所以Dalvik可以与应用的本地代码进行交互，而没有任何的安全限制。
- 由于移动设备有限的存储空间，一些开发者可能需要通过模块化构建应用程序，并使用动态加载技术。如果这么做，那么则需要考虑在哪接收应用的逻辑代码？又应当将这些代码存在哪？不要使用没有经过验证的代码，比如从不安全的网络资源上或者是外部存储器中加载的代码，因为这些代码很有可能会被其它程序篡改。

##本地代码的安全
一般我们推荐使用Android SDK来进行应用程序开发，而不是使用本地代码开发。由本地代码构建的程序会更加复杂，也缺少了灵活性，也更容易产生像缓冲区溢出等常见的内存泄露错误。

因为Android建立于Linux kernel基础之上，所以如果使用本地代码的话，那么Linux开发中所遇到的安全问题也同样适用于此。由于Linux安全相关超出了本文的范围，所以这里提供了很受欢迎的“Linux和Unix如何安全编程”的相关资源，相关地址：[http://www.dwheeler.com/secure-programs](http://www.dwheeler.com/secure-programs).

Android与其它大部分Linux环境最大的不同就在于程序沙箱。在Android中，所有的程序都运行在程序沙箱中，也包括那些本地代码。在最基本的层面上，对熟悉Linux的开发者来说，不同之处就是Android的每个应用程序都有一个唯一UID，也拥有少量的权限。如果要使用本地代码开发的话，那么应当对应用的权限极为了解才对。
