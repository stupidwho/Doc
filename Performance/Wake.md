# 手机唤醒

### 简述

正常情况下，有很多应用需要保持手机不息屏（视频app、游戏等等），或者息屏后后台运行（音乐后台播放），为了满足这些需求，Android提供了WakeLock、Alarm的方式给应用，使得这些应用能够提供更好的体验给用户，然而面对国内的流氓app，这个功能被疯狂滥用，甚至是各种奇葩手段绕过限制，手机的电量就是被这样强奸得无能为力，因此做好本分，优化好手机功耗显得尤为重要。

### WakeLock

[WakeLock保持唤醒]
https://developer.android.com/training/scheduling/wakelock

###### PARTIAL_WAKE_LOCK
比较常用且难以发现的唤醒锁，只要锁没有释放，应用可以在后台保持CPU唤醒并一直运行，同时屏幕可锁屏，这样会导致用户无法发现自己的手机是否真正的休眠了，很多后台下载、后台播放都是使用这个锁，但是也有很多其它不法应用使用这个锁保持后台运行，从而最终使手机无法休眠而异常耗电
```
PowerManager powerManager = (PowerManager) getSystemService(POWER_SERVICE);
WakeLock wakeLock = powerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,
        "MyWakelockTag");
wakeLock.acquire();
```

###### FLAG_KEEP_SCREEN_ON

保持屏幕常亮，像是看电影、玩游戏等等，用户有可能长时间不对手机进行操作，需要保持屏幕常亮，使用这个标志即可实现

```
// 保持屏幕常亮
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
}
<!-- 保持屏幕常亮 -->
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:keepScreenOn="true">
    ...
</RelativeLayout>
```


### AlarmManager

[设置定时唤醒]
https://developer.android.com/training/scheduling/alarms

设置某个时间点发出action，从而可以唤醒自己，及时应用没有在后台运行，也可以在某个时候启动

setInexactRepeating() // 不需要十分准确的唤醒，使得系统可以将多个唤醒操作合并到一起唤醒，减少多次唤醒的资源浪费
setRepeating() // 确定的时间唤醒

ELAPSED_REALTIME // 以开机时间或切换时间后多久时间开始算
RTC_WAKEUP // 确切地根据当前设定的时间算

### JobScheduler代替AlarmManager
用于并不需要太确定的时间执行的任务，网络连接的监听进行的操作

### 检测

[唤醒检查]
https://blog.csdn.net/green1900/article/details/41382279
https://www.jianshu.com/p/d78979b708e6

adb shell cat /sys/kernel/debug/wakeup_sources

adb shell dumpsys power
