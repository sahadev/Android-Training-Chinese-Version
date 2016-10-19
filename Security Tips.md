原文地址：[http://android.xsoftlab.net/training/articles/security-tips.html#IPC](http://android.xsoftlab.net/training/articles/security-tips.html#IPC)

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