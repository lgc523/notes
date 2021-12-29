---
title: "Pixels Image"
date: 2021-12-29T23:40:03+08:00
draft: true
toc: true
images:
tags: 
  - engineering
---

最近遇到一个 图片 base64 编码后传输的问题，有很多不懂的地方，记录一下。

遇到的问题就是，一个接口图片上传功能，上传的是图片数据流 base64 编码后的字符串，问题在于接口限制了大小 2MB，并且只支持四种格式的图片数据 (jpg、png、gif、bmp)。

我这边作为一个中间平台系统，需要对上游传过来的 base64 字符串进行校验过滤再传递给下游，要通过 base64 之后的字符串来校验图片大小和图片的格式。

目前的解决方案是，解码 base64 -> 字节数组，借助前几字节的标记数据 (16进制) 来区分图片格式，字节数组限制图片大小。

具体参考的数据是：

- Jpg      FF D8
- bmp   42 4D
- gif       47 49 46 38 39|37 61
- png     89 50 4E 47 0D 0A 1A 0A

图形学涉及的内容很多，这里简单记录一些简单的知识点，从英寸、像素开始。

## pixels

图像数据是一个二维的像素矩阵网格，每一个网格代表一个像素。

### 常见的色彩模式/空间

- **RGB**

  RGB 色彩模式适用于显示器、投影仪，由红、绿、蓝三个基色组成，也称为三个颜色通道。

  每个颜色通道的 "颜色浓度" 用一个字节来存储，无符号可表示范围 0-255，所以一个像素占了 3 字节。

- **CMYK**

  适用于打印机、印刷机，色彩通道多，但是色彩表示的范围小，效率低，质量差。

- **Lab**

  l 为无色通道，a 为 red-green 通道，b 为 yellow-blue 通道，是比较接近人眼视觉显示的一种颜色模式。



### inch

**1 inch = 2.54 cm**

### px

就是一个像素，也就是一个显示器上的一个方格。

### 分辨率

也就是显示器上的像素点，我的显示器是 3840 *2160，也就是显示器一共显示 3840 * 2160 个像素点。

那么截一张图的原始大小就是 3840 * 2160 * 3 /1024/1024 = 23.73046875 MB。

### ppi

Pixels Per Inch，一英寸(2.54 * 2.54 cm) 区域上的的像素点数，ppi 越高越清晰。

计算公式 根据勾股定理，计算出对角线上的像素点，除以对角线长度（inch)。

## 图像压缩

根据上面计算的在 3840 * 2160 分辨率的显示器上截一张图就需要 23 MB，这样对总线和网络都带来了很大的压力，所以需要图像压缩，来进行高效传输。

特别是视频，每一秒几十桢图片播放，越清晰图像堆叠的视频尺寸就越大。

**TODO**

## hexdump

使用 hexdump [  less | more | most ] 查看文件的 16 进制编码，可以粗略估计图片的大小，和查看一些标记位。

## 常见的图片格式



## GIF 



## 字节数组转十六进制

```java
1.String.format("%02x", byte)
2.public static String encodeHexString(byte[] data) {
    Formatter formatter = new Formatter();
    for (byte b : data) {
        formatter.format("%02x", b);
    }
    String result = formatter.toString();
    formatter.close();
    return result;
}
3.commons-codec lib
  Hex.encodeHexString(byteArray)
```

