---
title: 移动端利用SQLCipher加密数据库
categories: 
  - iOS摸索之路
excerpt: SQLite目前已经是比较流行的数据存储操作的API了。Android和iOS系统提供的API中操作数据库默认均采用了SQLite方案。而SQLCipher是基于SQLite的加密数据库存取方案，集成相对比较快捷而且透明，在一定程度上保证了数据的安全。
date: 2018-01-22 17:55:55
tags: 
---

# 简介

SQLite目前已经是比较流行的数据存储操作的API了。Android和iOS系统提供的API中操作数据库默认均采用了SQLite方案。而SQLCipher是基于SQLite的加密数据库存取方案，集成相对比较快捷而且透明，在一定程度上保证了数据的安全。

# 集成文档说明

- iOS：https://www.zetetic.net/sqlcipher/ios-tutorial/

- Android：https://www.zetetic.net/sqlcipher/sqlcipher-for-android/

# 集成方法

## iOS

### 工程配置

1. 下载sqlcipher针对iOS的版本的源码（这是一个静态库工程：https://github.com/sqlcipher/sqlcipher.git）；

2. 工程BuildSettings中Other C Flags 增加（App工程也需要此配置，代码有许多地方进行了这个宏的判断，没有此宏，则加密逻辑不生效）：```-DSQLITE_HAS_CODEC```

3. 编译sqlcipher工程生成libsqlcipher.a；

4. 将libsqlcipher.a静态库Link到你的app的主工程中，并引入SQLCipher版的sqlite3.h头文件；

5. 增加link系统库Security.framework；

6. 移除系统的sqlite库libsqlite3.tbd（如果你用了的话）；

7. 备注：第3和4步，编译libsqlcipher.a也可以不单独进行，可以以工程依赖的方式引入主工程，方法就是，直接把工程拖到主工程里，配置主工程添加Target，以及Link。

### 代码配置

1. 如果你用的是系统sqlite3的API，则代码基本不用变，因为SQLCipher是基于系统的sqlite3 API扩展的；

2. 如果你用的是FMDB，更无需担心，因为FMDB是封装的系统sqlite3 API；

3. 唯一需要做的，就是初始化打开数据库文件sqlite3_open之后，紧跟着执行sqlite3_key设置数据库加密的密钥，于是sqlcipher集成工作就完成了。如果你用的是FMDB，可以直接用setKey方法；如果不是，也可以去FMDataBase.m源码中拷贝这个方法的逻辑，方便调用，关键代码如下：

```
- (BOOL)setKey:(NSString*)key {
    NSData *keyData = [NSData dataWithBytes:[key UTF8String] length:(NSUInteger)strlen([key UTF8String])];
    
    return [self setKeyWithData:keyData];
}

- (BOOL)setKeyWithData:(NSData *)keyData {
#ifdef SQLITE_HAS_CODEC
    if (!keyData) {
        return NO;
    }
    
    int rc = sqlite3_key(dbHandle, [keyData bytes], (int)[keyData length]);
    BOOL ret = (rc == SQLITE_OK);
    if (!ret) {
        NSLog(@"setKeyWithData--->status: %d", rc);
    }
    return ret;
#else
    return NO;
#endif
}
```

需注意这里面也有判断SQLITE_HAS_CODEC这个宏的逻辑，如果不需要切换不加密的编译方式，此处也可以考虑去掉宏判断逻辑，直接用内部的逻辑即可。

另外，这里的key最好判断一下，如果为空字符串，就不要执行了，否则会崩溃。

### 注意事项

1. 不设置key，会使用默认sqlite3数据库；
2. 设置了key之后，SQLCipher不会自动加密旧数据。换句话说，如果数据库不是全新安装，设置key之后还想去打开没有加密的数据库，会出错；
3. sqlcipher自带一些sql命令，用来加解密数据库、更改密码以及转移数据等；
4. 必须移除系统默认sqlite3.tbd库，否则，sqlcipher初始化会失败报错；
5. Other C Flags增加-DSQLITE_HAS_CODEC，含义大概是：编译C代码之前，先注入一个名叫SQLITE_HAS_CODEC的全局宏变量。没有这个宏的话，sqlcipher中的加密逻辑不会被编译（具体内容可以搜索源码，有相关逻辑）；

## Android

### 工程配置

1. build.gradle文件中，加入：

```
compile 'net.zetetic:android-database-sqlcipher:3.5.7@aar'
```

2. 没了。

### 代码配置

1. 找到你的代码中所有引用系统sqliteAPI的地方，类似：

```
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
```

替换为下面的：

```
import net.sqlcipher.database.SQLiteDatabase;
import net.sqlcipher.database.SQLiteOpenHelper;
```

2. 换了新的依赖和import之后，新的SQLiteOpenHelper中getWritableDatabase方法增加了一个参数，即为密码。

```
mDatabaseHelper.getWritableDatabase(dbKey)
```

3. 集成完毕。

### 注意事项

1. 读写加密数据库时，open和close操作会慢不少，需要针对这个问题，使用时进行优化（比如频繁操作数据库的一段时间里，不要频繁开启关闭数据库）；
