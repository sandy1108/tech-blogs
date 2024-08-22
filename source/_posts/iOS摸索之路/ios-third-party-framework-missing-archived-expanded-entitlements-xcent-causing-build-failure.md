---
title: iOS第三方framework库缺少archived-expanded-entitlements-xcent导致打包失败问题
categories: 
  - iOS摸索之路
excerpt: 打包时，run debug没有问题，但是archive后export会发生错误，无法进入到选择证书的环节。
date: 2017-06-19 10:23:00
tags: 
---

# 问题

打包时，run debug没有问题，但是archive后export会发生错误，无法进入到选择证书的环节。


![出现问题的提示框.webp](出现问题的提示框.webp)


log如下：

```
2017-06-16 14:10:32 +0000 [MT] [OPTIONAL] Didn't find archived user entitlements for <DVTFilePath:0x7fcc77e68b00:'/Users/zhangyipeng/Library/Developer/Xcode/Archives/2017-06-16/AppCanPlugin 2017-6-16 下午10.08.xcarchive/Products/Applications/AppCanPlugin.app/Frameworks/ReactiveObjC.framework'>: Error Domain=NSCocoaErrorDomain Code=4 "Item at "/Users/zhangyipeng/Library/Developer/Xcode/Archives/2017-06-16/AppCanPlugin 2017-6-16 下午10.08.xcarchive/Products/Applications/AppCanPlugin.app/Frameworks/ReactiveObjC.framework" did not contain a "archived-expanded-entitlements.xcent" resource." UserInfo={NSLocalizedDescription=Item at "/Users/zhangyipeng/Library/Developer/Xcode/Archives/2017-06-16/AppCanPlugin 2017-6-16 下午10.08.xcarchive/Products/Applications/AppCanPlugin.app/Frameworks/ReactiveObjC.framework" did not contain a "archived-expanded-entitlements.xcent" resource.}

2017-06-16 14:10:32 +0000 [MT] [OPTIONAL] Didn't find archived user entitlements for <DVTFilePath:0x7fcc80b1dd00:'/Users/zhangyipeng/Library/Developer/Xcode/Archives/2017-06-16/AppCanPlugin 2017-6-16 下午10.08.xcarchive/Products/Applications/AppCanPlugin.app/Frameworks/RSKImageCropper.framework'>: Error Domain=NSCocoaErrorDomain Code=4 "Item at "/Users/zhangyipeng/Library/Developer/Xcode/Archives/2017-06-16/AppCanPlugin 2017-6-16 下午10.08.xcarchive/Products/Applications/AppCanPlugin.app/Frameworks/RSKImageCropper.framework" did not contain a "archived-expanded-entitlements.xcent" resource." UserInfo={NSLocalizedDescription=Item at "/Users/zhangyipeng/Library/Developer/Xcode/Archives/2017-06-16/AppCanPlugin 2017-6-16 下午10.08.xcarchive/Products/Applications/AppCanPlugin.app/Frameworks/RSKImageCropper.framework" did not contain a "archived-expanded-entitlements.xcent" resource.}

```

# 原因

可以看到，大体意思是说，打包时引用的一个framework文件，缺少了个资源叫"archived-expanded-entitlements.xcent"，网上查了一下资料好多都说是xcode6之后生成的。唉，这种回答都不靠谱，没有追根究底啊，有谁知道求教~


![旧的framework.webp](旧的framework.webp)


![新的framework.webp](新的framework.webp)


对比了一下新旧版本的framework库内容，发现RSKImageCropper.framework旧版多了个_CodeSignature目录，大概是放签名文件的，还有就是一些头文件和资源的改动。怀疑可能是由于签名不正确或者其他原因所致。

# 解决

我去找了个新版的framework库，更新了一下，就一切正常了。
