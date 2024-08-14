---
title: learning-linux-through-ss-deploy-2017-12-06
categories:
  - 瞎折腾系列
excerpt: 本文记录了SS部署的实践过程和一些体会，仅供学习讨论使用，**坚决抵制各种使用此类技术进行售卖、推广、传播科学上网渠道的违法行为。**
date: 2017-12-06 14:27:36
tags:
---

# 写在前面

本文记录了SS部署的实践过程和一些体会，仅供学习讨论使用，**坚决反对各种使用此类技术进行售卖、推广、传播科学上网渠道的行为。**

记录比较仓促，可能有些问题，请指正。

# 申请海外VPS

个人使用过两个VPS提供商：Vultr和GoogleCloudPlatform。下面列举一下各自的特点吧（不全面，个人感受）。

- Vultr

1. 提供免费的DNS解析服务；
2. 支持支付宝付款；
3. 选择东京的VPS可能会比较快一点。
4. 最低5美元一个月（号称有2.5美元的，但是常年缺货，呵呵~），1000G流量。

- GoogleCloudPlatform

1. 需要用Visa或者MasterCard（有小伙伴说，Visa卡支付失败。我的MasterCard是可以的，原因未知）；
2. 选择asia-east-c的VPS速度在国内直连应该是相对比较快的。不过最近我这边的网络貌似有些抽风了，之前都是延迟100ms以下，最近变成200ms+了，但是去测ping的网站测了，平均都是100ms以下。难道是帝都的网络有限制。。。
3. 价格是按照配置细分的，非常细，内存大小、CPU、存储空间等，平均下来，同等配置的GCE还是比Vultr贵一些的，而且流量还要单独收费。
4. 目前可以申请免费试用1年，送300美元体验金，爽翻，让我直接把vultr主机给暂时删除了。

# 域名申请

1. 一开始去了百度，头脑发热搞了个域名，结果发现好像必须要备案才行，还得有他的VPS。好麻烦，受不了。
2. 我在GoDaddy上申请的。可以申请各种后缀，当然价格也是不同的。可以支持支付宝。

# 最后准备

## 服务端系统选择

初始化VPS，可以选择安装标准的各种系统。我从大学时代的Ubuntu9.04入门Linux的（仅仅是瞎鼓捣而已），所以我还是倾向于用Ubuntu。选择当下稳定版本Ubuntu16.04LTS。

## SSH管理

1. 通过ssh连接可以远程管理服务端了。

```
//不输入用户名和@的话，会直接使用当前终端的用户。
ssh [用户名]@[主机IP或域名]
```

2. Windows中SSH客户端可以用PuTTY或者XShell。XShell注意不要下载带后门的版本，注意辨别。[相关了解传送门](https://oschina.net/news/87722/xshell-has-security-vulnerability)。文件传输可以用可以用WinSCP。

3. MacOS系统和其他Linux系统中SSH客户端我直接用系统自带的终端，支持scp。

## 基础服务

Shadowsocks基础服务现在已经有很多大牛开发使用各种不同的语言写了各种不同的版本，都收纳到了shadowsocks组中了。**总之，多选1即可，不用都装。**

# 安装服务端基础服务(of libev，基础服务n选1)

这是基础服务，有了这个。。。基本就够了。libev的比较底层，使用C写的，照顾到很多低端的设备或者嵌入式设备等，移植性较强。

## Github地址

https://github.com/shadowsocks/shadowsocks-libev

## 包管理直接安装（Ubuntu16.04LTS）

```
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:max-c-lv/shadowsocks-libev
sudo apt-get install rng-tools
sudo apt-get install haveged
sudo apt-get update
sudo apt install shadowsocks-libev
```

## 源码编译安装（未尝试，仅记录）

1. 下载源码：

```
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init --recursive
```

2. 编译deb包：

```
mkdir -p ~/build-area/
cp ./scripts/build_deb.sh ~/build-area/
cd ~/build-area
./build_deb.sh
```

# 安装服务端基础服务(via PIP，基础服务n选1)

这是python版本的服务端，pip是python版本的包管理工具。python版SS实现功能一样的，Github地址是：https://github.com/shadowsocks/shadowsocks ，这大概也是最早开发的一个版本了吧，一直也在维护。不过进入之后发现默认分支是rm，说是按照相关法规已经移除了代码。嗯，其实吧。。。开始记录步骤。

1. 基本步骤(Ubuntu或Debian)

```

apt-get install python-pip
pip install shadowsocks

```

2. 如果出现错误：

```

Collecting shadowsocks
  Downloading shadowsocks-2.8.2.tar.gz
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
    ImportError: No module named setuptools
    
    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-kOyFxM/shadowsocks/
You are using pip version 8.1.1, however version 9.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.


```

则可以执行下列命令进行修复，升级pip，然后再安装ShadowSocks：

```

pip install --upgrade pip
pip install -U setuptools

```

# 基础服务config配置文件

- 我们将配置文件放在用户目录下的ss目录中，故执行：

```
vim ~/ss/ssserver.json
```

- libev版的json配置文件内容示例：

```
{
    "server":"0.0.0.0",
    "server_port":8011, 
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"123456",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open":true
}
```

- python版的json配置文件内容示例（多端口和密码配置）：

```
{
    "server":"0.0.0.0",
    "port_password": {
        "8011": "123456",
        "8012": "123456"
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": true
}
```

# 运行服务

1. python版：

```
//一次性运行
ssserver -c ~/ss/ssserver.json

//长期运行
nohup ssserver -c ~/ss/ssserver.json -d start &

```

2. libev版：

加了-u是为了支持UDP转发。

```
ss-server -c ~/ss/ssserver.json -u

nohup ss-server -c ~/ss/ssserver.json -u &
```

# 停止服务

1. python版：

```

ssserver -c ~/ss/ssserver.json -d stop

```

2. libev版：

```
netstat -anp | grep ss-server
//找到对应的pid
kill [pid]
```

# 查看运行状态（libev）

```
netstat -anp | grep ss-server
```

# SS客户端们

1. 官网的各种开源客户端

https://github.com/shadowsocks

2. SSTap

支持Windows中代理部分游戏的UDP连接，实现游戏加速。

# 安装管理服务(非必需)(via NPM)

这是node版本的服务管理端，npm是nodejs版本的包管理工具。**注意，这不是一个完整的SS服务端，他是用来管理SS的基础服务的。如果是个人使用，就别折腾这玩意了。**

Github地址是：https://github.com/shadowsocks/shadowsocks-manager。

1. 安装6.x版本的NodeJS：

```
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

2. 安装shadowsocks服务：

```
npm i -g shadowsocks-manager

//use ssmgr to run this program.
```

# SS-manager配置文件

- yml配置文件示例：

```
type: s
empty: false

shadowsocks:
    address: 127.0.0.1:4000

manager:
    address: 0.0.0.0:4001
    password: '123456'

db: 'ss.sqlite'
```

- webgui.yml配置文件示例

```
type: m
empty: false

manager:
    address: ss.abc.pro:4001
    password: '123456'

plugins:
    flowSaver:
        use: true
    user:
        use: true
    account:
        use: true
        pay:
            hour:
                price: 0.03
                flow: 500000000
            day:
                price: 0.5
                flow: 7000000000
            week:
                price: 3
                flow: 50000000000
            month:
                price: 10
                flow: 200000000000
            season:
                price: 30
                flow: 200000000000
            year:
                price: 120
                flow: 200000000000
    email:
        use: true
        username: 'username'
        password: 'password'
        host: 'smtp.gmail.com'
    webgui:
        use: true
        host: '0.0.0.0'
        port: '80'
        site: 'ss.abc.pro'
        gcmSenderId: '456102641793'
        gcmAPIKey: 'AAAAGzzdqrE:XXXXXXXXXXXXXX'
    alipay:
        use: true
        appid: <支付宝 APPID>
        notifyUrl: ''
        merchantPrivateKey: '<rsa_private_key.pem 中的私钥>'
        alipayPublicKey: '<支付宝公钥>'
        gatewayUrl: 'https://openapi.alipay.com/gateway.do'

db: 'webgui.sqlite'
```
