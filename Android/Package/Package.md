# 打包基础

## Apktool

### 解包

* apktool d xxx.apk -f -r -s
* -s 不解压class，-r 不解压资源

### 打包

* apktool b xxxDir -o xxx.apk
* xxxDir需打包的文件夹

## jarsigner

* jarsigner -verbose -keystore xxx.keystore xxx.apk xxx别名
* xxx别名 是密钥库设置的别名
