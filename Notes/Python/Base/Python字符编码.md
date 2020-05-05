[TOC]

# 字符编码 

在计算机数据中,统一表示为二进制数据,即只能用0和1来表示,8个二进制位任意组合,可以有2^8=256种方式,一个字节最

低也需要8个二进制位来表示,所以一共可以表示出256个不同的字节,一个Byte代表一个字符;

最小表示单位为 bit（位）

最小存储单位为 Byte(字节)

8bits = 1Byte = 1字节

1024byte = 1KBytes

## 字符编码的种类：

### ASCII

(American Standard Code for Information)美国标准信息交换代码

是基于拉丁字母的一套单字节编码系统,主要用于显示现代英语和其他西欧语言;

其最多只能用8位二进制来表示1个字节;

所以ASCII码最多只能表示2**8-1=255个字符(字节)

![img](Python%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81.assets/8e05fbda-bf96-4208-9980-e2fa3b7c2385.png)

### Unicode 万国码,统一码

为了解决传统的字符编码方案的局限而产生的,为每种语言中的每个字符设定了统一并且唯一的二进制编码,规定字符和符号最

少由16位来表示,也就是最少2个字节的长度,可表示字符总数 2**16 = 65536

### UTF-8

是对Unicode编码的压缩和优化,不再使用最少2个字节的规则,而是将所有的字符和符号进行分类,ASCII码使用1个

字节保存,欧洲字符使用2个字符保存,东亚字符使用3个字节保存等,即先判断字符属于什么类型,然后选择最适合的长度去存储;

### GBK

中文编码,包含简体和繁体中文字符的表示,2个字节长度,最多可表示字符总数: 2**16 = 65536

### 优缺点

Unicode:简单粗暴,所有字符都是2Bytes

优点:字符 -> 数字的速度块;

缺点:占用空间大,内存中使用的是uniconde,用空间换时间;

Utf-8:精准,对不同的字符用不同的长度表示

优点:节省空间;

缺点:是转换速度较慢,因为每次都要计算字符需要多长的Bytes来表示,硬盘中或网络传输用utf-8.

## 字符格式的转换

文本编辑器：要保证存取编码格式一致

![img](Python%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81.assets/22dbe5ec-62ae-485e-8ef4-702c6555d2fc.png)

说明:

UTF-8转换成Unicode

UTF-8 --> .decode() --> Unicode

GBK转换成Unicode

GBK -> .decode() -> Unicode

GBK转换成UTF-8

GBK -> .decode() -> Unicode -> .encode() -> UTF-8

UTF-8转换成GBK

UTF-8 -> .decode() -> Unicode -> .encode() -> GBK

注意:

Python 2.x 默认使用ASCII字符编码格式

Python 3.x 默认Unicode编码格式.

在python3执行的过程中,所有的字符串都会被识别成unicode编码的结果.

我们在写python代码文件的时候,一般先指定默认字符编码:

 

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#
file = '文本'
file.decode(encoding="utf-8", errors="strict")
file.encode(encoding="gbk", errors="strict")

# encoding为要编码的格式，errors是指的是错误的处理方案
```

