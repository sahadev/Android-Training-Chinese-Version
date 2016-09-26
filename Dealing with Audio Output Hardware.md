原文地址：[http://android.xsoftlab.net/training/managing-audio/audio-output.html](http://android.xsoftlab.net/training/managing-audio/audio-output.html)

当用户使用Android设备享受音频时，它有多重的可选择替代方案。大多数的设备内置了一套音频系统：扬声器以及有线耳机的耳机插孔，也有很多功能蓝牙连接，以及对A2DP音频的支持。

##检查所使用的硬件类型
APP的工作方式取决了音频输出的硬件类型。

你可以通过AudioManager查询当前所被路由的设备扬声器、有线耳机或者附属的蓝牙设备，就像下面的代码显示的这样:
```java
if (isBluetoothA2dpOn()) {
    // Adjust output for Bluetooth.
} else if (isSpeakerphoneOn()) {
    // Adjust output for Speakerphone.
} else if (isWiredHeadsetOn()) {
    // Adjust output for headsets
} else { 
    // If audio plays and noone can hear it, is it still playing?
}
```

##处理音频硬件输出的变动
当耳机被拔出设备时，或者蓝牙设备断开连接时，那么音频流会自动的被重新到内置的扬声器设备上。如果你在一个很高的音量下听音乐，那么这样的话就会有一个很吵闹的体验。

幸运的是，当事件发生时，Android系统会广播一个[ACTION_AUDIO_BECOMING_NOISY](http://android.xsoftlab.net/reference/android/media/AudioManager.html#ACTION_AUDIO_BECOMING_NOISY)意图。最佳的实践方式就是在播放音频的时候，对这个意图注册一个[BroadcastReceiver](http://android.xsoftlab.net/reference/android/content/BroadcastReceiver.html)来监听它。在音乐播放器的例子中，用户通常会希望暂停音乐的播放。然而如果在进行游戏，那么你可以选择大幅度的降低音量。
```java
private class NoisyAudioStreamReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (AudioManager.ACTION_AUDIO_BECOMING_NOISY.equals(intent.getAction())) {
            // Pause the playback
        }
    }
}
private IntentFilter intentFilter = new IntentFilter(AudioManager.ACTION_AUDIO_BECOMING_NOISY);
private void startPlayback() {
    registerReceiver(myNoisyAudioStreamReceiver(), intentFilter);
}
private void stopPlayback() {
    unregisterReceiver(myNoisyAudioStreamReceiver);
}
```

