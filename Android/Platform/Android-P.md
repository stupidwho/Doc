# Android-P

### 新技术适配

* 异形屏适配：适配市面上广泛存在的刘海屏、挖孔屏
*  Wi-Fi RTT 进行室内定位
* 多摄像头支持：国内带起的风潮，N摄加持

### 新增功能

* ImageDecoder：创建Drawable/Bitmap
* AnimatedImageDrawable：支持Gif图和WebP
* 自动填充功能
* 电源管理：新增应用待机群组，基于应用最近使用时间和使用频率，帮助系统排定应用请求资源的优先级

### API变更

* 后台传感器权限
* 限制通话记录、号码等权限
* 强制执行 FLAG_ACTIVITY_NEW_TASK 要求：不能从非 Activity 环境中启动 Activity，除非您传递 Intent 标志 FLAG_ACTIVITY_NEW_TASK