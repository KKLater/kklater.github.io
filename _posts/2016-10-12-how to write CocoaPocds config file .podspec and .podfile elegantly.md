---
layout:         post
title:           如何优雅的书写CocoaPocds配置文件.podspec和.podfile
card-image: http://ouo0w16s7.bkt.clouddn.com/cocoapods%20guides.png?e=1515744001&token=X-k_MFo98a0VybdWmejDMbKtt0VLy2zl6vAXUB4r:5yBOJS5fvCkMLtoXus9pJKL_Wy8
subtitle:       
location:  北京 中国   
date:           2016.10.12 10:14
categories: CocoaPods iOS
tags:  CocoaPods
excerpt:  一直以来，`CocoaPods`都没有提供方便书写`podspec`配置文件的工具，`podfile`文件书写工具`CocoaPods.App`也是近期才刚刚出现不久。在没有代码提示，忘记关键词，书写错误，格式错误等等问题比比皆是的情况下，我们该如何“优雅”的书写`CocoaPods`的配置文件呢？
---

![CocoaPocds.png](http://upload-images.jianshu.io/upload_images/312211-29a47b5c209bf53c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一直以来，`CocoaPods`都没有提供方便书写`podspec`配置文件的工具，`podfile`文件书写工具`CocoaPods.App`也是近期才刚刚出现不久。在没有代码提示，忘记关键词，书写错误，格式错误等等问题比比皆是的情况下，我们该如何“优雅”的书写`CocoaPods`的配置文件呢？

经咨询，大体了解到，有使用Xcode创建文件后，直接书写配置文件的，也有使用Vim书写的，更有甚者直接使用文本编辑的。

这里推荐给大家一种方式：`Atom` + `Snippet`的方式。

关于`Atom`，不过多介绍，可以直接百度谷歌下。这里着重介绍其`Snippet`功能及如何利用`Snippet`功能优雅的书写`CocoaPods`配置文件。

首先，我们需要找到`Atom`的`snippet`配置文件:

![`snippet`配置文件.png](http://upload-images.jianshu.io/upload_images/312211-42f366458401b311.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击进入界面，这里就是我们的`snippet`编辑面板了：


![`snippet`编辑面板.png](http://upload-images.jianshu.io/upload_images/312211-682fb1e53a2a5dfe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
因为`CocoaPods`的配置文件都是基于`ruby`的，所以我们的snippet也是基于ruby来写的。所以，这里需要引入源：`'.source.ruby':`。（此处使用的事`cosn`语言方式，语法简单，又是个安利。）
引入源文件之后，例如我们要书写下面的代码：

```ruby
Pod :: Spec.new do |s|

end
```

此处我们需要配置的snippet代码为：

```ruby
'Pod :: Spec.new do |variable| … end':
    'prefix': 'PodSpecNew'
    'body': 'Pod :: Spec.new do |${1:variable}|\n\t$0\nend'
```

`command + s`保存该`snippet`文件。

试一下，在创建的`.podspec`文件里面直接输入`PodSpecNew`，是不是就有代码提示了？是不是有点小优雅了呢？
 
![演示.png](http://upload-images.jianshu.io/upload_images/312211-df513bbeb9378a3c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按照上面的步骤，再继续完成`pod`配置文件的各关键词的`snippet`，就可以拥有一个带代码提示的`pod`编辑器了，既提高了效率，又降低了错误率。

步骤总结：
  1、找到`Atom`的`snippet`配置文件。
  2、查找到要`ruby`源并引入。
  3、`cosn`书写`snippet`文件。
  4、测试

额，要完成上面的步骤，觉得还是得先安装一个`Atom`...

今天的看客们都比较幸运，此处已经做了`.podspec`和`.podfile`文件的书写`snippet`插件，现在可以直接在`Atom`的插件搜索部分，搜索[cocoa-pods-ruby-snippets](https://atom.io/packages/cocoa-pods-ruby-snippets)来直接安装使用了。

什么？你还不知道怎么安装插件？额，良心引导，直接到百度谷歌一下吧。


***

关于插件的发布，后续再谈，看客群众耐心等待。有兴趣的可以提前看看`乖小鬼`的：[如何发布一个Atom的package](http://www.jianshu.com/p/98f99c20493c)，不错的参考。