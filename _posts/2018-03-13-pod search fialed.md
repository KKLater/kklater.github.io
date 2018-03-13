---
layout:         post
title:          pod  search 失败的解决办法
subtitle:       
location:  北京 中国   
date:           2018-03-13 10:07:09
categories: iOS 
tags:  iOS Cocoapods
excerpt:  pod  search 失败的解决办法
---

iOS开发中经常会用到 cocoapods，尤其是组件化过程中，抽离的组件经常会做成 pod 私有库，但是，最近遇到一个问题，私有库制作成功了，使用 pod search 搜索自己刚刚制作成功的私有库，怎么也搜索不到。

what?

最终找到解决方案，移除掉 cocoapods 生成的搜索映射，重新执行搜索操作。此时，cocoapods 会重新创建搜索映射，则可以搜索到新创建的私有库了。

移除 cocoapods 搜索映射的操作：

```shell
rm ~/Library/Caches/CocoaPods/search_index.json
```


