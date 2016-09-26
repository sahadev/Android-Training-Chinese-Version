原文地址：[http://android.xsoftlab.net/training/connect-devices-wirelessly/wifi-direct.html#permissions](http://android.xsoftlab.net/training/connect-devices-wirelessly/wifi-direct.html#permissions)

Wi-Fi peer-to-peer (P2P) APIs可以使程序与附近的设备进行直接通讯，Android的Wi-Fi P2P框架由Wi-Fi Direct™提供技术支持。WI-FI P2P技术可以使程序快速的检索附近的设备并与之建立连接。其覆盖范围超过蓝牙的覆盖范围。

这节课会学习如何通过WI-FI P2P技术搜索附近的设备并与之建立连接。

##设置应用权限
如果要使用WI-FI P2P技术，需要在程序的清单文件中添加[CHANGE\\_WIFI\\_STATE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#CHANGE_WIFI_STATE), [ACCESS\\_WIFI\\_STATE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#ACCESS_WIFI_STATE), [INTERNET](http://android.xsoftlab.net/reference/android/Manifest.permission.html#INTERNET)三项权限。Wi-Fi P2P并不需要互联网连接，但是它需要使用标准的Java Socket通讯技术，所以需要使用[INTERNET](http://android.xsoftlab.net/reference/android/Manifest.permission.html#INTERNET)权限:
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.nsdchat"
    ...
    <uses-permission
        android:required="true"
        android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission
        android:required="true"
        android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission
        android:required="true"
        android:name="android.permission.INTERNET"/>
    ...
```

##设置广播接收器及P2P管理员
使用WI-FI P2P技术，需要监听广播意图，广播意图会通知程序某些事件的发生。所以在程序中需要添加[IntentFilter](http://android.xsoftlab.net/reference/android/content/IntentFilter.html)，并设置其监听以下行为：

[WIFI\\_P2P\\_STATE\\_CHANGED\\_ACTION](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_STATE_CHANGED_ACTION)

    监听Wi-Fi P2P是否可用

[WIFI\\_P2P\\_PEERS\\_CHANGED\\_ACTION](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_PEERS_CHANGED_ACTION)

    监听WI-FI P2P列表的变化

[WIFI\\_P2P\\_CONNECTION\\_CHANGED\\_ACTION](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_CONNECTION_CHANGED_ACTION)

    监听Wi-Fi P2P的连接状态

[WIFI\\_P2P\\_THIS\\_DEVICE\\_CHANGED\\_ACTION](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_THIS_DEVICE_CHANGED_ACTION)

	监听设备的配置变化

```java
private final IntentFilter intentFilter = new IntentFilter();
...
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);
    //  Indicates a change in the Wi-Fi P2P status.
    intentFilter.addAction(WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION);
    // Indicates a change in the list of available peers.
    intentFilter.addAction(WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION);
    // Indicates the state of Wi-Fi P2P connectivity has changed.
    intentFilter.addAction(WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION);
    // Indicates this device's details have changed.
    intentFilter.addAction(WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION);
    ...
}
```

在[onCreate()](http://android.xsoftlab.net/reference/android/app/Activity.html#onCreate(android.os.Bundle))方法的末尾，需要获取[WifiP2pManager](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html)的实例，然后调用它的[initialize()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#initialize(android.content.Context,%20android.os.Looper,%20android.net.wifi.p2p.WifiP2pManager.ChannelListener))方法。这个方法会返回一个[WifiP2pManager.Channel](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.Channel.html)的对象，它用于使程序应用层与Wi-Fi P2P框架建立连接。
```java
@Override
Channel mChannel;
public void onCreate(Bundle savedInstanceState) {
    ....
    mManager = (WifiP2pManager) getSystemService(Context.WIFI_P2P_SERVICE);
    mChannel = mManager.initialize(this, getMainLooper(), null);
}
```

接下来创建一个新的BroadcastReceiver类，它用于监听系统的Wi-Fi P2P的状态变化，在onReceive()方法中，需要添加一些基本的判断条件来处理每种P2P的状态并处理：
```java
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION.equals(action)) {
            // Determine if Wifi P2P mode is enabled or not, alert
            // the Activity.
            int state = intent.getIntExtra(WifiP2pManager.EXTRA_WIFI_STATE, -1);
            if (state == WifiP2pManager.WIFI_P2P_STATE_ENABLED) {
                activity.setIsWifiP2pEnabled(true);
            } else {
                activity.setIsWifiP2pEnabled(false);
            }
        } else if (WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION.equals(action)) {
            // The peer list has changed!  We should probably do something about
            // that.
        } else if (WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION.equals(action)) {
            // Connection state changed!  We should probably do something about
            // that.
        } else if (WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION.equals(action)) {
            DeviceListFragment fragment = (DeviceListFragment) activity.getFragmentManager()
                    .findFragmentById(R.id.frag_list);
            fragment.updateThisDevice((WifiP2pDevice) intent.getParcelableExtra(
                    WifiP2pManager.EXTRA_WIFI_P2P_DEVICE));
        }
    }
```

最后，将广播接收器与意图过滤器添加到上下文中，并需要在Activity暂停的时候注销这个广播接收器。放置这些代码的最佳位置就是onResume()方法与onPause()方法。

```java
    /** register the BroadcastReceiver with the intent values to be matched */
    @Override
    public void onResume() {
        super.onResume();
        receiver = new WiFiDirectBroadcastReceiver(mManager, mChannel, this);
        registerReceiver(receiver, intentFilter);
    }
    @Override
    public void onPause() {
        super.onPause();
        unregisterReceiver(receiver);
    }
```

##初始化端点搜索
如果开始要使用Wi-Fi P2P来搜索附近的设备，需要调用[discoverPeers()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#discoverPeers(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.ActionListener))方法。这个方法要求传入以下参数：

- 在初始化P2P管理员时获得的[WifiP2pManager.Channel](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.Channel.html)对象。
- [WifiP2pManager.ActionListener](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html)的实现，它用于监听搜索的成功与否。

```java
mManager.discoverPeers(mChannel, new WifiP2pManager.ActionListener() {
        @Override
        public void onSuccess() {
            // Code for when the discovery initiation is successful goes here.
            // No services have actually been discovered yet, so this method
            // can often be left blank.  Code for peer discovery goes in the
            // onReceive method, detailed below.
        }
        @Override
        public void onFailure(int reasonCode) {
            // Code for when the discovery initiation fails goes here.
            // Alert the user that something went wrong.
        }
});
```

要记住，这里只是初始化了端点搜索。[discoverPeers()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#discoverPeers(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.ActionListener))方法启动搜索进程后会立即返回。如果端点搜索进程成功初始化，那么系统会自动调用初始化时设置的回调方法。另外，端点搜索功能会一直保持在活动状态，直到连接初始化完成或者P2P组建立连接。

##获取端点列表
接下来需要获得并处理端点列表。首先需要实现[WifiP2pManager.PeerListListener](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.PeerListListener.html)接口，它提供了WI-FI P2P所搜索到的端点信息。下面的代码演示了这个过程：
```java
    private List peers = new ArrayList();
    ...
    private PeerListListener peerListListener = new PeerListListener() {
        @Override
        public void onPeersAvailable(WifiP2pDeviceList peerList) {
            // Out with the old, in with the new.
            peers.clear();
            peers.addAll(peerList.getDeviceList());
            // If an AdapterView is backed by this data, notify it
            // of the change.  For instance, if you have a ListView of available
            // peers, trigger an update.
            ((WiFiPeerListAdapter) getListAdapter()).notifyDataSetChanged();
            if (peers.size() == 0) {
                Log.d(WiFiDirectActivity.TAG, "No devices found");
                return;
            }
        }
    }

```

现在需要修改广播接收器的onReceive()方法，在收到[WIFI\\_P2P\\_STATE\\_CHANGED\\_ACTION](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_STATE_CHANGED_ACTION)行为时调用[requestPeers()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#requestPeers(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.PeerListListener))方法。在这之前需要将监听器的实例传入到广播接收器中，常规的方式是在广播接收器的构造方法中将这个监听器传进来。、
```java
public void onReceive(Context context, Intent intent) {
    ...
    else if (WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION.equals(action)) {
        // Request available peers from the wifi p2p manager. This is an
        // asynchronous call and the calling activity is notified with a
        // callback on PeerListListener.onPeersAvailable()
        if (mManager != null) {
            mManager.requestPeers(mChannel, peerListListener);
        }
        Log.d(WiFiDirectActivity.TAG, "P2P peers changed");
    }...
}
```

现在，[WIFI\\_P2P\\_STATE\\_CHANGED\\_ACTION](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_STATE_CHANGED_ACTION)的行为将会触发端点列表的更新。

##连接到端点
为了可以连接到端点，需要创建一个新的[WifiP2pConfig](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pConfig.html)对象，然后将[WifiP2pDevice](WifiP2pDevice)中的数据拷贝进这个对象中。[WifiP2pDevice](WifiP2pDevice)代表的将要连接的设备。然后调用[connect()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#connect(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pConfig,%20android.net.wifi.p2p.WifiP2pManager.ActionListener))方法。
```java
    @Override
    public void connect() {
        // Picking the first device found on the network.
        WifiP2pDevice device = peers.get(0);
        WifiP2pConfig config = new WifiP2pConfig();
        config.deviceAddress = device.deviceAddress;
        config.wps.setup = WpsInfo.PBC;
        mManager.connect(mChannel, config, new ActionListener() {
            @Override
            public void onSuccess() {
                // WiFiDirectBroadcastReceiver will notify us. Ignore for now.
            }
            @Override
            public void onFailure(int reason) {
                Toast.makeText(WiFiDirectActivity.this, "Connect failed. Retry.",
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
```

上述代码中的[WifiP2pManager.ActionListener](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html)接口只有在初始化成功或者失败的情况下才会调用。如果要监听连接状态的变化，需要实现[WifiP2pManager.ConnectionInfoListener](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.ConnectionInfoListener.html)接口，它的方法[onConnectionInfoAvailable()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.ConnectionInfoListener.html#onConnectionInfoAvailable(android.net.wifi.p2p.WifiP2pInfo))会在连接状态发生变化的时候回调。在多台设备连接一台设备的情况下(比如多人互动的游戏或者聊天类的APP)，其中一台设备会被指定为"group owner"。
```java
    @Override
    public void onConnectionInfoAvailable(final WifiP2pInfo info) {
        // InetAddress from WifiP2pInfo struct.
        InetAddress groupOwnerAddress = info.groupOwnerAddress.getHostAddress());
        // After the group negotiation, we can determine the group owner.
        if (info.groupFormed && info.isGroupOwner) {
            // Do whatever tasks are specific to the group owner.
            // One common case is creating a server thread and accepting
            // incoming connections.
        } else if (info.groupFormed) {
            // The other device acts as the client. In this case,
            // you'll want to create a client thread that connects to the group
            // owner.
        }
    }
```

现在回到广播接收器的onReceive()方法，修改监听[WIFI\\_P2P\\_CONNECTION\\_CHANGED\\_ACTION](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_CONNECTION_CHANGED_ACTION)的部分，当这个意图接收到时，调用[requestConnectionInfo()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#requestConnectionInfo(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.ConnectionInfoListener))方法。这是一个异步方法，所以结果会通过参数:连接信息监听器回调回来。
```java
        ...
        } else if (WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION.equals(action)) {
            if (mManager == null) {
                return;
            }
            NetworkInfo networkInfo = (NetworkInfo) intent
                    .getParcelableExtra(WifiP2pManager.EXTRA_NETWORK_INFO);
            if (networkInfo.isConnected()) {
                // We are connected with the other device, request connection
                // info to find group owner IP
                mManager.requestConnectionInfo(mChannel, connectionListener);
            }
            ...
```

