---
title: Android中使用ApkSigner进行签名以及签名轮换
categories: 
  - Android折腾之路
excerpt: Android打包apk时需要使用开发者证书进行签名，以前是keystore文件，现在是jks文件。如果使用不同的证书进行签名，相同包名的apk是无法覆盖安装进行升级的。那么，如果证书有效期到了怎么办？如果证书信息创建的时候都是瞎写的，现在想改了怎么办？应用的开发商换了，为了安全，新的开发商需要使用自己的证书怎么办？Google在Android9的时代就推出了v3版的签名方式，支持更换签名。但是说实在的，官方文档说的其实也不是特别明确，还是需要摸索和实践。
date: 2024-01-24 23:31:31
tags: 
---

## 问题

Android打包apk时需要使用开发者证书进行签名，以前是keystore文件，现在是jks文件。如果使用不同的证书进行签名，相同包名的apk是无法覆盖安装进行升级的。那么，如果证书有效期到了怎么办？如果证书信息创建的时候都是瞎写的，现在想改了怎么办？应用的开发商换了，为了安全，新的开发商需要使用自己的证书怎么办？Google在Android9的时代就推出了v3版的签名方式，支持更换签名。但是说实在的，官方文档说的其实也不是特别明确，还是需要摸索和实践。

## 参考

https://source.android.google.cn/docs/security/features/apksigning/v3?hl=zh-cn

https://developer.android.google.cn/tools/apksigner?hl=zh-cn#options-sign-general

https://wenchiching.wordpress.com/category/android/

## 实践

我准备了1个小的测试apk，命名为origin.apk。老开发者证书文件为old.jks，新开发者证书文件为new.jks，整体放在了本次的工作目录中。后续命令运行在此目录。

另外，为了方便执行命令，可以配置一下环境变量，主要把build-tools/xx.x.x和platform-tools两个目录配上，我们本次会用到的adb和apksigner就在这两个目录中，apksigner要选择高版本的，不能太旧，我使用的是build-tools/34.0.0中的apksigner（windows系统下可能是apksigner.bat，一样的）。

思路是这样的：我们先用两个证书分别签名origin.apk，生成old.apk，new.apk。然后再使用轮换签名的方法，生成一个过渡的old-new.apk。然后分别验证覆盖安装的情况。

期望效果是：old.apk可以升级到old-new.apk，old-new.apk可以直接升级到new.apk。

> 注意：使用本文的方法需要抛弃Android9以下的设备的支持。另外，命令中一定要指定统一的--rotation-min-sdk-version，这个版本的作用是，当apk被安装在大于等于这个系统版本的设备上时，才会启用签名轮换，否则将忽略。最低只能写28，再低没有意义了，因为不支持。

### 1. 分别签名

```
apksigner sign --ks old.jks --min-sdk-version 28 --out old.apk --in origin.apk
apksigner sign --ks new.jks --min-sdk-version 28 --out new.apk --in origin.apk
```

### 2. 生成轮换签名继承关系文件

> lineage文件的名字可以随意命名，可以理解为这是一个签名变换的历史，a换成b，b换成c，这些先后关系需要在文件中声明。打包apk时会用到。

```
apksigner rotate \
  --out ./lineage-old-to-new \
  --old-signer --ks old.jks \
  --new-signer --ks new.jks
```

### 3. 结合签名轮换记录对apk进行签名

```
apksigner sign --ks old.jks --next-signer --ks new.jks --lineage ./lineage-old-to-new --rotation-min-sdk-version 28 --min-sdk-version 28 --out old-new.apk --in origin.apk
```

### 4. 分别查看apk内签名情况

```
apksigner verify --print-certs [xxx.apk]
```

### 5. 如果有必要，也可以查看一下apk签名版本

```
apksigner verify -v [xxx.apk]
```

### 6. 尝试覆盖安装

```
adb install old.apk
adb install old-new.apk
adb install new.apk
```

以上顺序进行覆盖均可以平滑升级。然而：

```
adb install old.apk
adb install new.apk
```
或者
```
adb install old-new.apk
adb install old.apk
```

均会报错：
```
Failure [INSTALL_FAILED_UPDATE_INCOMPATIBLE: Package xxx signatures do not match previously installed version; ignoring!]
Performing Streamed Install
adb: failed to install xxx.apk: Failure [INSTALL_FAILED_UPDATE_INCOMPATIBLE: Package xxx signatures do not match previously installed version; ignoring!]
```

可见签名一旦变更就不能安装旧证书的apk了，升级是单向的，确保安全性。

