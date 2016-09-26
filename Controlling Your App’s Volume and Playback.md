原文地址：[http://android.xsoftlab.net/training/managing-audio/index.html](http://android.xsoftlab.net/training/managing-audio/index.html)

#引言
如果APP需要播放音频，允许用户可以控制音频的播放状态是很重要的一点。为了保证有极佳的用户体验，还有很重要的一点就是，APP需要管理音频的焦点来确保不会有多个APP同时播放音频。

在稍后的课程中，将会学习如何使APP响应物理按键的按下事件，这需要在播放音频时，请求音频的焦点，以及需要适当的响应由系统或者其它应用程序引起的音频焦点改变。

#控制APP的音量以及播放状态
一个良好的用户体验是可预测的。如果APP可以播放媒体，那么用户可以通过设备上的物理按键或者虚拟按键来控制APP的音量是非常重要的一点。比如蓝牙耳机或者头戴式耳机。


同样，在适当的情况下，通过APP所使用的音频流基础之上来控制播放，停止，暂停，跳过，以及原始的媒体播放按键都应该执行其各自的行为和功能。

##识别所用的音频流
创建可预测的音频体验的第一步是理解APP将使用的音频流。

Android对播放音乐、闹钟、通知以及电话铃声、系统声音、呼叫音量和DTMF铃声都维护了单独的音频流。这么做主要是允许用户可以控制每个流各自的音频。

大部分流都会受限于系统事件，所以除非APP是个闹钟应用，否则，几乎可以确定APP播放音频所使用的就是[STREAM_MUSIC](http://android.xsoftlab.net/reference/android/media/AudioManager.html#STREAM_MUSIC)流。

##使用物理按键控制APP声音的音量
默认情况下，按下音量键会修改当前正在活动的音频流的音量。如果APP当前没有播放任何东西，按下音量键只会调整铃声的音量。

如果正在使用一款游戏APP或者音乐APP，那么当用户按下音量键的时候调整音量是极好的，因为用户想要控制游戏或者音乐的音量，即使现在在两首歌之间或者当前的游戏界面上没有播放音乐。

你可能想试着监听音量按键的按下事件，然后修改音频流的音量。忍住这股冲动吧。Android提供了更方便的[setVolumeControlStream()](http://android.xsoftlab.net/reference/android/app/Activity.html#setVolumeControlStream(int))方法来让音量键直接用于到你所指定的音频流。

如果已经确认使用的音频流类型，你应该将其设置为音量流的目标。你应该确保这个调用在APP的生命周期之前，因为只需要在Activity的生命周期中调用一次，你应该在具有代表性的方法中调用它，比如onCreate()方法。这可以确保每当APP处于可见状态时，音量控制功能可用更符合用户的期望。

```java
setVolumeControlStream(AudioManager.STREAM_MUSIC);
```

从这点往后，每当目标activity或者fragment可见时，按下设备上的音量键会影响你所指定的音频流(在这个例子中是"音乐")。

##使用物理播放控制键来控制APP的音频播放
媒体播放按钮，如播放、暂停、停止、跳跃，和以前的一些手机和许多有线连接或无线连接的耳机。当用户按下了其中的某个键时，系统会广播一个[ACTION_MEDIA_BUTTON](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_MEDIA_BUTTON)行为的意图。

为了响应媒体按钮的点击事件，你需要在清单文件中注册一个[BroadcastReceiver](http://android.xsoftlab.net/reference/android/content/BroadcastReceiver.html)，以便监听这个广播行为：
```xml
<receiver android:name=".RemoteControlReceiver">
    <intent-filter>
        <action android:name="android.intent.action.MEDIA_BUTTON" />
    </intent-filter>
</receiver>
```

接收器的实现需要提取出哪一个键被按下而引起的广播。Intent会包含[EXTRA\_KEY\_EVENT](http://android.xsoftlab.net/reference/android/content/Intent.html#EXTRA_KEY_EVENT)这个依据，类[KeyEvent](http://android.xsoftlab.net/reference/android/view/KeyEvent.html)中包含了一列以KEYCODE\_MEDIA\_*开头的静态常量，这些静态常量代表了每个按下的媒体键，比如：KEYCODE\_MEDIA\_PLAY\_PAUSE及KEYCODE\_MEDIA\_NEXT。

下面这段代码展示了如何抽取媒体键的按下事件以及来影响媒体的播放：
```java
public class RemoteControlReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (Intent.ACTION_MEDIA_BUTTON.equals(intent.getAction())) {
            KeyEvent event = (KeyEvent)intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
            if (KeyEvent.KEYCODE_MEDIA_PLAY == event.getKeyCode()) {
                // Handle key press.
            }
        }
    }
}
```

因为可能存在多个程序想要监听媒体按钮的按下事件，所以APP应该接收媒体按钮按下事件时，你还必须通过编程来进行动态的控制。

下面的代码可以直接应用到APP中去，它可以通过[AudioManager](http://android.xsoftlab.net/reference/android/media/AudioManager.html)来注册与解注媒体按钮事件接收器。当被注册后，广播接收器会专门接收所有的媒体按钮广播。

```java
AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
...
// Start listening for button presses
am.registerMediaButtonEventReceiver(RemoteControlReceiver);
...
// Stop listening for button presses
am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
```

从通常情况上讲，APP应该在不活动时或者不可见时解注其它的接收器(比如onStop()回调方法)。然而，对媒体播放APP来说没有那么简单，实际上，当程序不可见以及不能够屏幕上的UI来控制媒体播放时，这时，通过广播响应媒体播放按钮事件就非常重要了。

更进一步的方法就是当程序获取或者失去音频焦点时注册或者解注媒体按钮事件接收器。这些知识将会在下节课详细讨论。