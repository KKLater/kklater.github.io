---
layout:         post
title:          Cocoapods 勿信Bate版
subtitle:       
location:  北京 中国   
date:           2017.03.09 02:45
categories: CocoaPods
tags:  CocoaPods
excerpt:  如果你的项目，执行 `pod update --no-repo-update`之后，无论模拟器运行还是真机运行，都是出现类似的`shell script`错误。很不幸，或许你当前使用的cocoapod是BATE版本，请升级到最新正式版本。
---
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/312211-c2c4dfb7ae03d1c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你的项目，执行 `pod update --no-repo-update`之后，无论模拟器运行还是真机运行，都是出现类似的`shell script`错误。很不幸，或许你当前使用的cocoapod是BATE版本，请升级到最新正式版本。

例如本主公司项目组成员都是使用的1.2.0正式版，而我的个人电脑使用的是1.2.0 bate.1版本。结果在我的个人电脑正常操作后，无论怎么运行，都会出现运行脚本错误，最后cocoapod降级到1.2.0正式版后，运行，错误排除！

多么惨痛的教训：**勿信BATE版！！！**