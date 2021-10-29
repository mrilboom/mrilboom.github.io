---
tags:
  - 
comments: true
hidden: false
date: 2021-10-19 11:08:59
title: base 64 编码详解
---
***
从事计算机这一领域的人员或多或少的会听说过 base64 这个编码方式，可能更多地听到的是 base64 加密解密，这两天刚好接触到了 base64 ，因此我又重温了一遍 base64 的内容。<!-- more -->

***
## 介绍

​	首先，base64 并不是一种加密方式，虽然它对于人类来说不可读，但是它的编码风格很有特色：

```
SXNhYWMgYW5kIGhpcyBtb3RoZXIgbGl2ZWQgYWxvbmUgaW4gYSBzbWFsbCBob3VzZSBvbiBhIGhpbGwuIElzYWFjIGtlcHQgdG8gaGltc2VsZiBkcmF3aW5nIHBpY3R1cmVzIGFuZCBwbGF5aW5nIHdpdGggaGlzIHRveXMgYXMgaGlzIG1vbSB3YXRjaGVkIENocmlzdGlhbiBicm9hZGNhc3RzIG9uIHRoZSB0ZWxldmlzaW9uLiBMaWZlIHdhcyBzaW1wbGUgYW5kIHRoZXkgd2VyZSBib3RoIGhhcHB5LiBUaGF0IHdhcyB1bnRpbCB0aGUgZGF5IElzYWFjJ3MgbW9tIGhlYXJkIGEgdm9pY2UgZnJvbSBhYm92ZToKCiJZb3VyIHNvbiBoYXMgYmVjb21lIGNvcnJ1cHRlZCBieSBzaW4uIEhlIG5lZWRzIHRvIGJlIHNhdmVkLiIKIkkgd2lsbCBkbyBteSBiZXN0IHRvIHNhdmUgaGltLCBteSBMb3JkLCIgSXNhYWMncyBtb3RoZXIgcmVwbGllZCwgcnVzaGluZyBpbnRvIElzYWFjJ3Mgcm9vbSwgcmVtb3ZpbmcgYWxsIHRoYXQgd2FzIGV2aWwgZnJvbSBoaXMgbGlmZS4uLg==
```

​	这串编码中只出现了大小写字母而且末尾有`=`,稍微了解过 base64 的人一眼就能看出用了 base64 编码，随便找个在线解码的网站复制一下解码就可以看到原文，因此 base64 对于数据的安全性来说没有任何的保障。

​	既然 base64 没有加密的功能，而且它把好好的文字数据搞成这样乱七八糟，有什么意义呢？
​	base64 主要是为了计算机之间数据传输而设计的（尽管它将原数据编码后长度反而会增加三分之一），两台计算机之间传输数据，通过网线传入传出的肯定会是 0101 这样的二进制数据，因此接收方必须对这些数据进行解码，但是有时候这些数据中包含了一些控制字符，比如`ACK`、`NULL`等，这些字符不可打印，在文本传输的时候接收方可能会错误的解码，导致数据出现问题，因此有人就采用了base64 这一套解决方案，它将所有字符编码成可打印输出的字符，避免因为控制字符的问题而导致的文本乱码。

## 编码方式

### 编码表

​	base64 之所以叫做64 ，是因为它选取了64个可打印字符进行编码，分别是大小写a-z，52个，0-9，10个，再加上`+`、`/`2个，一共64个字符。

| 数值 | 字符 | 数值 | 字符 | 数值 | 字符 | 数值 | 字符 |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|  0   |  A   |  16  |  Q   |  32  |  g   |  48  |  w   |
|  1   |  B   |  17  |  R   |  33  |  h   |  49  |  x   |
|  2   |  C   |  18  |  S   |  34  |  i   |  50  |  y   |
|  3   |  D   |  19  |  T   |  35  |  j   |  51  |  z   |
|  4   |  E   |  20  |  U   |  36  |  k   |  52  |  0   |
|  5   |  F   |  21  |  V   |  37  |  l   |  53  |  1   |
|  6   |  G   |  22  |  W   |  38  |  m   |  54  |  2   |
|  7   |  H   |  23  |  X   |  39  |  n   |  55  |  3   |
|  8   |  I   |  24  |  Y   |  40  |  o   |  56  |  4   |
|  9   |  J   |  25  |  Z   |  41  |  p   |  57  |  5   |
|  10  |  K   |  26  |  a   |  42  |  q   |  58  |  6   |
|  11  |  L   |  27  |  b   |  43  |  r   |  59  |  7   |
|  12  |  M   |  28  |  c   |  44  |  s   |  60  |  8   |
|  13  |  N   |  29  |  d   |  45  |  t   |  61  |  9   |
|  14  |  O   |  30  |  e   |  46  |  u   |  62  |  +   |
|  15  |  P   |  31  |  f   |  47  |  v   |  63  |  /   |

### 编码示例

来看一下它的编码方式：

{% asset_img coding.png %}

<font size=2>[阮一峰 | Base64 笔记](http://www.ruanyifeng.com/blog/2008/06/base64.html)</font>

首先将要编码的数据转换成二进制的形式，之后按照 6 个为一组进行分割，将分割后的二进制转换为十进制，再查表找到对应的字符即可，例如`Man`转换为二进制为

| 0    | 1    | 0    | 0    | 1    | 1    | 0    | 1    | *    | 0    | 1    | 1    | 0    | 0    | 0    | 0    | 1    | *    | 0    | 1    | 1    | 0    | 1    | 1    | 1    | 0    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |

分割之后变为

| 0    | 1    | 0    | 0    | 1    | 1    | *    | 0    | 1    | 0    | 1    | 1    | 0    | *    | 0    | 0    | 0    | 1    | 0    | 1    | *    | 1    | 0    | 1    | 1    | 1    | 0    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |

对应的字符为

| 19   | 22   | 5    | 46   |
| ---- | ---- | ---- | ---- |
| T    | W    | F    | u    |

它其实将原来的 3 个字节变成了 4 个字节，数据长度增加了三分之一。

因为不是编码前后等长对应的，有时候会出现不够划分的情况，比如`M`会被分为`0  1  0  0  1  1  *  0  1`,因此规定长度不够的一律低位补 0，因此M会被编码为`TQ`,但是 base64 还有一个规定，编码之后不够 4 个字节的在末尾补足`=`，使得编码后的数据能够 4 个一组进行划分，方便了之后的解码。因此实际上 base64 编码表有 65 个。这里的`=`也是 base64 编码最大的特色，因此以后如果看到一串乱七八糟的大小写混合的字符以`=`结尾，不妨试一试 base64 解码。


## 参考资料
[阮一峰 | Base64笔记](http://www.ruanyifeng.com/blog/2008/06/base64.html)