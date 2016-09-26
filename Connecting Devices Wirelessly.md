原文地址：[http://android.xsoftlab.net/training/connect-devices-wirelessly/index.html](http://android.xsoftlab.net/training/connect-devices-wirelessly/index.html)

#引言
Android设备除了可以与服务器建立连接之外，Android无线API还允许使处于同一网段下的两台设备建立连接，或者是物理距离相近的两台设备建立连接。Network Service Discovery (NSD)允许应用程序通过扫描来搜索附近可连接的设备。当然被扫描的设备也同样需要开启该服务。利用这项特性可以使应用程序拥有更强大的功能，比如和同伴在一个房间内一同玩游戏，或者从有NSD服务的网络摄像头上下载照片，或者远程操控屋内的其它设备(智能家电的基础)。

这节课会学习如何搜索其它设备并其它设备建立连接的API。需要特别说明的是，这些关键的API描述了如何使用NSD API来搜索可用的服务及如何利用Wi-Fi Peer-to-Peer (P2P) API来建立P2P方式的无线连接。这节课还会学习如何使用NSD与WIFI P2P结合的方式来搜索服务提供者。

#[使用网络搜索服务](http://android.xsoftlab.net/training/connect-devices-wirelessly/nsd.html#teardown)
使用Network Service Discovery (NSD)服务可以使应用程序搜索处于同一本地网络下的其它支持服务要求的设备。这对多样的P2P应用程序极为有用，比如可以利用这项技术共享文件，或者玩多人游戏。Android的NSD API可以使应用程序快速实现这种功能。

这节课会学习如何构建一个应用程序，这个应用程序需要可以广播程序的名称及连接到本地网络的连接信息及扫描上述信息的功能。最后，这节课会学习如何去其它设备建立连接。

##将服务注册到网络
> **Note:** 这节课是可选的，如果不需要将应用的服务广播到本地网络，则可以直接跳到下一段。

为了将服务注册到本地网络上，首先需要创建一个NsdServiceInfo对象，这个对象提供了使其它设备决定是否要连接的信息。
```java
public void registerService(int port) {
    // Create the NsdServiceInfo object, and populate it.
    NsdServiceInfo serviceInfo  = new NsdServiceInfo();
    // The name is subject to change based on conflicts
    // with other services advertised on the same network.
    serviceInfo.setServiceName("NsdChat");
    serviceInfo.setServiceType("_http._tcp.");
    serviceInfo.setPort(port);
    ....
}
```

上面的代码将服务的名称设置为"NsdChat"。这个名称对所有使用了USD服务的设备可见。要记住一点就是这个名称在这个网络环境中必须是唯一的，Android会自动处理这种名称冲突问题。如果在同一环境下有两个名称为"NsdChat"的设备，那么Android会自动将其中一个命名为"NsdChat (1)".

第二个参数设置了该服务的类型，指定了所使用的传输协议与运输层。这个定义语法是"_<protocol>._<transportlayer>".在上面的代码中，该服务使用的是运行在TCP上的HTTP协议。如果一个程序提供的是打印服务，那么它的服务类型应该是"_ipp._tcp".

> **Note:** 这些搜索服务协议比如NSD、Bonjour由International Assigned Numbers Authority (IANA)互联网地址编码分配机构统一管理。你可以通过[the IANA list of service names and port numbers](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xml)下载服务类型列表。如果计算使用一种新的服务类型，应当在[IANA Ports and Service registration form](http://www.iana.org/form/ports-services)上进行备案。

当为服务设置端口时，应避免将端口写死，因为这个端口可能会与其他端口冲突。假设，应用程序一直在使用端口1337，则会与其它也使用了相同端口的应用程序造成端口冲突，使用设备的下一个可用端口则可以避免这种冲突。因为这个信息是由其它应用通过广播提供的，所以在编译时不需要知道其它应用使用的是哪个接口。相反的，应用程序会在连接到设备之前通过广播获得端口的信息。

如果使用Socket进行通信，下面的代码展示了如何初始化一个Socket，并将其端口设置为万能的端口0.
```java
public void initializeServerSocket() {
    // Initialize a server socket on the next available port.
    mServerSocket = new ServerSocket(0);
    // Store the chosen port.
    mLocalPort =  mServerSocket.getLocalPort();
    ...
}
```

那么现在已经定义了[NsdServiceInfo](http://android.xsoftlab.net/reference/android/net/nsd/NsdServiceInfo.html)对象，接下来还需实现RegistrationListener接口。这个接口用于通知应用程序服务的注册于解注是成功还是失败。
```java
public void initializeRegistrationListener() {
    mRegistrationListener = new NsdManager.RegistrationListener() {
        @Override
        public void onServiceRegistered(NsdServiceInfo NsdServiceInfo) {
            // Save the service name.  Android may have changed it in order to
            // resolve a conflict, so update the name you initially requested
            // with the name Android actually used.
            mServiceName = NsdServiceInfo.getServiceName();
        }
        @Override
        public void onRegistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {
            // Registration failed!  Put debugging code here to determine why.
        }
        @Override
        public void onServiceUnregistered(NsdServiceInfo arg0) {
            // Service has been unregistered.  This only happens when you call
            // NsdManager.unregisterService() and pass in this listener.
        }
        @Override
        public void onUnregistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {
            // Unregistration failed.  Put debugging code here to determine why.
        }
    };
}
```

那么现在已经完成了所有的前期准备工作，调用[registerService()](http://android.xsoftlab.net/reference/android/net/nsd/NsdManager.html#registerService(android.net.nsd.NsdServiceInfo,%20int,%20android.net.nsd.NsdManager.RegistrationListener))方法完成服务注册。

注意这个方法是异步方法，所以任何在服务完成注册之后运行的代码必须放在[onServiceRegistered()](http://android.xsoftlab.net/reference/android/net/nsd/NsdManager.RegistrationListener.html#onServiceRegistered(android.net.nsd.NsdServiceInfo))方法中执行。

```java
public void registerService(int port) {
    NsdServiceInfo serviceInfo  = new NsdServiceInfo();
    serviceInfo.setServiceName("NsdChat");
    serviceInfo.setServiceType("_http._tcp.");
    serviceInfo.setPort(port);
    mNsdManager = Context.getSystemService(Context.NSD_SERVICE);
    mNsdManager.registerService(
            serviceInfo, NsdManager.PROTOCOL_DNS_SD, mRegistrationListener);
}
```

##搜索网络上的服务
网络的世界充满了各种生命，从狂野的网络打印机到温柔的网络摄像头，再到激烈运动的体感游戏机，无不是一种生命。可以使应用程序能够看到这些各式各样的生命体的核心是服务搜索功能。应用程序只需要监听网络上的服务广播就可以判断哪些服务是可用的，哪些是不需要的。

服务搜索功能与服务的注册很类似，也有两个步骤：设置相应的搜索监听器，调用discoverServices()方法。

首先，实例化一个实现了NsdManager.DiscoveryListener接口的匿名类：
```java
public void initializeDiscoveryListener() {
    // Instantiate a new DiscoveryListener
    mDiscoveryListener = new NsdManager.DiscoveryListener() {
        //  Called as soon as service discovery begins.
        @Override
        public void onDiscoveryStarted(String regType) {
            Log.d(TAG, "Service discovery started");
        }
        @Override
        public void onServiceFound(NsdServiceInfo service) {
            // A service was found!  Do something with it.
            Log.d(TAG, "Service discovery success" + service);
            if (!service.getServiceType().equals(SERVICE_TYPE)) {
                // Service type is the string containing the protocol and
                // transport layer for this service.
                Log.d(TAG, "Unknown Service Type: " + service.getServiceType());
            } else if (service.getServiceName().equals(mServiceName)) {
                // The name of the service tells the user what they'd be
                // connecting to. It could be "Bob's Chat App".
                Log.d(TAG, "Same machine: " + mServiceName);
            } else if (service.getServiceName().contains("NsdChat")){
                mNsdManager.resolveService(service, mResolveListener);
            }
        }
        @Override
        public void onServiceLost(NsdServiceInfo service) {
            // When the network service is no longer available.
            // Internal bookkeeping code goes here.
            Log.e(TAG, "service lost" + service);
        }
        @Override
        public void onDiscoveryStopped(String serviceType) {
            Log.i(TAG, "Discovery stopped: " + serviceType);
        }
        @Override
        public void onStartDiscoveryFailed(String serviceType, int errorCode) {
            Log.e(TAG, "Discovery failed: Error code:" + errorCode);
            mNsdManager.stopServiceDiscovery(this);
        }
        @Override
        public void onStopDiscoveryFailed(String serviceType, int errorCode) {
            Log.e(TAG, "Discovery failed: Error code:" + errorCode);
            mNsdManager.stopServiceDiscovery(this);
        }
    };
}
```

NSD API会在这些情况下回调这些方法：搜索服务启动时，搜索服务失败时，服务未找到时，服务丢失时。注意在服务找到时会有以下若干项检查：

- 1.会对找到的服务名称与本身服务的名称相比较，来判断是否是自己发出的广播。
- 2.服务类型检查，验证找到的服务是否可以连接。
- 3.服务名称检查，验证是否连接到了正确的程序上。

服务名称的检查并不是必须的，只有在连接到指定的程序上才会这一步。比如，应用程序只是希望连接其它设备上的相同应用程序。无论如何，如果程序希望连接到网络打印机上时，只需要判断其服务类型是否是"_ipp._tcp"就足够了。

在设置完监听器之后，调用discoverServices()，然后传入程序要监听的服务类型，要使用的搜索协议，及刚刚创建的监听器。
```java
    mNsdManager.discoverServices(
        SERVICE_TYPE, NsdManager.PROTOCOL_DNS_SD, mDiscoveryListener);
```

##连接到网络服务
当应用程序找到网络上的服务并试图与之建立连接时，必须先通过[resolveService()](http://android.xsoftlab.net/reference/android/net/nsd/NsdManager.html#resolveService(android.net.nsd.NsdServiceInfo,%20android.net.nsd.NsdManager.ResolveListener))方法检查该服务的连接信息。实现NsdManager.ResolveListener接口，然后将其传入方法resolveService()中，然后通过这个接口的回调方法获得一个NsdServiceInfo对象，这个对象包含了所需要的连接信息：
```java
public void initializeResolveListener() {
    mResolveListener = new NsdManager.ResolveListener() {
        @Override
        public void onResolveFailed(NsdServiceInfo serviceInfo, int errorCode) {
            // Called when the resolve fails.  Use the error code to debug.
            Log.e(TAG, "Resolve failed" + errorCode);
        }
        @Override
        public void onServiceResolved(NsdServiceInfo serviceInfo) {
            Log.e(TAG, "Resolve Succeeded. " + serviceInfo);
            if (serviceInfo.getServiceName().equals(mServiceName)) {
                Log.d(TAG, "Same IP.");
                return;
            }
            mService = serviceInfo;
            int port = mService.getPort();
            InetAddress host = mService.getHost();
        }
    };
}
```

一旦解析完成，那么程序就会获得包括IP地址与端口号在内的详细服务信息。这为连接到该服务的网络提供了帮助。

##在程序关闭时注销服务
在程序的生命周期内可以适当关闭NSD功能是很重要的。在程序关闭时注销服务有助于防止其它程序认为该程序还处于存活状态，并试图与其连接。另外，服务搜索功能的操作代价很高，所以在Activity停止时应当关闭服务，在Activity重新启动时再次打开。应用程序应当重写对应的Activity的生命周期回调方法，并在这些方法中插入适当的代码来启动或者关闭服务广播与服务搜索功能。
```java
//In your application's Activity
    @Override
    protected void onPause() {
        if (mNsdHelper != null) {
            mNsdHelper.tearDown();
        }
        super.onPause();
    }
    @Override
    protected void onResume() {
        super.onResume();
        if (mNsdHelper != null) {
            mNsdHelper.registerService(mConnection.getLocalPort());
            mNsdHelper.discoverServices();
        }
    }
    @Override
    protected void onDestroy() {
        mNsdHelper.tearDown();
        mConnection.tearDown();
        super.onDestroy();
    }
    // NsdHelper's tearDown method
        public void tearDown() {
        mNsdManager.unregisterService(mRegistrationListener);
        mNsdManager.stopServiceDiscovery(mDiscoveryListener);
    }
```

