---
layout:     post
title:      iOS PDFKit 开发指北
subtitle:   Guideline of PDFKit on iOS 11
date:       2018-11-21
author:     Japho
header-img: postimage/post-bg-20181121.jpg
catalog:    true
tags:
    - iOS
---

## 前言

2017年夏天，在苹果全球开发者大会（WWDC）上，苹果公司终于推出了针对于 iOS 的 PDFKit 支持。PDFKit 自从 MacOS 10.4 以来一直在 AppKit for MacOS 中。但 UIKit 却迟迟得不到支持，尽管苹果公司之前在 iBooks 和 Mail 中使用过 PDFKit ， 但是该框架并未向开发人员开房。

PDFKit 包含了大量关于 PDF 相关的功能，例如，打开，修改，绘图和保存 PDF ，也包含了搜索文本。在 iOS 11 后，苹果终于开放了 PDFKit 。目前（虽然离 PDFKit 发布已经过了一年多），但是目前中文资料和 Demo 确实比较少，下面笔者就带着大家简单的了解一下 PDFKit。

![](https://ws1.sinaimg.cn/large/006tNbRwly1fxfu2iy4ydj30m80ct43b.jpg)

## 核心功能

主要核心功能如下：

**PDFView**

>用于显示 PDF ，包括选择的内容，导航，缩放等功能。

**PDFDocument**

>表示允许您写入、搜索和选择PDF数据的PDF数据或文件。

**PDFPage**

>呈现PDF数据，添加注释，获取页面文本等等。

**PDFAnnotation**

>PDF 中的附加内容，包括注释、链接、表单等。

## 实现一个简单的 PDF 阅读器

