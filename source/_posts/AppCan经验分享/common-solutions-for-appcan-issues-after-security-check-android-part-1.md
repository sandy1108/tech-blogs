---
title: AppCan应用在安全检测后的常见问题解决方案之Android篇（上）
categories: 
  - AppCan经验分享
excerpt: 
date: 2017-04-10 10:52:39
tags: 
---

# 写在前面

现在的安全检测机构都很套路，就这些东西，跑脚本自动检测，有的甚至也不管你里面是否已经做了保护，仅仅是有代码存在的话，就给你报出来，让你害怕的要命，让你不得不掏腰包去找所谓的第三方安全公司给加壳加保护。其实，有些问题并没有那么可怕。

本文将从个人角度，把自己之前遇到的各种检测出来的问题做梳理，并不代表某官方，也不代表到底权威不权威，仅仅是一个分享。

下面先介绍Android问题。因为安全问题一般都发生在Android中，还是比较多的。iOS相对封闭，破解的魔爪伸向iOS的还是相对较少的。


# 问题列举

1. 检测项目：重要函数逻辑安全

- 关键词

函数、方法、反编译

- 安全报告解释

因为APK本身因为未进行专业加固保护，存在被baksmali/apktool/dex2jar直接反编译获取程序java代码，如果程序的重要函数使用android ndk技术通过c/c++实现能够提高重要函数的逻辑安全强度。 Elf格式的SO文件如果未进行专业的加固保护可能存在被ipa pro工具一键F5逆向出c代码被分析的风险。

- 详细报告

```
检测出重要函数共【177】条 ： 
Ljava/net/URLEncoder;->encode (com/tencent/open/GameAppOperation.smali)...
...    check-cast v1, Ljava/lang/String;
    invoke-virtual {v1}, Ljava/lang/String;->trim()Ljava/lang/String;
    move-result-object v1
    const-string v15, "UTF-8"
    invoke-static {v1, v15}, Ljava/net/URLEncoder;->encode(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
    move-result-object v1
    invoke-virtual {v13, v1}, Ljava/lang/StringBuffer;->append(Ljava/lang/String;)Ljava/lang/StringBuffer;
    :try_end_0
    .catch Ljava/io/UnsupportedEncodingException; {:try_start_0 .. :try_end_0} :catch_0
    .line 345
以下省略......
...
Ljava/security/KeyStore;->getInstance 、 Ljava/net/URLEncoder;->encode (com/tencent/open/utils/HttpUtils.smali)...
...    .line 714
    :goto_2
    new-instance v5, Ljava/lang/StringBuilder;
    invoke-direct {v5}, Ljava/lang/StringBuilder;-><init>()V
    invoke-static {v0}, Ljava/net/URLEncoder;->encode(Ljava/lang/String;)Ljava/lang/String;
    move-result-object v6
    invoke-virtual {v5, v6}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;
    move-result-object v5
    const-string v6, "="
    invoke-virtual {v5, v6}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;
以下省略......
...
Landroid/telephony/TelephonyManager;->getDeviceId 、 Landroid/telephony/TelephonyManager;->getSubscriberId 、 Landroid/telephony/TelephonyManager;->getCellLocation (com/baidu/location/aj.smali)...
...    :goto_0
    return-void
    :cond_0
    iget-object v0, p0, Lcom/baidu/location/aj;->Y:Landroid/telephony/TelephonyManager;
    invoke-virtual {v0}, Landroid/telephony/TelephonyManager;->getCellLocation()Landroid/telephony/CellLocation;
    move-result-object v0
    invoke-direct {p0, v0}, Lcom/baidu/location/aj;->a(Landroid/telephony/CellLocation;)V
    goto :goto_0
.end method
.method private f()Lcom/baidu/location/aj$a;
以下省略......
...
Ljava/net/URLDecoder;->decode (com/tencent/connect/auth/AuthAgent$FeedConfirmListener.smali)...
...    move-result-object v0
    move v1, v2
    .line 486
    :goto_0
    invoke-static {v0}, Ljava/net/URLDecoder;->decode(Ljava/lang/String;)Ljava/lang/String;
    move-result-object v0
    .line 487
    const-string v2, "TAG"
    new-instance v3, Ljava/lang/StringBuilder;
    invoke-direct {v3}, Ljava/lang/StringBuilder;-><init>()V
以下省略......
...
Landroid/telephony/TelephonyManager;->getCellLocation (com/iflytek/thirdparty/ac.smali)...
...    invoke-static {v0}, Lcom/iflytek/thirdparty/X;->c(Ljava/lang/String;)V
    goto/16 :goto_0
    :pswitch_0
    :try_start_1
    invoke-virtual {v0}, Landroid/telephony/TelephonyManager;->getCellLocation()Landroid/telephony/CellLocation;
    move-result-object v0
    check-cast v0, Landroid/telephony/cdma/CdmaCellLocation;
    invoke-virtual {v0}, Landroid/telephony/cdma/CdmaCellLocation;->getBaseStationId()I
    move-result v1
    invoke-virtual {v0}, Landroid/telephony/cdma/CdmaCellLocation;->getNetworkId()I
以下省略......
...
Ljava/net/URLEncoder;->encode (com/autonavi/amap/mapcore/NormalMapLoader.smali)...
...    const-string v7, "-"
    invoke-virtual {v5, v7}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;
    move-result-object v5
    const-string v7, "utf-8"
    invoke-static {v0, v7}, Ljava/net/URLEncoder;->encode(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
    move-result-object v0
    invoke-virtual {v5, v0}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;
    move-result-object v0
    invoke-virtual {v0}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;
    :try_end_0
以下省略......
...
... 此次省略【172】条数据...
```

- 解决方案

从错误分析看，风险代码均存在于第三方插件，如高德地图插件、科大讯飞插件、定位插件（用到了百度SDK）、腾讯旗下某插件（猜测可能是uexTent）。可以将这些插件升级到最新版本，如果仍然有问题，则需要联系开发商解决。

不过，这种问题其实主要要看app内的业务场景，如果不涉及太机密的东西，这个问题的风险并不大。

2. 检测项目：Activity最小化特权检测

- 关键词

android:exported="false"，拒绝服务漏洞，intent

- 安全报告解释

应用组件如果存在权限攻击漏洞则该组件能够被外部的其他组件直接调用，这样就可能产生泄露隐私数据或者应用程序崩溃等漏洞。恶意应用可以传递有害数据或者命令给受害的broadcast receiver，而receiver接收到有害的数据或者命令时可能泄露数据或者做一些不当的操作。也有可能receiver去开启其它的activity或者service，从而产生更大的危害。 activity被恶意应用开启，可能有一下危害：修改程序的状态或者数据；用户被欺骗（比如用户点击一个恶意应用的setting，恶意应用开启受害应用的设置，此时用户以为在修改恶意应用的setting，这样受害应用的设置可能被用户无意识的修改）；被调用的activity可能返回隐私的信息给恶意应用，造成数据泄露；可能会是应用程序崩溃，造成拒绝服务等漏洞。

- 详细报告

```
总共检测Activity配置代码【15】条;
检测到未进行正确配置的代码【4】条;
检测到正确配置的代码【11】条;

未进行正确配置的代码为：
activity android:alwaysRetainTaskState="true" android:configChanges="keyboardHidden|orientation|screenSize" android:launchMode="singleTask" android:name="org.zywx.wbpalmstar.engine.EBrowserActivity" android:screenOrientation="portrait" android:theme="@style/browser_main_theme" android:windowSoftInputMode="adjustResize|stateHidden"
activity android:launchMode="singleTask" android:name="com.tencent.tauth.AuthActivity" android:noHistory="true"
activity android:exported="true" android:launchMode="singleTop" android:name="com.club.mobile.wxapi.WXEntryActivity" android:theme="@style/plugin_uexweixin_Transparent"
activity android:exported="true" android:launchMode="singleTop" android:name="com.club.mobile.wxapi.WXPayEntryActivity" android:theme="@style/plugin_uexweixin_Transparent"
正确Activity组件配置的代码为：
activity android:configChanges="keyboardHidden|navigation|orientation" android:name="org.zywx.wbpalmstar.platform.mam.PolicyActivity" android:windowSoftInputMode="adjustPan"
activity android:configChanges="keyboardHidden|navigation|orientation" android:name="org.zywx.wbpalmstar.platform.mam.PolicyInfoActivity" android:windowSoftInputMode="adjustPan"
activity android:launchMode="standard" android:name="org.zywx.wbpalmstar.engine.TempActivity" android:screenOrientation="portrait" android:theme="@style/browser_loading_theme"
activity android:configChanges="keyboardHidden|orientation" android:exported="false" android:name="org.zywx.wbpalmstar.plugin.uexcamera.CustomCamera" android:theme="@android:style/Theme.NoTitleBar.Fullscreen"
activity android:exported="false" android:label="@string/plugin_camera_title_activity_second" android:name="org.zywx.wbpalmstar.plugin.uexcamera.ViewCamera.SecondActivity" android:theme="@android:style/Theme.Holo.Light"
...
此处省略【6】条数据
...
```

- 解决方案

首先，此问题在AppCan的框架老版本中确实存在，但都不是很危险的行为。使用最新的Android引擎版本都已经尽可能解决了此问题（3.4+的版本）。但是还有很多组件必须暴露，也都已经做好了保护。

至于某些第三方插件，就不好说了。

如果某自定义插件检测出这种问题，解决方法就是，将对应的组件在manifest文件中声明为android:exported="false"。配置之后，在应用外将无法调用这个组件。

如果这个组件必须暴露，则需要在处理intent的地方进行判断以及trycatch，防止恶意程序传参后导致app中intent遍历参数错误导致崩溃。

具体原理可以参考：http://jaq.alibaba.com/blog.htm?id=55 

- 详细解释

org.zywx.wbpalmstar.engine.EBrowserActivity属于引擎框架目前必须暴露的组件，已经做了保护处理，所以此处很安全；
.wxapi.WXPayEntryActivity和.wxapi.WXPayEntryActivity都是uexWeixin插件必备的组件，也可以放心跳过；
com.tencent.tauth.AuthActivity是腾讯微博中的必备组件。

3. 检测项目：Intent检测

- 关键词

Intent、拒绝服务漏洞

- 详细报告

```
检测出隐式Intent跳转【92】条 ： 
android.intent.action.VIEW 、 android.intent.action.VIEW 、 android.intent.action.VIEW 、 android.intent.action.VIEW...
...    aput-object v8, v6, v7
    invoke-static {v1, v2, v3, v6}, Lcom/tencent/connect/a/a;->a(Landroid/content/Context;Lcom/tencent/connect/auth/QQToken;Ljava/lang/String;[Ljava/lang/String;)V
    .line 386
    new-instance v1, Landroid/content/Intent;
    const-string v2, "android.intent.action.VIEW"
    invoke-direct {v1, v2}, Landroid/content/Intent;-><init>(Ljava/lang/String;)V
    move-object/from16 v0, p0
    iput-object v1, v0, Lcom/tencent/open/GameAppOperation;->mActivityIntent:Landroid/content/Intent;
    .line 387
    move-object/from16 v0, p0
以下省略......
... (com/tencent/open/GameAppOperation.smali)
android.intent.action.VIEW 、 android.intent.action.VIEW 、 android.intent.action.VIEW 、 android.intent.action.VIEW...
...    if-eqz p1, :cond_0
    .line 483
    :cond_2
    new-instance v0, Landroid/content/Intent;
    const-string v1, "android.intent.action.VIEW"
    invoke-direct {v0, v1}, Landroid/content/Intent;-><init>(Ljava/lang/String;)V
    .line 484
    const/high16 v1, 0x10000000
    invoke-virtual {v0, v1}, Landroid/content/Intent;->addFlags(I)Landroid/content/Intent;
    .line 485
以下省略......
... (org/zywx/wbpalmstar/engine/EUtil.smali)
android.intent.action.VIEW 、 android.intent.action.VIEW 、 android.intent.action.VIEW 、 android.intent.action.VIEW...
...    .catch Ljava/io/IOException; {:try_start_1 .. :try_end_1} :catch_0
    .line 642
    :goto_2
    new-instance v1, Landroid/content/Intent;
    const-string v2, "android.intent.action.VIEW"
    invoke-direct {v1, v2}, Landroid/content/Intent;-><init>(Ljava/lang/String;)V
    .line 643
    const/high16 v2, 0x10000000
    invoke-virtual {v1, v2}, Landroid/content/Intent;->addFlags(I)Landroid/content/Intent;
    .line 644
以下省略......
... (org/zywx/wbpalmstar/engine/universalex/l.smali)
android.intent.action.VIEW 、 android.intent.action.VIEW 、 android.intent.action.VIEW 、 android.intent.action.VIEW...
...    goto :goto_0
    .line 274
    :sswitch_1
    new-instance v1, Landroid/content/Intent;
    const-string v2, "android.intent.action.VIEW"
    invoke-direct {v1, v2}, Landroid/content/Intent;-><init>(Ljava/lang/String;)V
    .line 275
    const-string v2, "android.intent.category.DEFAULT"
    invoke-virtual {v1, v2}, Landroid/content/Intent;->addCategory(Ljava/lang/String;)Landroid/content/Intent;
    .line 276
以下省略......
... (org/zywx/wbpalmstar/platform/myspace/t.smali)
... 此次省略【89】条数据...
```

- 解决方案

问题同上类似，引擎和公共插件中均做了处理。此中列举的均为第三方插件或者使用了第三方SDK的插件，这些需要插件开发商提供修正版本。因为这个问题需要有特定的有针对性的恶意程序同时存在于手机上运行，才有可能导致app的故障崩溃等，因此影响并不是很大。

4. 检测项目：调试信息检测

- 关键词

Logcat，Log.i

- 详细报告

在开发安卓应用时候，添加调试log信息是一个非常常用的手段，打印log也是一个良好的习惯，有助于程序员快速的定位错误位置和错误信息。但是如果发布出去的应用未及关闭log信息的输出，则有可能泄露应用的逻辑处理和一些账号等信息

```
检测出调试信息函数共【234】条 ：
Landroid/util/Log;->d 、 Landroid/util/Log;->e (com/iflytek/thirdparty/aM.smali)
Landroid/util/Log;->d (org/zywx/wbpalmstar/plugin/uexcamera/CustomCamera$3.smali)
Landroid/util/Log;->e (org/zywx/wbpalmstar/base/ACEParcelFileDescriptorUtil$TransferThread.smali)
Landroid/util/Log;->i (org/zywx/wbpalmstar/plugin/uexweixin/AnalJson.smali)
Landroid/util/Log;->d 、 Landroid/util/Log;->e 、 Landroid/util/Log;->i 、 Landroid/util/Log;->w (org/zywx/wbpalmstar/engine/EUtil.smali)
Ljava/io/PrintStream;->println(Ljava/lang/String;)V (org/zywx/wbpalmstar/plugin/uexweixin/EuexWeChat$GetAccessTokenTask.smali)
Landroid/util/Log;->e (org/zywx/wbpalmstar/engine/universalex/l.smali)
Landroid/util/Log;->d 、 Landroid/util/Log;->i (org/zywx/wbpalmstar/engine/EBrowserWindow.smali)
Landroid/util/Log;->i (org/zywx/wbpalmstar/plugin/uexweixin/EuexWeChat$NetWorkAsyncTaskToken.smali)
Landroid/util/Log;->i (org/zywx/wbpalmstar/plugin/uexqq/EUExQQ$2.smali)
Landroid/util/Log;->d (org/zywx/wbpalmstar/plugin/uexsina/EUExSina$3.smali)
Landroid/util/Log;->d (com/baidu/location/ag.smali)
Landroid/util/Log;->i (org/zywx/wbpalmstar/plugin/uexcontrol/EUExControl.smali)
Ljava/io/PrintStream;->println(Ljava/lang/String;)V (com/ibm/mqtt/MqttHashTable.smali)
Landroid/util/Log;->d (com/baidu/location/ah.smali)
Landroid/util/Log;->i (org/zywx/wbpalmstar/plugin/ueximage/EUExImage$1.smali)
Landroid/util/Log;->d (com/baidu/lbsapi/auth/a.smali)
Landroid/util/Log;->d 、 Landroid/util/Log;->e (com/iflytek/speech/SpeechModuleAidl$1.smali)
Landroid/util/Log;->d (com/amap/api/mapcore/bh.smali)
Ljava/io/PrintStream;->println(Ljava/lang/String;)V (com/ibm/mqtt/Mqtt.smali)
Landroid/util/Log;->e 、 Landroid/util/Log;->i 、 Landroid/util/Log;->w (com/iflytek/thirdparty/b.smali)
... 此次省略【214】条数据...
```

- 解决方案

应当避免直接使用Log进行输出，如果必须使用，则一定防止敏感信息打印在日志中，防止破解方拿到重要信息。

AppCan个别插件可能存在Log，但其中不存在敏感信息，不会泄露。

5. 检测项目：SDCARD数据储存检测

- 关键词

SDCARD

- 详细报告

使用外部存储实现数据持久化，这里的外部存储一般就是指的是sdcard。使用sdcard存储的数据，不限制只有本应用访问，任何可以有访问Sdcard权限的应用均可以访问，容易导致信息泄漏安全风险

```
总共检测资源文件中包含敏感数据访问【4】条 ： 

    const/4 v5, 0x1
    :try_start_1    invoke-virtual {p0, v4, v5}, Landroid/content/Context;->openFileOutput(Ljava/lang/String;I)Ljava/io/FileOutputStream;
com/iflytek/thirdparty/b.smali

    const/4 v2, 0x1    invoke-virtual {v0, v1, v2}, Landroid/content/Context;->getSharedPreferences(Ljava/lang/String;I)Landroid/content/SharedPreferences;
org/zywx/wbpalmstar/plugin/uexdevice/EUExDevice.smali

    const/4 v0, 0x1
    :try_start_0    invoke-virtual {p0, p1, v0}, Landroid/content/Context;->getSharedPreferences(Ljava/lang/String;I)Landroid/content/SharedPreferences;
com/iflytek/thirdparty/S.smali

    const/4 v0, 0x1
    :try_start_0    invoke-virtual {p0, p1, v0}, Landroid/content/Context;->getSharedPreferences(Ljava/lang/String;I)Landroid/content/SharedPreferences;
com/iflytek/thirdparty/S.smali

```

- 解决方案

高德、腾讯微博、科大讯飞等插件都存在SD卡持久化数据，是否存在敏感信息未知。

6. 检测项目：敏感数据访问控制检测

- 关键词

SharePreference

- 详细报告

为了实现不同软件之间的数据共享，设置内部文件为全局可读或全局可写，导致其他应用可以读取和修改该文件。如果此类文件包含了关键配置信息，账户信息数据等敏感信息，可能会被盗取或者恶意篡改，导致如程序无法运行，业务逻辑被修改等问题。

```
总共检测资源文件中包含敏感数据访问【4】条 ： 

    const/4 v5, 0x1
    :try_start_1    invoke-virtual {p0, v4, v5}, Landroid/content/Context;->openFileOutput(Ljava/lang/String;I)Ljava/io/FileOutputStream;
com/iflytek/thirdparty/b.smali

    const/4 v2, 0x1    invoke-virtual {v0, v1, v2}, Landroid/content/Context;->getSharedPreferences(Ljava/lang/String;I)Landroid/content/SharedPreferences;
org/zywx/wbpalmstar/plugin/uexdevice/EUExDevice.smali

    const/4 v0, 0x1
    :try_start_0    invoke-virtual {p0, p1, v0}, Landroid/content/Context;->getSharedPreferences(Ljava/lang/String;I)Landroid/content/SharedPreferences;
com/iflytek/thirdparty/S.smali

    const/4 v0, 0x1
    :try_start_0    invoke-virtual {p0, p1, v0}, Landroid/content/Context;->getSharedPreferences(Ljava/lang/String;I)Landroid/content/SharedPreferences;
com/iflytek/thirdparty/S.smali

```

- 解决方案

根据需要严格控制文件的全局读写权限；对于必须使用的全局可读写文件，严格审核其中是否包含敏感信息。

uexDevice和科大讯飞插件可能存在部分数据为全局可读写，不过uexDevice的信息为非敏感信息，无关紧要。

7. 检测项目：敏感数据显示（输出）与输入检测

- 关键词

安全键盘

- 详细报告

客户端的敏感界面，如登录界面、注册界面、支付界面等，用户再输入敏感信息与显示（输出）如果未使用安全键盘使用第三方未知键盘或系统键盘的话可能存在被数据拦截与输入监听的风险导致敏感数据泄露

- 解决方案

这个就纯属奇葩了，明摆着让你掏钱买他们的安全键盘组件。
如果app内的安全标准没有那么高的话，就不需要安全键盘。如果真的涉及到支付等环节，也可以采用第三方的安全键盘服务，AppCan也有对应的安全键盘插件可以使用。

8. 检测项目：H5文件明文存储检测

- 关键词

html，js，hybrid

- 详细报告

如果存在明文存储的H5资源文件，则会泄露页面基本布局和一些重要的信息

- 解决方案

此问题如果app内业务逻辑不涉及安全级别很高，则无需关心。

如果在意的话，AppCan开发的应用可以通过config.xml文件中配置obfuscation为true，或者打包最后直接选择代码加密即可。同时，其中的debug值应调整为false。

9. 检测项目：资源文件_查看分析检测

- 关键词

res，assets

- 详细报告

程序在未进行资源隐藏或者加密时会出现直接把APK解压缩造成APK 的资源被窃取、查看分析的风险。

- 解决方案

此问题如果app内的资源不存在敏感资源，只是一些普通的布局的话，其实并没有什么问题。

如果在意的话，只能通过第三方加密加固方式进行加固。

10. 检测项目：WebView远程代码执行漏洞

- 关键词

addJavascriptInterface

- 详细报告

Android API level 16以及之前的版本存在远程代码执行安全漏洞，该漏洞源于程序没有正确限制使用WebView.addJavascriptInterface方法，远程攻击者 可通过使用Java Reflection API利用该漏洞执行任意Java对象的方法，简单的说就是通过addJavascriptInterface给WebView加入一个 JavaScript桥接接口，JavaScript通过调用这个接口可以直接操作本地的JAVA接口

```
总共检测资源文件中包含webView【9】条

检测代码中包含webView路径：
检测到包含webview路径
[/smali/com/tencent/open/PKDialog.smali]
检测到包含webview路径
[/smali/com/tencent/open/yyb/AppbarActivity.smali]
检测到包含webview路径
[/smali/com/iflytek/sunflower/CollectorJs.smali]
检测到包含webview路径
[/smali/com/tencent/connect/auth/AuthDialog.smali]
检测到包含webview路径
[/smali/com/umeng/analytics/MobclickAgentJSInterface.smali]
...
此处省略【4】条数据
...

```

- 解决方案

AppCan引擎在3.2版本之后已经修改了webview中JS与原生交互的方式，而不依赖webview自带的addJavascriptInterface接口了，故不存在这个漏洞。

上述列举的问题，集中在uexUmeng，uexTent，以及科大讯飞插件中，属于第三方SDK的问题，需要第三方开发商配合解决。

但如果这些代码涉及到的业务不会存在读取远程页面的逻辑，则这个问题根本就不会出现。这里只是说有这个隐患的可能而已。
