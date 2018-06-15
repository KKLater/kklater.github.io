---
layout:         post
title:          pod 'libwebp'失败的解决办法
subtitle:       
location:  北京 中国   
date:           2018-02-09 10:07:09
categories: iOS 
tags:  iOS Cocoapods
excerpt:  pod 'libwebp'失败，翻墙无解，解决方案
---
![](./_image/2018-02-09/2018-06-15-10-18-25.jpg)

在使用SDWebImage加载webp格式图片时，需要引入库SDWebImage/WebP。

Podfile 文件需要导入：

```ruby

    pod'SDWebImage'

    pod'SDWebImage/WebP'

```
但是执行 `pod install` 命令后，你会发现，报错了。
错误警告：
![](./_image/2018-02-09/2018-06-15-10-16-08.jpg)
what?
why?

分析原因：
我们通过查看SDWebImage 的 `.podspec` pod 配置文件，可知道 `SDWebImage` 支持 `webp` 格式图片是依赖于其 `Webp` 子库，而子库进一步依赖于 `libwebp` 库。
![](./_image/2018-02-09/2018-06-15-10-16-22.jpg)

再看下 `libwebp` 的 `.podspec` pod配置文件：
```json
{
  "name": "libwebp",
  "version": "0.6.0",
  "summary": "Library to encode and decode images in WebP format.",
  "homepage": "https://developers.google.com/speed/webp/",
  "authors": "Google Inc.",
  "license": {
    "type": "BSD",
    "file": "COPYING"
  },
  "source": {
    "git": "https://chromium.googlesource.com/webm/libwebp",
    "tag": "v0.6.0"
  },
  "compiler_flags": "-D_THREAD_SAFE",
  "requires_arc": false,
  "platforms": {
    "osx": null,
    "ios": null,
    "tvos": null,
    "watchos": null
  },
  "subspecs": [
    {
      "name": "webp",
      "header_dir": "webp",
      "source_files": "src/webp/*.h"
    },
    {
      "name": "core",
      "source_files": [
        "src/utils/*.{h,c}",
        "src/dsp/*.{h,c}",
        "src/enc/*.{h,c}",
        "src/dec/*.{h,c}"
      ],
      "dependencies": {
        "libwebp/webp": [

        ]
      }
    },
    {
      "name": "utils",
      "dependencies": {
        "libwebp/core": [

        ]
      }
    },
    {
      "name": "dsp",
      "dependencies": {
        "libwebp/core": [

        ]
      }
    },
    {
      "name": "enc",
      "dependencies": {
        "libwebp/core": [

        ]
      }
    },
    {
      "name": "dec",
      "dependencies": {
        "libwebp/core": [

        ]
      }
    },
    {
      "name": "demux",
      "source_files": "src/demux/*.{h,c}",
      "dependencies": {
        "libwebp/core": [

        ]
      }
    },
    {
      "name": "mux",
      "source_files": "src/mux/*.{h,c}",
      "dependencies": {
        "libwebp/core": [

        ]
      }
    }
  ]
}
```

可知道其库源地址为 `https://chromium.googlesource.com/webm/libwebp`。

通过域名可知，这是来自google的，而在国内，google是被屏蔽的。so~
可叹国墙强大！

试试翻墙咋样？不用试了，你会失败的，静静崇拜国家之强大吧。

那咋解决呢？

我们知道 `libwebp` 是个库，那万能的 `github` 有没有其镜像库呢？试着在 `github` 搜了下，还真是惊喜不断，果然有这样一个库。
![](./_image/2018-02-09/2018-06-15-10-18-07.jpg)

因为 `pod` 在执行 `pod install` 时，是根据其 `source` 中 `git` 地址源来加载源代码的，我们在 `google` 拿不到源代码，那是不是可以试着用 `git` 镜像地址替换掉其`podspec` 中的 `google` 源地址的方法，让`pod` 执行 `pod install`  的时候，从 `git` 镜像来拉去 `libwebp` 源代码呢？

我们用 `git` 库的源地址替换了下 `google` 的源地址，执行 `pod install` ，成功。

总结一下，`SDWebImage` 库支持加载 `webp` 格式的图片，依赖于`google` 出的 `libwebp` 库，而 `pod` 里面的 `libwebp` 库的源代码地址为 `google` 的地址，因为国家之强大，我们是不能从` google`拿到其源码的，于是想到使用 `github` 上的镜像地址来下载 `libwebp` 源码。于是就使用 `github` 上的镜像地址替换了 `libwebp` 的 `podspec`文件的 `source` 地址，执行 `pod install` 成功。

操作步骤：
1、在终端执行 `pod repo` 获取到 `cocoapods` 本地库地址。
![](./_image/2018-02-09/2018-06-15-10-16-38.jpg)

2、Findle 右键，前往我们获取到的本地`pod`仓库地址
![](./_image/2018-02-09/2018-06-15-10-16-50.jpg)

![](./_image/2018-02-09/2018-06-15-10-17-01.jpg)

3、仓库内搜索 `libwebp`，并打开搜索到的文件夹。

![](./_image/2018-02-09/2018-06-15-10-17-12.jpg)

4、打开错误版本号对应的 `podspec` 文件
![](./_image/2018-02-09/2018-06-15-10-17-24.jpg)

![](./_image/2018-02-09/2018-06-15-10-17-35.jpg)

5、找到对应 `source - git` 地址，替换为 `https://github.com/webmproject/libwebp.git`

![](./_image/2018-02-09/2018-06-15-10-17-47.jpg)

6、重新执行 `pod install` 等待成功。

`libwebp git` 源地址：
```ruby
https://github.com/webmproject/libwebp.git
```
替换 libwebp.podspec 的源地址：
```ruby
https://chromium.googlesource.com/webm/libwebp
```

**注意点：** `webmproject/libwebp` 镜像未必包含你所用到的版本号，修改 `podspec` 前请先确认支持你所需要的版本。如果没有支持，那只能降低版本要求，或者自求多福了！