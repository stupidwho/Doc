# Tinker初探

## Tinker

### 需注意问题

* 1.Android Studio与命令行编译有时表现会不一样，当出现问题时，先用命令行编译试试
* 2.Tinker和TinkerPatch是两部分内容，Tinker是核心开源内容，TinkerPatch基于Tinker进一步封装，更加傻瓜的调用，增加下载管理、然后圈钱

### 限制

* 更新包括检测、下载、合并、应用几个流程
* Tinker的API可控制检测逻辑，但是下载以及合并均需要时间，因此在检测到更新后太早杀进程会导致Patch不及时
* 合并后一定要杀进程重新启动才会应用到最新的Patch

### 可选

* TinkerPatch收费版：客户端简化接入、后台管理、后台监控、灰度发布、条件发布


## TinkerPatch

### 客户端功能

* 一键式接入，仅需几行代码即可接入，无需重构Application代码，so库自动反射
* 补丁版本管理、自动重试与自动回退
* 增加自动化配置，简化编译配置难度

### 后台功能

* Tinker后台单独配置接口
* 比例、数量灰度更新，条件更新，渠道更新
* 在线监控下载、应用、合成数量

## 局限性

### Manifest影响

**Tinker在合成的时候只会合成dex、资源、so等等内容，AndroidManifest由于是在应用安装时读取引用，因此补丁包即使包含了相关的代码也无法使用，代码可以正常合成，但是在使用时若通过AndroidManifest进行查找则会找不到**

* Activity限制：通过在Manifest中预留Activity的方式实现，因此process、configChanges、screenOrientation和特殊主题无法在补丁中配置，部分特殊主题的效果会出现异常（透明展示有问题）
* 无法新增Service、ContentProvider、BroadcastReceiver
* 组件IntentFilter无法增加，权限无法增加、metadata无法增加

### 部分可选解决方案

* screenOrientation、configChanges目前没有特殊需求，没有大的影响，process和主题配置问题会对代码实现有限制
* 组件无法新增，可通过预留Service、ContentProvider、BroadcastReceiver组件的方式，在后续需要使用时对逻辑进行封装并切换到预留的组件中使用
* BroadcastReceiver可通过在App中动态注册监听广播，主动调用补丁中的BroadcastReceiver
* 预先申请足够多的权限

### 对第三方SDK影响

* 某些第三方的sdk包含特殊Activity、BroadcastReceiver、权限，对于实现复杂的第三方SDK基本无法在补丁中使用