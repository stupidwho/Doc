# Hybrid技术进阶

### 基本功能

* 通过应用内集成WebView的方式，即可使用本地界面，又可使用Web动态更新
* 使用JsBridge方式增强应用内Web页面的功能
* 使用Url拦截方式增强页面跳转等功能

### 性能优化

#### 提升原生WebView性能

原生WebView性能较差，通过替换腾讯X5内核之类第三方的sdk，可有部分提升

#### 提升资源加载速度

* 利用Http协议缓存机制进行资源的缓存（Cache-Control），即前端常用的优化机制
* 首次加载：通过预先打包资源/配置预先加载资源方式，部分资源使用本地访问
* 增量更新：通过配置列表进行文件增量更新

#### 体验提升

* 通过@JavaNativeInterface接口提供Android方法的调用，增加体验
* 监听WebView加载过程回调，使用原生的页面loading样式
* 提供Android控件调用方法，提升效果
* 第三方支付接入：调用Web页面进行支付，通过overrideUrl的回调拦截特定Url进行后续操作

#### 安全问题

* 登录、个人信息以及加密等功能，由Android端完成，提供相应的接口给js调用
* 建权验证：将Android提供的本地接口功能进行权限划分，对Url地址进行判断并分别提供相应的接口权限

### 测试相关

* 性能指标：1.元素加载时间 2.接口流量消耗 3.接口耗时 4.容器加载时间
* 测试框架：支付宝appium，anyproxy

