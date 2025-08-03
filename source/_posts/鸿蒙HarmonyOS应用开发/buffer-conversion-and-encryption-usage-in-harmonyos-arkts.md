---
title: 鸿蒙ArkTS中各种Buffer转换与加密的使用
categories: 
  - 鸿蒙HarmonyOS启程之路
excerpt: ArrayBuffer，Uint8Array，MD5，SHA-1，SHA-256，base64等等
date: 2025-06-17 17:18:19
tags: 
---

## 导入Buffer模块

```ts
import { buffer } from '@kit.ArkTS';
```

## buffer对象的作用

Buffer对象用于表示固定长度的字节序列，是专门存放二进制数据的缓存区。

推荐使用场景： 可用于处理大量二进制数据，图片处理、文件接收上传等。

平时常用的场景比如：加解密过程中的字节数组处理，保存，转换utf-8字符串，16进制字符串，base64字符串等等。

导入的buffer模块是一个包装体，其中还有个buffer属性存放了ArrayBuffer类型的实际数据。（最开始用的时候我就纳闷，为啥是buffer.from了之后又要.buffer，套娃么，很奇怪的表达）。

## buffer对象使用场景示例记录

### MD5加密中使用

```ts
  public static getMD5Code(inString: string): string {
    // 使用了cryptoFramework自带API
    const md5 = cryptoFramework.createMd("MD5");
    // utf-8字符串转为Uint8Array
    md5.updateSync({ data: new Uint8Array(buffer.from(inString, 'utf-8').buffer) });
    const result = md5.digestSync();
    // result.data转为16进制字符串
    const resultStr = buffer.from(result.data.buffer).toString('hex');
    return resultStr;
  }
```

### 16进制字符串转为Uint8Array

```ts
export class HexConverter {

    /**
     * 将十六进制字符串转换为字节数组
     * @param hexString 十六进制字符串
     * @returns 转换后的字节数组
     */
    public static hexStringToBinary(hexString: string): Uint8Array {
        // 转为buffer再转为Uint8Array
        return new Uint8Array(buffer.from(hexString, "hex").buffer);
    }

    /**
     * 将字节数组转换为十六进制字符串
     * @param bytes 字节数组
     * @returns 转换后的十六进制字符串
     */
    public static binaryToHexString(bytes: Uint8Array): string {
        // 转为buffer
        const bytesBuffer: ArrayBuffer = bytes.buffer as object as ArrayBuffer;
        // 转为16进制字符串
        const hexString = buffer.from(bytesBuffer).toString("hex");
        return hexString;
    }
}
```

### RC4加密中使用

**注意**

RC4算法已经不安全了，没有必要使用了，可以使用更好的算法，比如AES加密，或者SHA256加密等等。但是由于某些业务逻辑有兼容需求，故本人在开发过程中实现了一下。

```
import { HexConverter } from "./HexConverter";
import { buffer } from "@kit.ArkTS";

export class RC4Encryption {
    private static readonly F_KEY: string = "这里是密钥";

    private static readonly sk: number[] = [
        0x89, 0x72, 0xaa, 0x4c, 0x0c, 0x56, 0xcf,
        ...省略，这里硬编码了S-box，降低了安全性，只为了示例代码...
        0x86, 0x5d, 0xf3, 0x5e, 0x65, 0x77, 0x48, 0x51, 0xfb, 0xf0, 0x3c,
        0xe6, 0xc7, 0x24, 0x7d, 0x44, 0x05, 0x7f
    ];

    private static swap(S: number[], i: number, j: number): void {
        const temp: number = S[i];
        S[i] = S[j];
        S[j] = temp;
    }

    private static re_S(S: number[]): void {
        for (let i: number = 0; i < RC4Encryption.sk.length; i++) {
            S[i] = RC4Encryption.sk[i];
        }
    }

    private static re_T(T: number[], key: string): void {
        const keylen: number = key.length;
        const keys: number[] = [];
        for (let i: number = 0; i < key.length; i++) {
            keys.push(key.charCodeAt(i) & 0xFF);
        }
        for (let i: number = 0; i < 256; i++) {
            const k: number = i % keylen;
            T[i] = keys[k];
        }
    }

    private static re_Sbox(S: number[], T: number[]): void {
        let j: number = 0;
        for (let i: number = 0; i < 256; i++) {
            j = (j + S[i] + T[i]) % 256;
            RC4Encryption.swap(S, i, j);
        }
    }

    private static re_RC4(S: number[], key: string): void {
        const T: number[] = new Array(256).fill(0);
        RC4Encryption.re_S(S);
        RC4Encryption.re_T(T, key);
        RC4Encryption.re_Sbox(S, T);
    }

    private static RC4(src: number[], n: number, dest: number[], key: string): number {
        const S: number[] = new Array(256).fill(0);
        RC4Encryption.re_RC4(S, key);

        let i: number = 0;
        let j: number = 0;
        for (let nIndex: number = 0; nIndex < n; nIndex++) {
            i = (i + 1) % 256;
            j = (j + S[i]) % 256;
            RC4Encryption.swap(S, i, j);
            const t: number = (S[i] + (S[j] % 256)) % 256;
            dest[nIndex] = src[nIndex] ^ S[t];
        }
        return n;
    }

    public static os_decrypt(pBuffer: Uint8Array, n: number, pKey: string = RC4Encryption.F_KEY): Uint8Array {
        const newInt: number[] = new Array(n);
        const resInt: number[] = new Array(n);

        for (let i: number = 0; i < n; i++) {
            newInt[i] = pBuffer[i];
        }

        RC4Encryption.RC4(newInt, n, resInt, pKey);

        const newData: Uint8Array = new Uint8Array(n);
        for (let k: number = 0; k < n; k++) {
            newData[k] = resInt[k];
        }

        return newData;
    }

    /**
     * 加密字符串并返回16进制结果
     * @param plainText 明文字符串
     * @param pKey 加密密钥，默认为F_KEY
     * @returns 加密后的16进制字符串
     */
    public static encryptString(plainText: string, pKey: string = RC4Encryption.F_KEY): string {
        // 字符串转字节数组
        const bytes: Uint8Array = HexConverter.hexStringToBinary(plainText);
        // 加密字节数组
        const encrypted: Uint8Array = RC4Encryption.os_decrypt(bytes, bytes.length, pKey);
        // 转换为16进制字符串
        return HexConverter.binaryToHexString(encrypted);
    }

    /**
     * 解密16进制字符串并返回原始字符串
     * @param hexString 16进制格式的加密字符串
     * @param pKey 解密密钥，默认为F_KEY
     * @returns 解密后的原始字符串
     */
    public static decryptString(hexString: string, pKey: string = RC4Encryption.F_KEY): string {
        // 16进制字符串转字节数组
        const bytes: Uint8Array = HexConverter.hexStringToBinary(hexString);
        // 解密字节数组
        const decrypted: Uint8Array = RC4Encryption.os_decrypt(bytes, bytes.length, pKey);
        // 字节数组转字符串
        return buffer.from(decrypted).toString('utf-8');
    }
}

```

## 总结 

1. 使用buffer对象时，需要先导入buffer模块，然后使用buffer对象的from方法将字符串转换为buffer对象。
2. 获取ArrayBuffer对象，可以使用buffer对象的buffer属性。
3. 使用buffer对象的buffer.from(bytesBuffer).toString("hex")方法将buffer对象转为16进制字符串。
4. 使用buffer对象的buffer.from(bytesBuffer).toString("utf-8")方法将buffer对象转为utf-8字符串。

## buffer API文档

https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/js-apis-buffer-V5