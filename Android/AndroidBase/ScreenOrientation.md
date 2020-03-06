# Android横竖屏切换

### 1.生命周期

每一次的横竖屏切换，都会导致Activity销毁并重新创建，会onPause-onStop-onDestroy-onCreate-onStart-onResume走一遍

### 2.onConfigurationChanged方法调用

当AndroidManifest中配置configChanged参数时（2.3以下版本配置orientation|keyboardHidden，2.3以上需配置orientation|keyboardHidden|screenSize，8.0以上只需配置orientation即可，具体版本分界不确定，没空一个个试），即可实现不重新走一遍生命周期，而只会调用onConfigurationChanged

### 3.多个Activity重叠

* 调用的顺序是按照栈的顺序，顶部activity会先经历一遍完整生命周期或者onConfigurationChanged，然后底部activity才会继续经历生命周期
* 如果顶层的activity是透明的activity，则能看到底下的activity，因此底下的这个activity只会调到onPause，屏幕旋转经历生命周期时会从onStop开始调用，然后到onCreate之后的周期

### 4.Android 8.1限制

当targetSdkVersion的版本>=27时，非全屏activity（包括透明的activity，具体包含那些style需自行看源码）禁止使用固定屏幕方向的方法或配置，否则会crash；是否固定屏幕方向需要由可见的全屏activity来确定，而透明activity这些非全屏activity被强制需要适配横竖屏的切换
