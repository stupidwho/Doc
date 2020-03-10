# Android 10

### 新技术适配

* 折叠设备
* 5G支持
* 手势导航

#### 深色主题

* 继承官方提供的DayNight主题
* 使用ForceDark，添加android:forceDarkAllowed配置，由系统根据当前浅色界面展示深色界面

### 隐私权限

* 前后台位置权限：将位置信息权限读取分为前后台状态
* 外部存储分区存储
* 后台启动：

#### 设备唯一标识符禁用（IMEi、SN之类）

* 禁用目的：大数据、用户画像侵犯用户隐私
* 阻止设备跟踪：IMEI、序列号之类的信息禁止获取，设备无法确定唯一
* 唯一标识符最佳做法-海外：[Google官方做法](https://developer.android.com/training/articles/user-data-ids)
* 唯一标识符最佳做法-国内：[移动安全联盟](http://www.msa-alliance.cn/)