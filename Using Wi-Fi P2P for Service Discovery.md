原文地址：[http://android.xsoftlab.net/training/connect-devices-wirelessly/nsd-wifi-direct.html](http://android.xsoftlab.net/training/connect-devices-wirelessly/nsd-wifi-direct.html)

本阶段的第一节课 Using Network Service Discovery 展示了如何搜索本地网络服务。然而，使用WI-FI P2P搜索服务可以直接搜索附近的设备，而不需要专门通过本地网络。这项特性使得在没有本地网络或者热点的情况下还可以在不同的设备间进行通信。

虽然这里的API与NSD的API的目的很相似，但是实现的过程却完全不同。这节课展示了如何通过WI-FI P2P网络来搜索附近的可用服务。这节课建立在已经对Wi-Fi P2P API熟悉的基础之上。

##设置清单文件
如果要使用WI-FI P2P技术，需要在程序的清单文件中添加[CHANGE\\_WIFI\\_STATE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#CHANGE_WIFI_STATE), [ACCESS\\_WIFI\\_STATE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#ACCESS_WIFI_STATE), [INTERNET](http://android.xsoftlab.net/reference/android/Manifest.permission.html#INTERNET)三项权限。虽然Wi-Fi P2P并不需要互联网连接，但是它需要使用标准的Java Socket通讯技术，所以需要使用[INTERNET](http://android.xsoftlab.net/reference/android/Manifest.permission.html#INTERNET)权限:
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

##添加本地服务
如果程序提供了本地服务，还需要将该服务注册到搜索服务中。一旦本地服务完成注册，那么框架会自动的响应另一端点的搜索服务请求。

创建本地网络有以下过程：

- 1.创建一个[WifiP2pServiceInfo](http://android.xsoftlab.net/reference/android/net/wifi/p2p/nsd/WifiP2pServiceInfo.html)对象。
- 2.将服务的相关信息填入其中。
- 3.调用[addLocalService()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#addLocalService(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.nsd.WifiP2pServiceInfo,%20android.net.wifi.p2p.WifiP2pManager.ActionListener))方法完成本地服务注册。

```java
     private void startRegistration() {
        //  Create a string map containing information about your service.
        Map record = new HashMap();
        record.put("listenport", String.valueOf(SERVER_PORT));
        record.put("buddyname", "John Doe" + (int) (Math.random() * 1000));
        record.put("available", "visible");
        // Service information.  Pass it an instance name, service type
        // _protocol._transportlayer , and the map containing
        // information other devices will want once they connect to this one.
        WifiP2pDnsSdServiceInfo serviceInfo =
                WifiP2pDnsSdServiceInfo.newInstance("_test", "_presence._tcp", record);
        // Add the local service, sending the service info, network channel,
        // and listener that will be used to indicate success or failure of
        // the request.
        mManager.addLocalService(channel, serviceInfo, new ActionListener() {
            @Override
            public void onSuccess() {
                // Command successful! Code isn't necessarily needed here,
                // Unless you want to update the UI or add logging statements.
            }
            @Override
            public void onFailure(int arg0) {
                // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY
            }
        });
    }

```

##搜索附近的服务
Android会使用回调方法来通知应用程序有可用的服务，所以首先要做的就是设置该回调。创建一个[WifiP2pManager.DnsSdTxtRecordListener](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdTxtRecordListener.html)来监听传入的记录。这个记录由其它设备随意广播。当其中一条记录到达时，会将该设备的地址及其它相关的信息拷贝到一个外部的数据结构中，这样的话就可以晚一些访问。下面的代码假设这个记录包含一条"buddyname"的属性，用于识别用户的身份。
```java
final HashMap<String, String> buddies = new HashMap<String, String>();
...
private void discoverService() {
    DnsSdTxtRecordListener txtListener = new DnsSdTxtRecordListener() {
        @Override
        /* Callback includes:
         * fullDomain: full domain name: e.g "printer._ipp._tcp.local."
         * record: TXT record dta as a map of key/value pairs.
         * device: The device running the advertised service.
         */
        public void onDnsSdTxtRecordAvailable(
                String fullDomain, Map record, WifiP2pDevice device) {
                Log.d(TAG, "DnsSdTxtRecord available -" + record.toString());
                buddies.put(device.deviceAddress, record.get("buddyname"));
            }
        };
    ...
}

```

如要获取服务的相关信息，需要创建一个[WifiP2pManager.DnsSdServiceResponseListener](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdServiceResponseListener.html)接口。 这个接口会接收实际的连接信息。上面代码段中的Map对象将设备的地址与"buddy name"组成了键值对。服务响应监听器利用这项特性与DNS记录建立连接。一旦两个监听器都已经实现，那么将它们添加到[WifiP2pManager](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html)的[setDnsSdResponseListeners()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#setDnsSdResponseListeners(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.DnsSdServiceResponseListener,%20android.net.wifi.p2p.WifiP2pManager.DnsSdTxtRecordListener))方法中即可。
```java
private void discoverService() {
...
    DnsSdServiceResponseListener servListener = new DnsSdServiceResponseListener() {
        @Override
        public void onDnsSdServiceAvailable(String instanceName, String registrationType,
                WifiP2pDevice resourceType) {
                // Update the device name with the human-friendly version from
                // the DnsTxtRecord, assuming one arrived.
                resourceType.deviceName = buddies
                        .containsKey(resourceType.deviceAddress) ? buddies
                        .get(resourceType.deviceAddress) : resourceType.deviceName;
                // Add to the custom adapter defined specifically for showing
                // wifi devices.
                WiFiDirectServicesList fragment = (WiFiDirectServicesList) getFragmentManager()
                        .findFragmentById(R.id.frag_peerlist);
                WiFiDevicesAdapter adapter = ((WiFiDevicesAdapter) fragment
                        .getListAdapter());
                adapter.add(resourceType);
                adapter.notifyDataSetChanged();
                Log.d(TAG, "onBonjourServiceAvailable " + instanceName);
        }
    };
    mManager.setDnsSdResponseListeners(channel, servListener, txtListener);
    ...
}
```

接下来需要创建一个新的服务请求，然后将其作为参数调用[addServiceRequest()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#addServiceRequest(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.nsd.WifiP2pServiceRequest,%20android.net.wifi.p2p.WifiP2pManager.ActionListener))方法，这个方法同样需要一个监听器来反应成功还是失败。
```java
        serviceRequest = WifiP2pDnsSdServiceRequest.newInstance();
        mManager.addServiceRequest(channel,
                serviceRequest,
                new ActionListener() {
                    @Override
                    public void onSuccess() {
                        // Success!
                    }
                    @Override
                    public void onFailure(int code) {
                        // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY
                    }
                });
```

最后，调用[discoverServices()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#discoverServices(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.ActionListener))方法开始搜索服务。
```java
        mManager.discoverServices(channel, new ActionListener() {
            @Override
            public void onSuccess() {
                // Success!
            }
            @Override
            public void onFailure(int code) {
                // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY
                if (code == WifiP2pManager.P2P_UNSUPPORTED) {
                    Log.d(TAG, "P2P isn't supported on this device.");
                else if(...)
                    ...
            }
        });
```

如果上面的都已经完成，那么可以喊一声哈利路亚了，已经完成了所有的步骤。如果遇到了问题，寻找那个将[WifiP2pManager.ActionListener](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html)作为参数的方法，这个回调方法会告知程序是成功还是失败。如果要解决这个问题，请将调试代码放入[onFailure()](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html#onFailure(int))方法中。方法所提供的错误代码会告知问题所在。下面是可能出现的错误代码以及它们的解释：

[P2P\\_UNSUPPORTED](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#P2P_UNSUPPORTED)

        当前的设备不支持Wi-Fi P2P

[BUSY](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#BUSY)

        系统处于繁忙处理状态

[ERROR](http://android.xsoftlab.net/reference/android/net/wifi/p2p/WifiP2pManager.html#ERROR)

        由于内部错误造成的操作失败

