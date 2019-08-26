---
layout:     post
title:      UIButton中imageView和titleLabel的位置调整
subtitle:   头大的问题终于解决了
date:       2016-10-25
author:     Japho
header-img: postimage/post-bg-rwd.jpg
catalog:    true
tags:
    - iOS
---

### 前言

在使用UIButton时，有时候需要调整按钮内部的imageView和titleLabel的位置和尺寸。在默认情况下，按钮内部的imageView和titleLabel的显示效果是图片在左文字在右，然后两者紧挨在一起构成组合居中显示。如下图：

![](http://upload-images.jianshu.io/upload_images/1269906-d70a97d50e4aabe5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以使用`setImageEdgeInsets:`和`setTitleEdgeInsets:`方法来调整两者的位置。在使用这两个方法之前，我们首先要将`imageView`和`titleLabel`定位在UIButton的左上角位置，方便参照坐标调节位置。使用以下语句来定位(UIButton实例名为btn)：

```swift
[[btn setContentHorizontalAlignment: UIControlContentHorizontalAlignmentLeft];  
[btn setContentVerticalAlignment: UIControlContentVerticalAlignmentTop];  
```

另外需要说明，btn的高度为200、宽度为屏幕宽度，imageView的图片的原本尺寸为150x150，`titleLabel`使用了`sizeToFit：`方法。在定义后，我们分多种情况来讨论`imageView`和`titleLabel`的位置：

### 1、正常情况：

使用了上述的语句后，不进行任何操作，此时按钮的显示情况和坐标情况如下：

![](http://upload-images.jianshu.io/upload_images/1269906-d1dcc04b88e252ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
imageView.x=0.000000, imageView.y=0.000000, imageView.width=150.000000, imageView.height=150.000000  
titleLabel.x=150.000000, titleLabel.y=0.000000, titleLabel.width=180.500000, titleLabel.height=21.500000  
```

可以看到：`titleLabel`的x值是150，说明普通状态下，titleLabel的原点是(150 ,0)。

### 2、将imageView居中，将titleLabel左移至贴边：

使用以下语句来实现这种情况：

```swift
[btn setImageEdgeInsets: UIEdgeInsetsMake(0, (btn.bounds.size.width-btn.imageView.bounds.size.width)*0.5, 0, 0)];  
[btn setTitleEdgeInsets: UIEdgeInsetsMake(0, -btn.imageView.bounds.size.width, 0, 0)];  
```

此时按钮的显示情况和坐标情况如下：

![](http://upload-images.jianshu.io/upload_images/1269906-bc9ac6ed11edadf9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，imageView和titleLabel两者是有可能重叠的，并且titleLabel的x值确实是imageView的宽度。

### 3、使imageView和titleLabel都居中：
使用以下语句来实现这种情况

```swift
[btn setImageEdgeInsets: UIEdgeInsetsMake(0, (btn.bounds.size.width-btn.imageView.bounds.size.width)*0.5, 0, 0)];  
[btn setTitleEdgeInsets: UIEdgeInsetsMake(btn.imageView.bounds.size.height, (btn.bounds.size.width-btn.titleLabel.bounds.size.width)*0.5-btn.imageView.bounds.size.width, 0, 0)]; 
```

 此时按钮的显示情况和坐标情况如下：

![](http://upload-images.jianshu.io/upload_images/1269906-41441c76574d6de6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
imageView.x=112.500000, imageView.y=0.000000, imageView.width=150.000000, imageView.height=150.000000  
titleLabel.x=97.250000, titleLabel.y=150.000000, titleLabel.width=180.500000, titleLabel.height=21.500000  
```

此时一切正常，符合猜想。

### 4、这时如果imageView的尺寸被压缩：
使用以下语句来将imageView压缩成100x100：

```swift
#define kImageWidth 100.  
[btn setImageEdgeInsets: UIEdgeInsetsMake(0, (btn.bounds.size.width-kImageWidth)*0.5, btn.bounds.size.height-kImageWidth, (btn.bounds.size.width-kImageWidth)*0.5)];  
[btn setTitleEdgeInsets: UIEdgeInsetsMake(btn.imageView.bounds.size.height, (btn.bounds.size.width-btn.titleLabel.bounds.size.width)*0.5-btn.imageView.bounds.size.width, 0, 0)];  
```

此时按钮的显示情况和坐标情况如下：

![](http://upload-images.jianshu.io/upload_images/1269906-0c7894661b375130.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
imageView.x=137.500000, imageView.y=0.000000, imageView.width=100.000000, imageView.height=100.000000  
titleLabel.x=147.250000, titleLabel.y=100.000000, titleLabel.width=180.500000, titleLabel.height=21.500000  
```

可以发现，问题出现了，此时titleLabel在水平方向上不再居中了。但是我们同时可以发现，titleLabel在垂直方向上的位置仍然是正确的，并且可以看到`titleLabel.y=100`。这说明在使用了`setImageEdgeInsets:`方法后，imageView的尺寸会被改变。

那么为何titleLabel在水平方向上还会偏移呢？原因是这样的，我们一直把titleLabel的初始x值和imageView的宽度等同看待了。而在这里imageView的宽度变小了，但是titleLabel的初始x值仍然是等于缩小前的imageView的宽度的，我们却使用一个缩小后的imageView的宽度来代替titleLabel的初始x值，于是导致了偏移的出现。
要处理这种情况，有两种方法：

### 5、处理方法1，先定义titleLabel再定义imageView：

使用以下语句来实现这种情况：

```swift
#define kImageWidth 100.  
[btn setTitleEdgeInsets: UIEdgeInsetsMake(kImageWidth, (btn.bounds.size.width-btn.titleLabel.bounds.size.width)*0.5-btn.imageView.bounds.size.width, 0, 0)];  
[btn setImageEdgeInsets: UIEdgeInsetsMake(0, (btn.bounds.size.width-kImageWidth)*0.5, btn.bounds.size.height-kImageWidth, (btn.bounds.size.width-kImageWidth)*0.5)];  
```

此时按钮的显示情况和坐标情况如下：

![](http://upload-images.jianshu.io/upload_images/1269906-28f02915e069c005.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
imageView.x=137.500000, imageView.y=0.000000, imageView.width=100.000000, imageView.height=100.000000  
titleLabel.x=97.250000, titleLabel.y=100.000000, titleLabel.width=180.500000, titleLabel.height=21.500000  
```

由于titleLabel在imageView被压缩前就使用了它的宽度来定位，所以能准确定位，也不会被之后的imageView压缩所影响。

### 6、处理方法2，先将imageView形变前的宽度记录下来：

使用以下语句来实现这种情况：

```swift
#define kImageWidth 100
CGFloat imageWidth = btn.imageView.bounds.size.width;
[btn setImageEdgeInsets: UIEdgeInsetsMake(0, (btn.bounds.size.width-kImageWidth)*0.5, btn.bounds.size.height-kImageWidth, (btn.bounds.size.width-kImageWidth)*0.5)];
[btn setTitleEdgeInsets: UIEdgeInsetsMake(btn.imageView.bounds.size.height, (btn.bounds.size.width-btn.titleLabel.bounds.size.width)*0.5-imageWidth, 0, 0)];
```

此时按钮的显示情况和坐标情况如下：

![](http://upload-images.jianshu.io/upload_images/1269906-e7fa7263eef68b24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
imageView.x=137.500000, imageView.y=0.000000, imageView.width=100.000000, imageView.height=100.000000  
titleLabel.x=97.250000, titleLabel.y=100.000000, titleLabel.width=180.500000, titleLabel.height=21.500000  
```

先将imageView的宽度记录下来，用作titleLabel的初始x值，那么之后imageView的压缩就不会再影响到这个值了。

### 7、另外，如果想将imageView放大，比如将imageView的宽度定为180，那么会有以下情况：

![](http://upload-images.jianshu.io/upload_images/1269906-3741758410282937.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
imageView.x=97.500000, imageView.y=0.000000, imageView.width=150.000000, imageView.height=150.000000  
titleLabel.x=97.250000, titleLabel.y=150.000000, titleLabel.width=180.500000, titleLabel.height=21.500000  
```

可以发现，imageView的尺寸还是150x150，并没有没放大，反而imageView的居中受到了影响，产生偏移了。

### 8、经过上面的测试，可以得到以下结论：

- (1)、在UIButton中，titleLabel的初始x值是imageView的宽度；
- (2)、imageView可以被压缩；
- (3)、当imageView被压缩后，imageView的宽度会变小，此时就不可以再用imageView的宽度来代替titleLabel的初始x值来调整位置了；
- (4)、imageView不可以被放大。
