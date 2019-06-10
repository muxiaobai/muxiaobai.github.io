---
title: 前言技术之OCR识别
date: 2018-12-21 09:10:12
tags: 前沿技术
categories: OCR
description: "OCR,文字识别技术"
---

OCR技术是光学字符识别的缩写(Optical Character Recognition)，是通过扫描等光学输入方式将各种票据、报刊、书籍、文稿及其它印刷品的文字转化为图像信息，再利用文字识别技术将图像信息转化为可以使用的计算机输入技术。

主要是两个步骤，先获取输入源数据，例如扫描仪、相机等，然后进行识别，目前流行的识别技术主要是RNN循环神经网络、LSTM等，传统的OCR
<!--more-->
tesseract-ocr 识别，当前使用版本4.0，下载的时候,直接在github中的wiki即可。

win

环境变量

增加一个PATH变量名，变量值还是我的安装路径C:\Program Files (x86)\Tesseract-OCR;

增加一个TESSDATA_PREFIX变量名，变量值还是我的安装路径C:\Program Files (x86)\Tesseract-OCR\tessdata;


```
tesseract -v

tesseract test.png output_1 –l eng

```

[wiki](https://github.com/tesseract-ocr/tesseract/wiki)
[安装](https://www.cnblogs.com/jianqingwang/p/6978724.html)
[使用](https://www.cnblogs.com/cnlian/p/5765871.html)