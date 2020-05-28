# 打包基础

## Apktool

### 解包

* apktool d xxx.apk -f -r -s
* -s 不解压class，-r 不解压资源

### 打包

* apktool b xxxDir -o xxx.apk
* xxxDir需打包的文件夹

## 7zip

* apk的本质就是zip文件，因此通过对压缩包处理来实现重新打包
* 解压：7za x xxx.apk -tzip -oxxxDir
* 压缩：7za a xxx.apk -tzip
* 7za a 会打包当前路径的所有内容

## jarsigner

* jarsigner -verbose -keystore xxx.keystore xxx.apk xxx别名
* xxx别名 是密钥库设置的别名
