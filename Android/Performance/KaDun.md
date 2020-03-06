### 一、相关blog

[官网ANR介绍][1]

[1]: https://developer.android.com/topic/performance/vitals/anr.html

[Android显示绘制流程][2]

[2]: http://djt.qq.com/article/view/987

[性能调优][3]

[3]: https://www.csdn.net/article/2015-06-12/2824949

[官网TraceView][4]

[4]: https://developer.android.com/studio/profile/traceview.html#overview

[常见类型][5]

[5]: http://www.csdn.net/article/2015-06-12/2824949/4

### 二、ANR

巨卡，卡顿中的终极问题，这个是无法忍受的问题，因为界面几秒钟无响应，系统级要结束整个应用，和crash无异。

#### 检测方法
* 系统会有记录，只需获取相应文件进行分析即可，获取系统中的trace文件(/data/anr/trace.txt)
* 有ouc系统做这个操作，省了很多事，如果是独立自己实现，需要用FileObserver监控trace文件读写或者watchdog监控，然后记录anr信息（会在logcat中打印）

#### 分析流程
* 1.查看log，同时分析CPU占用，判断是否由于其它进程占用过多CPU资源导致anr，这类anr与本应用无关

```
04-24 16:02:02.382 1586-1635/system_process E/ActivityManager: ANR in com.meizu.account.pay (com.meizu.account.pay/.activity.FlymeCoinMainActivity)
                                                               PID: 13029
                                                               Reason: Input dispatching timed out (Waiting to send non-key event because the touched window has not finished processing certain input events that were delivered to it over 500.0ms ago.  Wait queue length: 24.  Wait queue head age: 69895.0ms.)
                                                               Load: 8.27 / 7.84 / 7.3
                                                               CPU usage from 9469ms to 0ms ago (2018-04-24 16:01:49.246 to 2018-04-24 16:01:58.716):
                                                                 98% 13029/com.meizu.account.pay: 98% user + 0.4% kernel
                                                                   98% 13029/izu.account.pay: 98% user + 0.4% kernel
                                                                 12% 1586/system_server: 3.8% user + 9.1% kernel / faults: 480 minor 3 major
                                                                   4.9% 1633/android.bg: 0.5% user + 4.4% kernel
                                                                   1.5% 1718/PowerManagerSer: 0.6% user + 0.9% kernel
                                                                                     。
                                                                                     。
                                                                                     。
                                                                   0.2% 2649/Binder:606_5: 0% user + 0.2% kernel
                                                                 1.5% 476/logd: 0.5% user + 1% kernel
                                                                   0.7% 479/logd.writer: 0.4% user + 0.3% kernel
                                                                   0.4% 11890/logd.reader.per: 0% user + 0.4% kernel
                                                                   0.4% 12853/logd.reader.per: 0.2% user + 0.2% kernel
                                                                 0.9% 2072/com.android.systemui: 0.5% user + 0.4% kernel / faults: 397 minor
                                                                   0.4% 2072/ndroid.systemui: 0.4% user + 0% kernel
                                                                   0.1% 2267/RenderThread: 0.1% user + 0% kernel
                                                                 0.7% 11889/
```
* 2.查看trace文件，找到对应的包名，找到main线程对应的堆栈信息，查看发生anr的位置；
```
suspend all histogram:	Sum: 360us 99% C.I. 6us-308us Avg: 60us Max: 308us
DALVIK THREADS (34):
"main" prio=5 tid=1 Runnable
  | group="main" sCount=0 dsCount=0 obj=0x74b0b590 self=0xe9d05400
  | sysTid=13029 nice=-10 cgrp=default sched=0/0 handle=0xeca0a534
  | state=R schedstat=( 216204705008 231558367 2704 ) utm=21588 stm=32 core=6 HZ=100
  | stack=0xff2d7000-0xff2d9000 stackSize=8MB
  | held mutexes= "mutator lock"(shared held)
  at com.meizu.account.pay.activity.FlymeCoinMainActivity.onClick(FlymeCoinMainActivity.java:405)
  at android.view.View.performClick(View.java:5692)
  at android.view.View$PerformClick.run(View.java:22596)
  at android.os.Handler.handleCallback(Handler.java:751)
  at android.os.Handler.dispatchMessage(Handler.java:95)
  at android.os.Looper.loop(Looper.java:154)
  at android.app.ActivityThread.main(ActivityThread.java:6422)
  at java.lang.reflect.Method.invoke!(Native method)
  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1003)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:893)
```

#### ANR类型
> * 1.按键响应超时，5s内没有响应用户操作；
> * 2.BroadcastReceiver在10s内没有执行完成，耗时操作应该启动一个service代替
> * 3.Service服务超过20s没响应

#### 如何排查
###### 1.耗时操作
文件读写
网络请求
复杂内容、图片等内容计算(例如Bitmap的矩阵转换计算、加密解密等计算)
使用Binder跨进程调用
线程死锁
###### 2.简单自查
* a.排查代码当中BroadcastReceiver的onReceive方法，是否有进行耗时操作，比如网络访问、文件读写、数据库操作等
* b.排查Service各个生命周期回调是否有耗时操作
* c.排查界面操作响应函数，UI线程调用是否有耗时操作
* d.onActivityResult()等UI线程回调的方法，容易遗忘并进行耗时操作
###### 3.StrictMode
StrictMode，预先检测是否有主线程进行耗时操作，在Application或者Activity的onCreate方法调用
ThreadPolicy 线程相关
VMPolicy 虚拟机相关
```
         StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                 .detectCustomSlowCalls() //API等级11，使用StrictMode.noteSlowCode
                 .detectDiskReads() // 磁盘读操作
                 .detectDiskWrites() // 磁盘写操作
                 .detectNetwork()   // 网络操作
                 .detectAll() //打开所有检测
                 .penaltyDialog() //弹出违规提示对话框
                 .penaltyLog() //在Logcat 中打印违规异常信息
                 .penaltyFlashScreen() //API等级11 造成屏幕闪烁
                 .permit*() // 关掉某项检测
                 .build())
         StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                 .detectLeakedSqlLiteObjects() // 检测数据库忘记关闭的情况导致的泄露
                 .detectLeakedClosableObjects() // 检测可关闭的对象没有关闭的情况，即实现Closeable的类
                 .detectLeakedRegistrationObjects() // 检测是否忘记释放BroadcastReceiver和ServiceConnection
                 .setClassInstanceLimit() // 设置某个实例的上限
                 .penaltyLog()
                 .penaltyDeath() //触发直接crash
                 .build())
```

> * 注意事项
> * 只在开发阶段启用StrictMode，发布应用或者release版本一定要禁用它。
> * 严格模式无法监控JNI中的磁盘IO和网络请求。
> * 应用中并非需要解决全部的违例情况，比如有些IO操作必须在主线程中进行。
> * 系统中也存在一些调用会导致报错，需要忽略，例如一些资源文件的读取
###### 4.常见ANR问题收集（部分从网上收集）

1.如果CPU使用量很少，说明主线程被BLOCK了，需要从为何阻塞的方向去查
2.如果IOwait很高，说明ANR有可能是主线程在进行I/O操作造成的，需要检查是否进行了IO操作
3.发生在消息队列处的anr，这个经常和底层或者其它应用相关
```
  atandroid.os.MessageQueue.nativePollOnce(Native Method)
  atandroid.os.MessageQueue.next(MessageQueue.java:119)
  atandroid.os.Looper.loop(Looper.java:110)
  at android.app.ActivityThread.main(ActivityThread.java:3688)
  at java.lang.reflect.Method.invokeNative(Native Method)
  atjava.lang.reflect.Method.invoke(Method.java:507)
 
  atcom.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:866)
  at 
  com.android.internal.os.ZygoteInit.main(ZygoteInit.java:624)
  at dalvik.system.NativeStart.main(Native Method) 
```
4.atdalvik.system.VMRuntime.trackExternalAllocation(NativeMethod)内存不足导致block在创建bitmap上

### 三、性能进一步优化

###### 1.过度绘制

打开手机的开发者选项中显示过度绘制区域，直接观察手机颜色：蓝色，淡绿，淡红，深红

###### 2.View层级过多

使用Hierarchyviewer检查UI层级是否会嵌套太多，从而影响性能，但是这个工具仅仅依靠开发自测

###### 3.Profile GPU rendering，柱状图显示界面渲染时间

* 当你的应用程序在运行时,你会看到一排柱状图在屏幕上,从左到右动态地显示,每一个垂直的柱状图代表一帧的渲染,越长的垂直柱状图表示这一帧需要渲染的时间越长.随着需要渲染的帧数越来越多,他们会堆积在一起,这样你就可以观察到这段时间帧率的变化.
* 蓝色代表测量绘制的时间,或者说它代表需要多长时间去创建和更新你的DisplayList.在Android中,一个视图在可以实际的进行渲染之前,它必须被转换成GPU所熟悉的格式,简单来说就是几条绘图命令,复杂点的可能是你的自定义的View嵌入了自定义的Path. 一旦完成,结果会作为一个DisplayList对象被系统送入缓存,蓝色就是记录了需要花费多长时间在屏幕上更新视图(说白了就是执行每一个View的onDraw方法,创建或者更新每一个View的Display List对象).
* 橙色部分表示的是处理时间,或者说是CPU告诉GPU渲染一帧的地方,这是一个阻塞调用,因为CPU会一直等待GPU发出接到命令的回复,如果柱状图很高,那就意味着你给GPU太多的工作,太多的负责视图需要OpenGL命令去绘制和处理.
* 红色代表执行的时间,这部分是Android进行2D渲染 Display List的时间,为了绘制到屏幕上,Android需要使用OpenGl ES的API接口来绘制Display List.这些API有效地将数据发送到GPU,最总在屏幕上显示出来.

### 四、常见卡顿问题解决方法
* UI异步加载
* AsyncTask，用于简单的异步回调，例如后台做个什么网络操作，UI起个loading，在结束后结束loading，注意默认为单个实例的线程池，此时不能起多个AsyncTask
* Thread+Handler，用于复杂一点的前后台交互
* Loader，官方为Activity和Fragment提供的一套类，结合Cursor数据库查询、网络请求等使用
* View.post，在UI线程中更新这个View

### 五、Systrace

* 进入monitor，收集trace相关内容，可选择时长、Buffer缓存大小以及需要收集的内容，包括界面、WebView等等相关内容
* 可根据提示查看对应出现问题的地方，或者自行寻找调用时间过长的地方，从而定位到性能出现瓶颈的位置
* 一般情况下查看UIThread、RenderThread这些主线程相关的内容，以及SurfaceFlinger这些系统绘制界面相关，找出调用时间过长的方法，从而最终定位问题所在

### 六、TraceView

* 代码内：
Debug.startMethodTracing("sample"); // 启动trace收集
Debug.stopMethodTracing(); //停止收集

* 工具：Monitor

* Timeline pane:查看各个线程调用情况，可只管看出出现长时间调用的模块

* Profile pane：具体查看某个方法的详细调用情况，包括调用时间，调用次数等等；主要有两种情况，一种是某个方法调用耗时太长，根据CPU使用时长排序即可找出，另一种是某个方法多次重复调用，可以按调用次数排序，并找出占用CPU时长较长的方法，找出瓶颈所在

* Systrace与TraceView区别：两个应该结合使用，Systrace能够更方便快速找到出现掉帧的区域方法，或者是调用时间过长的内容，但是对于一些重复性调用，具体内容的显示并不直观，而TraceView在找到对应的方法后，有CPU统计时间、调用次数等等内容，并且有关联函数之间的调用，能更快速找到性能需要优化的点。


### 七、流程规范
需在每次迭代中都进行以下检查
1.自查代码中可能出现ANR的地方
2.使用debug版本通过StrictMode完整跑流程，检测相关不合规的地方（AAF已有相应检查）
3.过度绘制检查
4.Profile GPU rendering检查

### 八、掉帧检测
* 1.BlockCanary,MainLooper.setMessageLogging，检查start和end时间，设置阈值判断是否卡顿
* 2.Choreographer，从帧率入手检测，可自行设定掉帧卡顿判断逻辑
* 3.直观看fps
