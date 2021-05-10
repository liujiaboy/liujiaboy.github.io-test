---
title: Hash
date: 2021-05-10 17:38:40
tags:
    - ARM汇编
categories:
    - 逆向
---


# base64 补充

> Base64就是一种基于64个字符来表示二进制数据的方法。没6个比特为一个单元，


具体可以查看[base64的解释](https://zh.wikipedia.org/wiki/Base64)

64个字符包括 `A-Z a-z 0-9 + /`，再加上`=`用来补位，加上【等号】就是65个。
64个字符分别对应 `0 - 63` 这64个数字，64个数字对应着4个6位二进制数。

![](base64.jpg)


下方代码是在iOS中的一种编码、解码方式：

```
//编码
- (NSString *)base64Encode:(NSString *)string{
    NSData *data = [string dataUsingEncoding:NSUTF8StringEncoding];
    return [data base64EncodedStringWithOptions: 0];
}
//解码
- (NSString *)base64Decode:(NSString *)string{
    NSData *data = [[NSData alloc] initWithBase64EncodedString:string options:0];
    return [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
}
```

1. base64只适用于表示二进制文件。
2. base64编码后，文件数量变多，不使用与大型数据。
3. base64和数据一一对应，不安全。


# Hash

Hash，一般翻译做“散列”，也有直接音译为“哈希”的，就是把任意长度的输入通过散列算法变换成固定长度的输出，该输出就是散列值。

这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，所以不可能从散列值来确定唯一的输入值。简单的说就是一种将任意长度的消息压缩到某一固定长度的消息摘要的函数。

## Hash特点

* 算法公开
* 对相同数据运算，得到的结果是一样的
* 对不同数据运算，得到的结果是定长的，如MD5得到的结果默认是128位,32个字符（16进制标识）。
* 无法逆运算
* 信息摘要，信息“指纹”，是用来做数据识别的

## 用途

* 用户密码加密
* 搜索引擎
* 版权
* 数字签名


## 常见的散列算法

常见的就是MD5，SHA-1，HSA-256，HSA-512等等~




# 密码加密逻辑：

1.







