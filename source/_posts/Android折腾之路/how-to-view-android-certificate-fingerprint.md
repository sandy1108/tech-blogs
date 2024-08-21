---
title: 查看Android证书指纹信息
categories: 
  - Android折腾之路
excerpt: 我们在高德、百度等这些第三方SDK的集成和注册时，都会让我们提供包名或者证书指纹（MD5、SHA-1、SHA-256）等信息，这些信息如何获取呢？有多种方法。
date: 2019-07-09 11:44:55
tags: 
---

## 用途

我们在高德、百度等这些第三方SDK的集成和注册时，都会让我们提供包名或者证书指纹（MD5、SHA-1、SHA-256）等信息，这些信息如何获取呢？有多种方法。

## 准备工作

安装好JDK。最好配置好环境变量（为啥要配置环境变量我就不解释了，反正大佬们都懂，小白们起码也知道怎么配，都是基本功了），这样用起来更方便。

我们主要用的是jdk的bin目录中的keytool工具。

## 动手

### 情况1：证书在手

> 这种情况下，我就是开发者，我有keystore或者jks文件。可以使用命令直接查看证书信息。

```
keytool -list -v -keystore [keystore或jks文件路径] -storepass [密钥库密码]
```

```-storepass``` 也可以不写，回车后会提示输入密码。

关于这个命令用法的详细介绍，我们可以看一下命令本身提供的帮助输出：

```
keytool -list [OPTION]...

列出密钥库中的条目

选项:

 -rfc                            以 RFC 样式输出
 -alias <alias>                  要处理的条目的别名
 -keystore <keystore>            密钥库名称
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令

使用 "keytool -help" 获取所有可用命令
```

### 情况2：手中只有用目标证书打包出来的apk包

> 有的时候，是给别人帮忙，或者仅仅是查看一下别人家apk的信息，这时候，我们就需要稍微绕点弯路。我们知道，apk打包之后，包内会保存有RSA文件，这相当于是证书的公钥，用来给系统验证apk包签名有效性的（非对称加密的知识如果还不太了解，大家可以去查一下，这里暂时不再赘述）。而一般第三方厂商校验的，也就是这个公钥证书的指纹。

1. 解压缩apk文件（初级开发者们不要吃惊，apk包其实就是一个zip）；

2. 找到RSA文件：apk包/META-INF/xxx.RSA。（这个xxx一般是签名证书的别名，不重要）；

3. 使用命令读取RSA的信息：

```
keytool -printcert -file [RSA文件路径]
```

4. 关于此命令的用法介绍：

```
keytool -printcert [OPTION]...

打印证书内容

选项:

 -rfc                        以 RFC 样式输出
 -file <filename>            输入文件名
 -sslserver <server[:port]>  SSL 服务器主机和端口
 -jarfile <filename>         已签名的 jar 文件
 -v                          详细输出

使用 "keytool -help" 获取所有可用命令
```

## 结束

拿到了输出信息之后，想必大家已经知道要用什么了。有的时候可能还要考虑大小写，还有去除冒号之类的，这个具体要看人家第三方需要什么格式了，睁大眼睛看好了。

