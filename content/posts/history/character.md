---
title: "符号、编码"
date: 2021-09-18T23:19:56+08:00
draft: true
toc: true
images:
tags: 
  - 历史
---

记录一下字符集、字符编码和常见的格式。

## ASCII

American Standard Code for Information Interchange，美国信息交换标准代码，规定了128 个字符的编码(也就是对应的二进制表示)

其中 【0000,0000 - 0001，1111】前 32 位是控制字符，【0010,0000 - 0111,1111】是打印字符，共128 个，全世界如果都用英语就足够了。

一些欧洲国家就决定利用闲置的最高位，做了拓展，后面 ISO组织在 ASCII 基础之上又制定了一系列标准 ISO-8859-1 ～ ISO-8859-15。后面就出现了多个标准基于 ASCII 的拓展，虽然可以向下兼容 ASCII ，但是拓展的高位存在差异性，ASCII 就既可以表示字符集，同时也是一种字符编码。

**man 7 ascii 查看 8、10、16进制的ASCII对应的值。**

## GB

字符型语言的字符数量较少，用一个字节(8bit) 就基本够用了，对于中文是完全不够用的 。这个时候出现了国标 **GB2312**，使用了两个字节表示，收入了大部分常见字，共7445个字符，包括6763个汉字和682个其他符号。

后来拓展到了 **GBK1.0**拓展编码，向下兼容GB2312，还收录了Unicode基本多文种平面中的所有CJK（Chinese、Japane、Korea）汉字、汉字部首符号、竖排标点符号等字符。

**GB18030** 取代GBK1.0 称为正式国家标准，收录了所有Unicode3.1中的字符、少数民族自负，并且向下兼容GBK1.0、GB2312。GB系列使用区位码来区分编码，<区，位>。

GBK 和 BGB2312 都是**双字节等宽编码**，因为兼容单字节的ASCII ，也可以认为是单字节和双字节混合的变长编码(Multi-Bytes Character Set，**MBCS**)。

GB18030编码是变长编码，采用了单字节和双字节、4字节方案，其中单字节、双字节和GBK是完全兼容的，4字节编码的码位还收录了CJK拓展的6582个汉字。

## Unicode

Universal Multipie-Octet Coded Character Set ，规定的字符集也被称为 Universal Character Set ,**UCS**。整合全世界的语言文字，是一个**符号集**，只规定了符号的二进制，用多长的字节去表示各种符号文字，没有规定这个二进制代码应该如何存储，**UTF**（Transformation Format）规范规定了怎么去存储和传输这些编码。

## UCS

UCS有两种不同的版本，UCS-2，UCS-4，分别使用2个字节和4个字节编码，UCS-2 可以表示 2^16 = 65536 个码位，差不多足够用了。UCS-4 使用高字节位划分Group，次高字节位划分Plane，第三个字节划分 Rows，第四个字节划分 Cell。

Group 为0的Plane 0 被称作 Basic Multilingual Plane，BMP。将UCS-4 的BMP去掉前面的两个零字节，就得到了UCS-2。

## 统一的问题

ASCII 使用一个字节就可以表示英文字母，Unicode 统一规定的用2个或4个字节表示每一个符号，这样英文字母前面都要用0去填充，浪费存储，导致出现了Unicode的多种存储方式，Unicode 在很长一段时间无法被推广(直到UTF-8)。

## UTF

Unicode/UCS Transformation Format，Unicode 字符集和编码标准，是对Unicode 字符集的具体实现，主要有UTF-16，UTF-32,UTF-8。

A *Unicode transformation format* (UTF) is an algorithmic mapping from every Unicode code point (except surrogate code points) to a unique byte sequence. The ISO/IEC 10646 standard uses the term “UCS transformation format” for UTF; the two terms are merely synonyms for the same concept.

### UTF-16

由RFC2781规定，使用2个字节来表示一个字符，把UCS-2 规定的代码点通过 Big Endian/Little Endian 方式存储，包括 UTF-16，UTF-16BE，UTF-16LE。

UTF-16 需要通过在文件头以名为 BOM（Byte Order Mark）的字符来表明文件是 Big Endian 还是 Little Endian。

由于 UCS-2 没有定义 \uFFFE ，只要出现FF FE 或者  FE FF 这样的字节序列，就能够判断出格式。

FE FF -> Big Endian ,FF FE -> Little Endian，主要看是从高字节解释还是低字节解释

系统一般默认小端序

## BOM

Unicode 编码规范定义，每一个文件的最前面分别加入一个表示编码顺序的字符，这个字符叫做 “**零宽度非换行空格**”(**Zero Width No-Break Space**)，编码是 **FEFF** 。FFFE 在 UCS 中是不存在的字符，UCS 规范建议在传输字节流前，先传输字符 "Zero Width No-Break Space" ，正好是两个字节，FF 比 FE大1，如果一个文本的头两个字节是 FE FF ，就表示该文件采用大端序列存储，如果头两个字节是 FF FE ，就表示文件采用小头方式，也被称作 **BOM**。



## 查看OS 编码序列

主要差别是从高字节还是低字节开始存储、解释。

```java
public static void main(String[] args) throws IOException {
        Integer i = 0x12345678;
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        DataOutputStream dos = new DataOutputStream(bos);
        dos.writeInt(i);

        byte[] bytes = bos.toByteArray();
        for (byte aByte : bytes) {
            System.out.println(Integer.toHexString(aByte));
        }
    }
// 12 34 56 78
```

## UTF-8

被广泛应用的Unicode 的一种实现，变长的编码方式，可以使用**1-4 个字节表示一个符号**，根据不同的符号变化字节长度，使用变长的机制和字符出现的概率分布有关。

- **单字节的符号，字节的第一个设为0，后面7位是这个符号的Unicode 码，对于英文字母，UTF-8编码和ASCII 码是一样的。**
- **多字节的符号，第一个字节的前n位都设为 1，第 n +1 位设为 0，后面字节的前两位一律设为10，剩下的全部为这个字符的Unicode 码。**

**UTF-8 可以不再需要 BOM**

1. 单字节编码的第一个字节为 00-7F       \u 0000 0000 - \u 0000 007F，[0xxx xxxx]
2. 双字节编码的第一个字节为 C0-DF      \u 0000 0080 - \u 0000 07FF，[110x xxxx,10xx xxxx ]
3. 三字节编码的第一个字节为 E0-EF       \u 0001 0000 - \u 0010 FFFF， [1110 xxxx,10xx xxxx ,10xxxxxx ]

只要看到第一个字节的范围就可以知道编码的字节数。

UTF-8 不需要BOM来表明字节序列，但可以用BOM来表明编码方式。字符 “Zero Width No-Break Space“ 的UTF-8 编码是 EF BB BF,如果接受者收到以 EF BB BF 开头的字节流，就知道这是UTF-8编码了。

**编码、转码 只存在字符到字节或者字节到字符的转换时。**

## ANSI

Windows 、IBM PC 上的一种编码，我不想知道。

## 转义字符

**转义字符(Escape Character)**放在字符序列时，会对后面几个字符进行替代并解释，使得转义字符的开头的该字符序列具有不同于该字符单独出现时的语义，转义字符开头的字符序列也被叫做转义序列。

一个转义字符可能并没有它自己的意思，所有转义序列具有两个或更多字符。

通常的功能是 

1. 编码一个句法上的实体(eg:设备命令或则会无法被字母表直接表示的特殊数据)
2. 字符引用，用于表示无法在当前上下文中被键盘录入的字符(eg:回车符)

**转义字符不属于控制字符，控制字符也不属于转义字符。**

如果控制字符的定义是非图形的字符，或者对输出设备有特殊意义的字符，那么针对这些设备的转义字符也是控制字符。

程序设计用的转义字符是图形字符，因此不是控制字符。

大多数ASCII 控制字符单独都具有控制功能，但是他们不是转义字符。

## Emoji

Unicode 允许多个码点组合表示一个Emoji，通过 Zero-width-joiner,ZWJ U+200D连接。

ZWJ，零宽连字符，是一个不打印字符，放在某些需要复杂排版语言的两个字符之间，使得两个本不会发生连字的字符产生了连字效果，对应的Unicode 码位是 U+200D。

## 霍夫曼编码



## 查看编码

```
yum install -y ascii # 查看ascii表
echo "A" | tr -d "\n" | od -An -t dC # 查看字符
echo lgc |od -tcx1  # 十六进制显示同时显示原字符
#dump
hexdump
od
```



## Unicode 

- 映射
- 排序
- 兼容性分解
- 混淆
- 正则表达式
- 双向文门
- 有效存储、查找分布稀疏的编码点数据
- 优化UTF-8解码、字符串比较、NFC标准化

## Base64

之前只知道base64 大部分都是带==，其实编码就是把3个二进制数据用4个ASCII字符表示，是一个转码。

编码时，将三个8位二进制码重新分组成4个6位的二进制码，，不足6位的，右侧补零，高位补两个零，形成4个8位的字节数据，最后取每个字节的十进制值在编码表中对应的字符作为最终的编码数据。

标准的Base64编码要求最终的数据长度是4字节的整数倍，不足4字节倍数时要用[=] padding ，所有base64转码后的字符串不一定存在==。



## 参考资料

[1] BOM https://www.unicode.org/faq/utf_bom.html#BOM

[2] emoji-zwj-sequences https://www.unicode.org/Public/emoji/13.0/emoji-zwj-sequences.txt

