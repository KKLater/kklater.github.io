---
layout:         post
title:          pod 导入 'openssl-ios-bitcode' 失败
subtitle:       
location:  北京 中国   
date:           2018-1-23 00:41:00
categories: iOS 
tags:  iOS Cocoapods
excerpt:  pod 导入 'openssl-ios-bitcode' 失败，导致项目不能正常运行
---

##问题描述

项目的 Podfile 文件中包含有 'pod 'openssl-ios-bitcode', '~> 1.0.212'' ， ’pod update —no-repo-update'，报错。

![](./_image/2018-01-23-00-43-02.jpg)

##解决方案

```shell
 rm -r ${TMPDIR}/openssl/
```

![](./_image/6757D235-7D85-4AFC-A578-D02D05D190D3.png)

参考资料:[https://github.com/FredericJacobs/OpenSSL-Pod/issues/32](https://github.com/FredericJacobs/OpenSSL-Pod/issues/32)

##问题原因

思考中，有知道的给告诉下。