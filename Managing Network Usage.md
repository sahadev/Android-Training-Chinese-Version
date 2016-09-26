原文地址：[http://android.xsoftlab.net/training/basics/network-ops/managing.html](http://android.xsoftlab.net/training/basics/network-ops/managing.html)

这节课将会学习如何对网络资源的使用情况拥有更细粒度的控制力。如果应用程序经常执行大量的网络操作，那么程序应当提供一项设置，以便用户可以控制应用的数据习性，比如多久同步一次数据，是否只在WIFI情况下上传下载数据，是否使用移动数据流量等等。随着这些设置能力的提供，用户可以设置应用在接近网络流量限制的情况下禁止应用再次访问网络，因为用户可以直接控制应用程序可以使用多少数据流量。

##检测设备的网络连接状况
一台设备拥有多种类型的网络连接。这节课所关注的是使用WI-FI或者移动数据网络连接。有关全面的网络连接类型，请参见[ConnectivityManager](http://android.xsoftlab.net/reference/android/net/ConnectivityManager.html).

WIFI通常情况下很快，而移动数据通常按量收费，还很昂贵。APP的通常使用策略是在WIFI网络可用的情况下才去获取大量的数据。

在执行网络操作之前，最好是检查一下网络的连接状态。执行网络状态检查，通常会使用到下面的类：

- [ConnectivityManager](http://android.xsoftlab.net/reference/android/net/ConnectivityManager.html) ： 可以获取当前网络的连接状况，还可以在网络连接状况发生变化时通知应用程序。
- [NetworkInfo](http://android.xsoftlab.net/reference/android/net/NetworkInfo.html) ： 描述了指定类型的网络接口状态。

下面的代码测试了WIFI及移动数据的连接状态。它会检查这些网络接口是否可用及是否已连接：
```java
private static final String DEBUG_TAG = "NetworkStatusExample";
...      
ConnectivityManager connMgr = (ConnectivityManager) 
        getSystemService(Context.CONNECTIVITY_SERVICE);
NetworkInfo networkInfo = connMgr.getNetworkInfo(ConnectivityManager.TYPE_WIFI); 
boolean isWifiConn = networkInfo.isConnected();
networkInfo = connMgr.getNetworkInfo(ConnectivityManager.TYPE_MOBILE);
boolean isMobileConn = networkInfo.isConnected();
Log.d(DEBUG_TAG, "Wifi connected: " + isWifiConn);
Log.d(DEBUG_TAG, "Mobile connected: " + isMobileConn);
```
需要注意的是，不应当关注网络是否可用，而应该在每次执行网络操作之前检查[isConnected()](http://android.xsoftlab.net/reference/android/net/NetworkInfo.html#isConnected())，因为[isConnected()](http://android.xsoftlab.net/reference/android/net/NetworkInfo.html#isConnected())会处理这些状态：移动网络信号不好、飞行模式或者受限的后台数据。

有一种更简明的方式可以用来检查网络接口是否可用：[getActiveNetworkInfo()](http://android.xsoftlab.net/reference/android/net/ConnectivityManager.html#getActiveNetworkInfo%28%29)方法会返回一个[NetworkInfo](http://android.xsoftlab.net/reference/android/net/NetworkInfo.html)的实例，这个对象代表了所能搜索到的第一个已连接的网络接口，如果没有搜索到任何网络连接则会返回null，null代表了互联网络连接不饿用。
```java
public boolean isOnline() {
    ConnectivityManager connMgr = (ConnectivityManager) 
            getSystemService(Context.CONNECTIVITY_SERVICE);
    NetworkInfo networkInfo = connMgr.getActiveNetworkInfo();
    return (networkInfo != null && networkInfo.isConnected());
}  
```

你可以使用[NetworkInfo.DetailedState](http://android.xsoftlab.net/reference/android/net/NetworkInfo.DetailedState.html)查询更细粒度的网络状态，不过很少会被用到。

##管理网络的使用
可以通过实现一个参数设置的Activity来让用户控制网络资源的使用。

- 你可能只允许用户在WIFI网络状态下才可以上传视频资源。
- 你可能要允许用户设置在指定的条件下才去同步数据，比如：网络可用状态下，或者隔多长时间等等。

为了使APP可以支持网络的访问及网络使用的管理，那么清单文件中必须包含以下权限以及意图过滤器：

- 清单文件应当包含以下权限：
 - android.permission.INTERNET 允许应用程序可以访问网络插口(Socket)。
 - android.permission.ACCESS_NETWORK_STATE 允许应用程序可以访问网络信息。

- 你可以通过声明ACTION_MANAGE_NETWORK_USAGE的意图过滤器来指明当前的Activity提供了控制数据使用策略的功能。当应用中含有允许用户管理网络数据使用策略的Activity时，应当声明该意图过滤器。在这里的示例程序中，这个行为被SettingsActivity所处理，这个Activity允许用户决定什么时候开始下载。
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.networkusage"
    ...>
    <uses-sdk android:minSdkVersion="4" 
           android:targetSdkVersion="14" />
        
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <application
        ...>
        ...
        <activity android:label="SettingsActivity" android:name=".SettingsActivity">
             <intent-filter>
                <action android:name="android.intent.action.MANAGE_NETWORK_USAGE" />
                <category android:name="android.intent.category.DEFAULT" />
          </intent-filter>
        </activity>
    </application>
</manifest>
```
##实现一个参数配置Activity
正如你在上面所看到的，SettingsActivity的意图过滤器含有一个ACTION_MANAGE_NETWORK_USAGE的行为，SettingsActivity是PreferenceActivity的子类，它的展示效果如下：
![这里写图片描述](http://android.xsoftlab.net/images/training/basics/network-settings1.png) ![这里写图片描述](http://android.xsoftlab.net/images/training/basics/network-settings2.png)

下面是SettingsActivity的代码，注意它实现了OnSharedPreferenceChangeListener接口。每当用户更改了参数，系统会调用onSharedPreferenceChanged()方法，该方法内将refreshDisplay设置为true，这是因为当用户返回到主界面是需要刷新界面。
```xml
public class SettingsActivity extends PreferenceActivity implements OnSharedPreferenceChangeListener {
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // Loads the XML preferences file
        addPreferencesFromResource(R.xml.preferences);
    }
  
    @Override
    protected void onResume() {
        super.onResume();
        // Registers a listener whenever a key changes            
        getPreferenceScreen().getSharedPreferences().registerOnSharedPreferenceChangeListener(this);
    }
  
    @Override
    protected void onPause() {
        super.onPause();
       // Unregisters the listener set in onResume().
       // It's best practice to unregister listeners when your app isn't using them to cut down on 
       // unnecessary system overhead. You do this in onPause().            
       getPreferenceScreen().getSharedPreferences().unregisterOnSharedPreferenceChangeListener(this);    
    }
  
    // When the user changes the preferences selection, 
    // onSharedPreferenceChanged() restarts the main activity as a new
    // task. Sets the refreshDisplay flag to "true" to indicate that
    // the main activity should update its display.
    // The main activity queries the PreferenceManager to get the latest settings.
    
    @Override
    public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String key) {    
        // Sets refreshDisplay to true so that when the user returns to the main
        // activity, the display refreshes to reflect the new settings.
        NetworkActivity.refreshDisplay = true;
    }
}
```

##响应参数的变更
当用户更改了参数时，这个行为会使APP的习性也跟着发生了变化。在下面的代码段中，APP会在onStart()方法中检查参数配置，如果在设备的当前连接状态与设置之间有相匹配的，那么APP将会下载信息，并刷新界面。

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
    
    // The user's current network preference setting.
    public static String sPref = null;
    
    // The BroadcastReceiver that tracks network connectivity changes.
    private NetworkReceiver receiver = new NetworkReceiver();
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // Registers BroadcastReceiver to track network connection changes.
        IntentFilter filter = new IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION);
        receiver = new NetworkReceiver();
        this.registerReceiver(receiver, filter);
    }
    
    @Override 
    public void onDestroy() {
        super.onDestroy();
        // Unregisters BroadcastReceiver when app is destroyed.
        if (receiver != null) {
            this.unregisterReceiver(receiver);
        }
    }
    
    // Refreshes the display if the network connection and the
    // pref settings allow it.
    
    @Override
    public void onStart () {
        super.onStart();  
        
        // Gets the user's network preference settings
        SharedPreferences sharedPrefs = PreferenceManager.getDefaultSharedPreferences(this);
        
        // Retrieves a string value for the preferences. The second parameter
        // is the default value to use if a preference value is not found.
        sPref = sharedPrefs.getString("listPref", "Wi-Fi");
        updateConnectedFlags(); 
       
        if(refreshDisplay){
            loadPage();    
        }
    }
    
    // Checks the network connection and sets the wifiConnected and mobileConnected
    // variables accordingly. 
    public void updateConnectedFlags() {
        ConnectivityManager connMgr = (ConnectivityManager) 
                getSystemService(Context.CONNECTIVITY_SERVICE);
        
        NetworkInfo activeInfo = connMgr.getActiveNetworkInfo();
        if (activeInfo != null && activeInfo.isConnected()) {
            wifiConnected = activeInfo.getType() == ConnectivityManager.TYPE_WIFI;
            mobileConnected = activeInfo.getType() == ConnectivityManager.TYPE_MOBILE;
        } else {
            wifiConnected = false;
            mobileConnected = false;
        }  
    }
      
    // Uses AsyncTask subclass to download the XML feed from stackoverflow.com.
    public void loadPage() {
        if (((sPref.equals(ANY)) && (wifiConnected || mobileConnected))
                || ((sPref.equals(WIFI)) && (wifiConnected))) {
            // AsyncTask subclass
            new DownloadXmlTask().execute(URL);
        } else {
            showErrorPage();
        }
    }
...
    
}
```

##监听连接变化
最后一个问题就是[BroadcastReceiver](http://android.xsoftlab.net/reference/android/content/BroadcastReceiver.html)的子类NetworkReceiver。当设备的网络连接发生变化时，NetworkReceiver会拦截CONNECTIVITY_ACTION的行为，这个行为用于检查当前是哪种网络连接状态，并会相应的将wifiConnected和mobileConnected设置为true或者false。那么在NetworkActivity.refreshDisplay设置为true时，那么APP会只下载最近一次的资源。

设置的广播监听器需要在系统不需要的情况下解除注册。示例应用中在onCreate()方法中将NetworkReceiver注册到系统，在onDestroy()方法中将其注销。这比在清单文件中注册更为轻量。当在清单文件中声明了广播接收器，系统会在任何时候调用该接收器，甚至是很久都没有启动过。在Activity中注册与注销广播接收器，可以确保用户在离开APP后系统不会再调用广播接收器。如果在清单文件中注册了广播接收器，那么你必须清楚在什么地方需要它，你可以适当的使用[setComponentEnabledSetting()](http://android.xsoftlab.net/reference/android/content/pm/PackageManager.html#setComponentEnabledSetting(android.content.ComponentName,%20int,%20int))方法来开启或者关闭它。

以下是 NetworkReceiver 的实现内容：
```java
public class NetworkReceiver extends BroadcastReceiver {   
      
@Override
public void onReceive(Context context, Intent intent) {
    ConnectivityManager conn =  (ConnectivityManager)
        context.getSystemService(Context.CONNECTIVITY_SERVICE);
    NetworkInfo networkInfo = conn.getActiveNetworkInfo();
       
    // Checks the user prefs and the network connection. Based on the result, decides whether
    // to refresh the display or keep the current display.
    // If the userpref is Wi-Fi only, checks to see if the device has a Wi-Fi connection.
    if (WIFI.equals(sPref) && networkInfo != null && networkInfo.getType() == ConnectivityManager.TYPE_WIFI) {
        // If device has its Wi-Fi connection, sets refreshDisplay
        // to true. This causes the display to be refreshed when the user
        // returns to the app.
        refreshDisplay = true;
        Toast.makeText(context, R.string.wifi_connected, Toast.LENGTH_SHORT).show();
    // If the setting is ANY network and there is a network connection
    // (which by process of elimination would be mobile), sets refreshDisplay to true.
    } else if (ANY.equals(sPref) && networkInfo != null) {
        refreshDisplay = true;
                 
    // Otherwise, the app can't download content--either because there is no network
    // connection (mobile or Wi-Fi), or because the pref setting is WIFI, and there 
    // is no Wi-Fi connection.
    // Sets refreshDisplay to false.
    } else {
        refreshDisplay = false;
        Toast.makeText(context, R.string.lost_connection, Toast.LENGTH_SHORT).show();
    }
}
```

