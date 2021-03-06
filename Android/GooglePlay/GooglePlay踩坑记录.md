# Google Play 踩坑实录

**注意：Google的文档一直处于不断的完善当中，相关的文档、接口可能半年就来个大变样，因此对许多笔记内容必须和时间进行关联才行，否则过时了，坑倒一大片**

### Google Play上架(2019-12-06)

#### 质量问题修改

* 上传应用后Google会检测代码，列出不合适的内容，跟进修改即可
* 关注应用的兼容性，除了版本还有许多特性需要关注。最坑的是NFC，国内经常会使用NFC Pay功能并且在Manifest中加入NFC特性，最终导致Google会判断应用仅能在有NFC功能的手机上使用，过滤大半的手机，即使已经上架，依然无法通过Google Play下载，会提示你的设备不兼容这个应用
* 可以在版本信息界面查看支持的设备，本人是通过分析这些设备的共通点才发现NFC问题，巨坑！

#### 审核

* 审核不严（上架海外版本，但是全是中文也依然能过），但是新应用首次上架，无论是Alpha、Beta还是正式版本，估计都需要一周左右的时间进行审核（2019-12-06）
* 上架这种命运控制在第三方手里的操作，必须尽早实施，且尽量去除违规内容

#### 官方推荐版本管理

##### 简述

* Alpha封闭测试版本：内部封闭测试使用
* Beta开放测试版本：外部开发测试使用
* 正式版本：正式上线版本

##### 相同点

* 均需要审核上架
* 均通过Google Play Store获取对应的apk
* Alpha版本虽然是通过链接进入，但最终都是跳转到Google Play Store获取，简单来说测试版本就是对入口的简单隐藏

##### 区别

* 正式版本可在Google Play Store搜索到，Beta版本（不完全明确）在搜索会看到，但是有特殊的提醒是Beta版，Alpha版本需要通过测试下载链接进入方可获取
* Alpha版本必须在开发者账号的对应版本中配置测试人员账号，只有配置的账号才能正常进入测试下载链接并下载。
* Alpha和Beta版本其余方面没有任何区别

### Google Play结算库服务(2019-12-06)

* 接入Google Play应用内购买是多年以来最坑爹的经验，耗时长，注意点多

#### 商品ID

* 需要配置收钱的商户
* 配置权限com.android.vending.BILLING并至少发布一个Alpha版本后商品ID才能生效且可以获取到
* 静态测试用商品ID只能用于最简单的客户端内购调起

#### 测试账号问题

* 开发者账号控制台-设置-许可测试，应用版本对应的管理测试人员，两个地方均可以添加测试账号，但是意义不同
* 许可测试：在测试内购的时候可以加进去，这样不需要真实给钱就可以测试“完整”购买调用流程，不锁区，会产生真实的订单（订单在后台会标记为测试订单），接口都可以正常回调对应支付成功的接口，只是不会真实扣钱
* 版本管理测试人员：配置了的账号才可以通过下载链接下载对应版本，与许可测试没有关联
* 测试账号的配置生效时间还是比较快的，但是并不完全确定，因此修改配置后至少一天后才完全确认配置生效

#### 账号锁区问题

* 实际上联想我们平时通过Google Play购买海外的游戏应用就可以得出相同的结论
* 如何创建当地账号：连当地网络（vpn方式），手机设置切换语言、区域、时间与当地匹配，清除Google三大件数据重启，注册账号并正常使用一段时间（网上找个visa生成器生成一下当地假卡号，添加一下支付方式，虽然加不成功，但是貌似这样操作一下就变当地人了）（2019-12-06），等Google的账号显示你是当地人，即创建成功
* 未上架应用必须从Google Play商店下载之后，商品才能在正式购买，言下之意：必须至少发一个Alpha版本，必须从当地Google Play Store下载，下载之后可通过覆盖安装的方式安装Debug版本进行调试，但必须保持签名、版本号一致，正式版本上架后不从Google Play下载也可以使用支付，但要确保版本号签名一致

#### 正式支付测试

* 可使用礼品卡测试正式内容，但是淘宝不会所有地区礼品卡都有卖
* 通过在其它热门地区上架，先跑通整个流程，最后再切换实际国家测试，但是并不推荐这种方式，因为有些问题可能发现不了（货币问题），不在当地环境跑一遍，终究不放心。
* 更好的方案是配置Google的测试账号，能完全走通流程，后续也是当地环境跑一遍即可

#### 货币问题

* Google商品定价，按填的收款银行账号货币定价，Google自动生成或手动调整商品当地价格
* Google Play 内购支付时实际展示当地货币价格
* 实际支付金额由Google在用户支付时即时换算成收款货币给到收款账号