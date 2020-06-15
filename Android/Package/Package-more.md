# 一件项目打渠道包

## 一、简述

### 前言

* 为什么要研究怎么打渠道包？因为众多周知的原因，国内渠道多得不得了，国外就一个Google Play，并且生态原因导致谷歌不愿意替我们考虑太多打渠道包相关的功能，国内Coder唯有自立更生

### 已有项目的打包流程

* 正常流程：Release打包->Andresguard处理(混淆资源->签名->7zip压缩->zipalign)->Packer-Ng
* TinkerPatch打包：生成patch包前插入对编译的最新包进行

### 不同渠道的差异内容

* 不同位数so：本次遇到的需求，由于渠道的限制，需要将64位so也打包进apk中
* 渠道信息：只要能获取到不同渠道的信息，写在哪都没有关系
* 应用名称：AndroidManifest.xml的android:label
* 特殊配置：meta-data

### 常见打渠道包方案

* Packer-ng
* Walle
* 多Flavor
* 自己动手重新打包修改

## 二、自定义打包

### 目标

* 新增可打入64位so的渠道包
* 不同渠道对应不同的应用名称
* 兼容Tinker逻辑，只需编译一个patch包，所有渠道都可用
* 兼容andresguard逻辑，即资源混淆和7zip压缩逻辑

### 怎样新增64位so

* 原理：引用什么abi的so并没有被写进apk里面的配置当中，而是在安装的过程中直接读取apk中lib下的对应abi文件夹中的so文件，因此只需在apk中加入完整对应的so即可
* **1.Split方案**：使用官方提供的Split配置，可以直接将apk按照abi、density分别切割生成不同apk（app-arm64-v8a.apk，app-armeabi-v7z。。。）
* 优点：官方提供的配置，不需要整体编译
* 缺点：andresguard不兼容
* **2.多flavor方案**：针对不同渠道flavor配置不同的abifilter
* 优点：andresguard、tinker可以兼容
* 缺点：每个渠道都需要完整编译一遍，多一个渠道，编译时间成倍增加，无法接受，且tinker能否正常使用也十分怀疑
* **3.自定义重打包**：解包apk->新增64位so->重新打包->签名->zipalign
* 优点：速度快
* 缺点：andresguard兼容问题，但是可解决

### 应用名修改方案

* **1.旧方案**：apktool解包->修改Manifest->apktool重新打包->签名->zipalign
* 问题：资源被解包，andresguard失效，tinker资源patch失效（dex文件patch成功）
* **2.旧方案改进**：apktool解包->修改Manifest->apktool重新打包->andresguard再次处理
* 问题：apktool解包的不完美，导致resguard无法再复原最初的资源混淆模样，tinker资源patch依旧失效
* **2.多Flavor的方式**：每个渠道对应一个Flavor
* 问题：编译时间成倍增长
* **3.使用aapt进行打包**：调用系统的aapt工具在修改Manifest后再次对所有资源进行重新打包
* 问题：只能对完整资源打包，单独修改某个文件并不行
* **4.7zip方案**：将Apk作为一个Zip压缩包进行处理，解压、修改其中的内容、重新压缩
* 问题：需要对二进制内容进行修改

## 三、解包打包不简单

### Apktool

* 用什么进行解包再打包，很多人的第一反应就是apktool
* 解包混淆过的apk，加-r会解不出资源，最终没法再重新打包回去，需要-r对资源进行
* 然而，Apktool的解包会导致资源文件夹变化，最终导致的结果是apk的资源路径和原apk资源路径不一致，资源混淆出现偏差

### Zip方式解包打包

* 为什么使用Zip方式处理Apk？一切源于一句话：Apk文件实际上就是一个zip文件
* 因为Apktool解包和我们想要的解包不一样，Apktool解包不仅会对Apk进行解压，同时还会把内容根据Android标准进行解析，生成可读性强的内容，即最原始的内容被破坏了，这不是我想要的，我要的是除了我修改过的内容，其余东西都不变，因此，将apk作为zip包的直接解压才是最理想的方案

### AndroidManifest怎样修改

* 从[这篇文章](https://blog.zongheng.pro/2019/09/21/fast-change-androidmanifest-in-apk/)得到的启发
* apk中的Manifest是被处理成二进制的文件，不能直接处理
* [AXMLEditor](https://github.com/fourbrother/AXMLEditor)可以对二进制xml直接修改，可惜没有更新的维护，当前版本无法使用
* [xml2axml](https://github.com/hzw1199/xml2axml)可以对Manifest进行解码编码

## 四、最终方案

* Release包->缓存apk中所有文件的压缩方法->zip解包->从build目录复制64位so->解压二进制Manifest->修改Manifest中的应用名、渠道->压缩Manifest回二进制->重新zip压缩回apk->签名->按缓存的方法再次使用7zip重新解压压缩->zipalign
* 这个方案尽可能的保留apk中所有文件的原有模样，从而让tinker的patch功能尽可能成功
* 修改应用名称直接修改Manifest的方式，而不是旧方案的修改string，是因为string修改会导致resource.arsc变化，这会导致tinker在patch资源时失败
* 使用xml2axml工具对Manifest进行解码编码
