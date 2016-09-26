原文地址：[http://android.xsoftlab.net/training/keyboard-input/commands.html](http://android.xsoftlab.net/training/keyboard-input/commands.html)

当用户将焦点给到可编辑文本的View时，例如[EditText](http://android.xsoftlab.net/reference/android/widget/EditText.html)这种，并且该设备还拥有实体键盘，那么所有的输入都会被系统处理。然而，如果你希望可以拦截或者直接处理键盘的输入事件的话，你可以通过实现回调方法[KeyEvent.Callback](http://android.xsoftlab.net/reference/android/view/KeyEvent.Callback.html)接口来做到。比如[onKeyDown()](http://android.xsoftlab.net/reference/android/view/KeyEvent.Callback.html#onKeyDown(int, android.view.KeyEvent))和[onKeyMultiple()](http://android.xsoftlab.net/reference/android/view/KeyEvent.Callback.html#onKeyMultiple(int, int, android.view.KeyEvent))。

Activity与View都实现了[KeyEvent.Callback](http://android.xsoftlab.net/reference/android/view/KeyEvent.Callback.html)接口，所以通常你应该重写这两个类的回调方法。

>> **Note:** 当通过KeyEvent类或其它相关API处理键盘的输入事件时，应当认为这些键盘事件都来自于实体键盘。绝不要仰仗接收软键盘的按键事件。

##处理单个按键事件
如果要处理独立的按键事件，需要恰当的使用[onKeyDown()](http://android.xsoftlab.net/reference/android/app/Activity.html#onKeyDown(int, android.view.KeyEvent))方法或者[onKeyUp()](http://android.xsoftlab.net/reference/android/app/Activity.html#onKeyUp(int, android.view.KeyEvent))方法。通常情况下，如果要确保只有一个按键被按下时，应当只使用[onKeyUp()](http://android.xsoftlab.net/reference/android/app/Activity.html#onKeyUp(int, android.view.KeyEvent))方法。如果用户按下并没有放开某个按钮的话，那么[onKeyDown()](http://android.xsoftlab.net/reference/android/app/Activity.html#onKeyDown(int, android.view.KeyEvent))将会被调用多次。

举个例子，下面的实现通过响应某些按键来控制游戏：
```java
@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    switch (keyCode) {
        case KeyEvent.KEYCODE_D:
            moveShip(MOVE_LEFT);
            return true;
        case KeyEvent.KEYCODE_F:
            moveShip(MOVE_RIGHT);
            return true;
        case KeyEvent.KEYCODE_J:
            fireMachineGun();
            return true;
        case KeyEvent.KEYCODE_K:
            fireMissile();
            return true;
        default:
            return super.onKeyUp(keyCode, event);
    }
}
```

##处理组合按键
为了响应组合按键事件，比如某些按键需要与Shift或者Control组合使用，你可以查询通过回调方法传回的[KeyEvent](http://android.xsoftlab.net/reference/android/view/KeyEvent.html)对象。一些方法还为组合按键的提供了查询信息的功能，比如[getModifiers()](http://android.xsoftlab.net/reference/android/view/KeyEvent.html#getModifiers())和[getMetaState()。](http://android.xsoftlab.net/reference/android/view/KeyEvent.html#getMetaState())。不管如何，最简单的方案就是通过[isShiftPressed()](http://android.xsoftlab.net/reference/android/view/KeyEvent.html#isShiftPressed())或者[isCtrlPressed()](http://android.xsoftlab.net/reference/android/view/KeyEvent.html#isCtrlPressed())检查你所关心的组合按键是否被按下了。

举个例子，下面是onKeyUp()方法的改良版本，增添了一些专门对于Shift键的额外处理:
```java
@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    switch (keyCode) {
        ...
        case KeyEvent.KEYCODE_J:
            if (event.isShiftPressed()) {
                fireLaser();
            } else {
                fireMachineGun();
            }
            return true;
        case KeyEvent.KEYCODE_K:
            if (event.isShiftPressed()) {
                fireSeekingMissle();
            } else {
                fireMissile();
            }
            return true;
        default:
            return super.onKeyUp(keyCode, event);
    }
}
```

