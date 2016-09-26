原文地址：[http://android.xsoftlab.net/training/managing-audio/audio-focus.html](http://android.xsoftlab.net/training/managing-audio/audio-focus.html)

因为可能会存在多个APP播放音频，所以考虑它们之间的交互方式是一件很重要的事情。为了避免多个音乐播放器APP在同一时间播放音乐，Android使用了音频焦点的方式来管理音频的播放，只有获取了音频焦点的APP才可以播放音频。

在APP开始播放音频之前，APP需要请求以及接收音频的焦点。同样的，APP还应该知道如何监听音频焦点的丢失事件，以及当事件发生的时候，如何恰当的作出响应。

##请求音频焦点
在APP播放音频之前，APP应该对使用的音频流持有音频焦点。可以使用[ requestAudioFocus() ](http://android.xsoftlab.net/reference/android/media/AudioManager.html#requestAudioFocus(android.media.AudioManager.OnAudioFocusChangeListener,%20int,%20int))方法来获取焦点，如果请求成功的话，它会返回[ AUDIOFOCUS_REQUEST_GRANTED ](http://android.xsoftlab.net/reference/android/media/AudioManager.html#AUDIOFOCUS_REQUEST_GRANTED)。

你必须指定你所使用的音频流，并且还需要请求短暂的音频焦点，或者永久的音频焦点。当你想要播放一个短暂的音频时可以请求短暂的音频焦点(举例：当播放导航声音时)。当需要播放一个长时间的音频时，就需要请求永久的音频焦点(举例，播放音乐)。

下面的代码展示了对音乐音频流上请求了永久的音频焦点。你应该在开始播放之前立即请求音频焦点，比如当用户按下了播放按钮或者游戏中下节背景音乐开始之前。
```java
AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
...
// Request audio focus for playback
int result = am.requestAudioFocus(afChangeListener,
                                 // Use the music stream.
                                 AudioManager.STREAM_MUSIC,
                                 // Request permanent focus.
                                 AudioManager.AUDIOFOCUS_GAIN);
   
if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
    am.registerMediaButtonEventReceiver(RemoteControlReceiver);
    // Start playback.
}
```

一旦APP结束了播放，应该确保调用了[abandonAudioFocus()](http://android.xsoftlab.net/reference/android/media/AudioManager.html#abandonAudioFocus(android.media.AudioManager.OnAudioFocusChangeListener))方法。这会通知到系统你不再需要请求焦点，以及它会解注与之关联的[AudioManager.OnAudioFocusChangeListener](http://android.xsoftlab.net/reference/android/media/AudioManager.OnAudioFocusChangeListener.html)回调接口。在丢弃短暂焦点的例子中，它允许任何被中止的APP可以继续播放音频。
```java
// Abandon audio focus when playback complete    
am.abandonAudioFocus(afChangeListener);
```

当请求短暂的音频焦点时，有一些附加选项：无论你是否想要正常开启"ducking"选项，当一个友好的APP失去音频焦点的时候，它会立即停止播放。通过请求一个瞬态音频焦点时，它允许闪避，当你告诉其它音频APP它可以接收它们的持续播放，只要它们会降低音量直到音频焦点返回给它。
```java
// Request audio focus for playback
int result = am.requestAudioFocus(afChangeListener,
                             // Use the music stream.
                             AudioManager.STREAM_MUSIC,
                             // Request permanent focus.
                             AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK);
   
if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
    // Start playback.
}
```

闪避对间歇性的音频流APP来说特别适合，比如导航方向的播报信息。

当其它APP如上面描述的那样请求音频焦点时，它可以在永久的音频焦点或者短暂的音频焦点之间选择，这两个焦点会由请求焦点时注册的监听回调方法回调回来。

##处理音频焦点的丢失
如果APP可以请求到音频焦点，那么接下来在其它APP请求焦点的时候，它就可能会失去那个焦点。APP如何响应焦点的丢失，取决于丢失的方式。

音频焦点改变接口的回调方法[onAudioFocusChange()](http://android.xsoftlab.net/reference/android/media/AudioManager.OnAudioFocusChangeListener.html#onAudioFocusChange(int))会接收一个参数，这个参数用于描述焦点改变事件。特别的，焦点的丢失事件可能与原先部分请求的焦点类型相对应--永久丢失，短暂丢失以及短暂的限制性闪避。

通常来说，音频焦点的短暂丢失应该致使APP的音频流静音。否则就应该保持相同的状态。你应该继续监视音频焦点的改变，以便当APP恢复焦点的时候从暂停的地方恢复播放。

如果音频焦点的丢失是永久性的，假设现在有其他APP被用来监听音频焦点，这时，APP应该有效的自动终止。在实际情况中，这意味着停止播放，移除媒体按钮监听器--这运行一个新的音频播放器来专门处理这些事件，以及丢弃持有的音频焦点。在那时，在APP重新播放之前，你会期待有用户来触发那个请求。

在下面的代码中，如果音频丢失是短暂的，我们会暂停播放或者暂停我们的媒体播放器对象，并会在重新获得焦点时恢复它们。如果丢失是永久性的，它会解注我们的媒体按钮事件接收器以及停止对音频焦点改变的监听。
```java
OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
    public void onAudioFocusChange(int focusChange) {
        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT
            // Pause playback
        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
            // Resume playback 
        } else if (focusChange == AudioManager.AUDIOFOCUS_LOSS) {
            am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
            am.abandonAudioFocus(afChangeListener);
            // Stop playback
        }
    }
};
```

当闪避在允许的时候短暂的失去了音频焦点，不应该是暂停播放，而是"duck"。

##Duck!
Ducking 是降低音频流输出音量的过程，这可以使得其他APP的短暂音频听起来更轻快更和谐，而不会总是打断APP的播放过程。

下面的代码，当我们临时的失去焦点时，会降低媒体播放器对象的音量，当再一次获得焦点时，会将音量等级恢复至原来的值。
```java
OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
    public void onAudioFocusChange(int focusChange) {
        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK) {
            // Lower the volume
        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
            // Raise it back to normal
        }
    }
};
```

音频焦点的丢失是由大多数重要的广播影响的，但是不单单只是有它引起的。系统会广播一系列的意图来使你留意用户的听觉体验。下节课会演示如何监视音频焦点来增进用户的整体体验。