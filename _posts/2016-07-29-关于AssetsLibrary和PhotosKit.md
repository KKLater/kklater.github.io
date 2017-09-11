---
layout:     post
title:      关于AssetsLibrary和PhotosKit
date:       2016-07-29 17:48
summary:    关于AssetsLibrary和PhotosKit
categories: jekyll pixyll
---

随着iOS不断的升级，苹果逐步放弃了AssetsLibrary，提供了新的框架Photos。

以下是`AssetsLibrary`与`Photos`的对比：

|       | AssetsLibrary  | PhotosKit         |
| ----- | -------------- | ----------------- |
| 资源库   | AssetsLibrary  | PHPhotoLibrary    |
| 相册    | ALAssetsGroup  | PHAssetCollection |
| 照片或视频 | ALAsset        | PHAsset           |
| 过滤    | ALAssetsFilter | PHFetchOptions    |

## 1、组织结构对比

##### AssetsLibrary组织结构

- AssetsLibrary: 代表整个设备中的资源库（照片库），通过 AssetsLibrary 可以获取和包括设备中的照片和视频


- ALAssetsGroup: 映射照片库中的一个相册，通过 ALAssetsGroup 可以获取某个相册的信息，相册下的资源，同时也可以对某个相册添加资源。


- ALAsset: 映射照片库中的一个照片或视频，通过 ALAsset 可以获取某个照片或视频的详细信息，或者保存照片和视频。


- ALAssetRepresentation: ALAssetRepresentation 是对 ALAsset 的封装（但不是其子类），可以更方便地获取 ALAsset 中的资源信息，每个 ALAsset 都有至少有一个 ALAssetRepresentation 对象，可以通过 defaultRepresentation 获取。而例如使用系统相机应用拍摄的 RAW + JPEG 照片，则会有两个 ALAssetRepresentation，一个封装了照片的 RAW 信息，另一个则封装了照片的 JPEG 信息。
- ALAssetsFilter：照片的过滤器

##### PhotoKit组织结构

![PhotoKit](/Users/Later/Desktop/PhotoKit.png)

* PHPhotoLibrary：资源库
* PHAssetCollection：PHCollection 的子类，表示一个相册或者一个时刻，或者是一个「智能相册（系统提供的特定的一系列相册，例如：最近删除，视频列表，收藏等等，如下图所示）
* PHAsset: 代表照片库中的一个资源，跟 ALAsset 类似，通过 PHAsset 可以获取和保存资源
* PHFetchOptions：过滤，取资源时设置，过滤相册资源

## 2、资源获取

### 2.1、获取资源权限

AssetsLibrary获取资源权限：


```objc
aa
NSString *tipTextWhenNoPhotosAuthorization; // 提示语
// 获取当前应用对照片的访问授权状态
ALAuthorizationStatus authorizationStatus = [ALAssetsLibrary authorizationStatus];
// 如果没有获取访问授权，或者访问授权状态已经被明确禁止，则显示提示语，引导用户开启授权
if (authorizationStatus == ALAuthorizationStatusRestricted || authorizationStatus == ALAuthorizationStatusDenied) {
    NSDictionary *mainInfoDictionary = [[NSBundle mainBundle] infoDictionary];
    NSString *appName = [mainInfoDictionary objectForKey:@"CFBundleDisplayName"];
    tipTextWhenNoPhotosAuthorization = [NSString stringWithFormat:@"请在设备的\"设置-隐私-照片\"选项中，允许%@访问你的手机相册", appName];
    // 展示提示语
}
```

PhotosKit获取资源权限：

```objc
+ (void)fbt_libraryAuthorizationCompleted:(void(^)(BOOL))completed {
    PHAuthorizationStatus status = [PHPhotoLibrary authorizationStatus];
  	//直接询问用户使用权限状态
    switch (status) {
        case PHAuthorizationStatusNotDetermined: {	//没有设置过，则直接询问设置
            [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
                    switch (status) {
                        case PHAuthorizationStatusAuthorized: {
                            !completed ?: completed(YES);
                            break;
                        }
                        default: {
                            !completed ?: completed(NO);
                            break;
                        }
                    }
            }];
            break;
        }
        case PHAuthorizationStatusRestricted:
        case PHAuthorizationStatusDenied: {
          //拒绝或限制了权限
            !completed ?: completed(NO);
            break;
        }
        case PHAuthorizationStatusAuthorized:
      	//准许使用权限
        default: {
            !completed ?: completed(YES);
            break;
        }
    }
}
//获取使用权限状态后，如果当前为拒绝或者限制权限，则可以通过以下方式直接引导用户跳转到设置项，配置用户权限
- (void)openSetting {
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:nil message:@"此应用没有权限访问您的相册或视频，打开“隐私设置”获取访问权限？" preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDestructive handler:^(UIAlertAction * _Nonnull action) {
      //“设置”URL
        NSURL *url = [NSURL URLWithString:UIApplicationOpenSettingsURLString];
        if ([[UIApplication sharedApplication] canOpenURL:url]) {
            [[UIApplication sharedApplication] openURL:url];
        }
    }];
    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        [self dismissViewControllerAnimated:YES completion:nil];
    }];
    [alert addAction:cancelAction];
    [alert addAction:sureAction];
    [self presentViewController:alert animated:YES completion:nil];
}
```

### 2.2获取相册资源

AssetsLibrary获取相册资源：

```objc
_assetsLibrary = [[ALAssetsLibrary alloc] init];
_albumsArray = [[NSMutableArray alloc] init];
[_assetsLibrary enumerateGroupsWithTypes:ALAssetsGroupAll usingBlock:^(ALAssetsGroup *group, BOOL *stop) {
    if (group) {
      	//设置过滤
        [group setAssetsFilter:[ALAssetsFilter allPhotos]];
        if (group.numberOfAssets > 0) {
            // 把相册储存到数组中，方便后面展示相册时使用
            [_albumsArray addObject:group];
        }
    } else {
        if ([_albumsArray count] > 0) {
            // 把所有的相册储存完毕，可以展示相册列表
        } else {
            // 没有任何有资源的相册，输出提示
        }
    }
} failureBlock:^(NSError *error) {
    NSLog(@"Asset group not found!\n");
}];
```

AssetsLibraby 通过

```objc
- (void)enumerateGroupsWithTypes:(ALAssetsGroupType)types usingBlock:(ALAssetsLibraryGroupsEnumerationResultsBlock)enumerationBlock failureBlock:(ALAssetsLibraryAccessFailureBlock)failureBlock;
```

方法获取所有相册对象ALAssetsGroup。ALAssetsGroup相册对象可通过`- (void)setAssetsFilter:(ALAssetsFilter *)filter;`方法设置过滤。亦可通过`- (NSInteger)numberOfAssets;`方法获取当前相册的资源个数。

PhotoKit获取相册资源就没有这么直接了，其过滤原则也是灵活的多。以获取照片“相机胶卷”相册为例：

```objc
//获取相机胶卷
PHFetchResult  *cameraFetch = [PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeSmartAlbum subtype:PHAssetCollectionSubtypeSmartAlbumUserLibrary options:nil];
PHAssetCollection *assetCollection  = [cameraFetch lastObject];
```

PhotosKit通过`+ (PHFetchResult<PHAssetCollection *> *)fetchAssetCollectionsWithType:(PHAssetCollectionType)type subtype:(PHAssetCollectionSubtype)subtype options:(nullable PHFetchOptions *)options;`方法获取所有符合过滤的相册资源，其中，`PHAssetCollectionType`表示相册资源的类型，`PHAssetCollectionSubtype`表示各类型下的子类。其分别如下：

| PHAssetCollectionType           | PHAssetCollectionSubtype                 |        |
| ------------------------------- | ---------------------------------------- | ------ |
| PHAssetCollectionTypeAlbum      | PHAssetCollectionSubtypeAlbumRegular     | 定期     |
|                                 | PHAssetCollectionSubtypeAlbumSyncedEvent | 同步时刻   |
|                                 | PHAssetCollectionSubtypeAlbumSyncedFaces | 同步人脸识别 |
|                                 | PHAssetCollectionSubtypeAlbumSyncedAlbum | 同步相册   |
|                                 | PHAssetCollectionSubtypeAlbumImported    | 重要     |
|                                 | PHAssetCollectionSubtypeAlbumMyPhotoStream | 我的相册组  |
|                                 | PHAssetCollectionSubtypeAlbumCloudShared | 云共享    |
| PHAssetCollectionTypeSmartAlbum | PHAssetCollectionSubtypeSmartAlbumGeneric | 普通     |
|                                 | PHAssetCollectionSubtypeSmartAlbumPanoramas | 全景     |
|                                 | PHAssetCollectionSubtypeSmartAlbumVideos | 视频     |
|                                 | PHAssetCollectionSubtypeSmartAlbumFavorites | 最喜欢    |
|                                 | PHAssetCollectionSubtypeSmartAlbumTimelapses | 时间轴    |
|                                 | PHAssetCollectionSubtypeSmartAlbumAllHidden | 隐藏     |
|                                 | PHAssetCollectionSubtypeSmartAlbumRecentlyAdded | 新添加    |
|                                 | PHAssetCollectionSubtypeSmartAlbumBursts |        |
|                                 | PHAssetCollectionSubtypeSmartAlbumSlomoVideos | 延时摄影   |
|                                 | PHAssetCollectionSubtypeSmartAlbumUserLibrary | 用户相册   |
|                                 | PHAssetCollectionSubtypeSmartAlbumSelfPortraits |        |
|                                 | PHAssetCollectionSubtypeSmartAlbumScreenshots | 截屏     |
| 不关心子类型的可以使用                     | PHAssetCollectionSubtypeAny              | 任意     |

除此之外，亦可以通过设置PHFetchOptions，来做更详细的过滤。

### 2.3、获取照片资源

AssetLibrary获取照片资源

```objc
_imagesAssetArray = [[NSMutableArray alloc] init];
[assetsGroup enumerateAssetsWithOptions:NSEnumerationReverse usingBlock:^(ALAsset *result, NSUInteger index, BOOL *stop) {
    if (result) {
        [_imagesAssetArray addObject:result];
    } else {
        // result 为 nil，即遍历相片或视频完毕，可以展示资源列表
    }
}];
```

获取照片资源详细信息

```objc
// 获取资源图片的详细资源信息，其中 imageAsset 是某个资源的 ALAsset 对象
ALAssetRepresentation *representation = [imageAsset defaultRepresentation];
// 获取资源图片的 fullScreenImage
UIImage *contentImage = [UIImage imageWithCGImage:[representation fullScreenImage]];
```

PhotosKit获取照片资源：

```objc
//options：用于对照片的过滤，设置type、排序等
PHFetchResult *fetchR    = [PHAsset fetchAssetsInAssetCollection:assetCollection options:[self _fbt_options]];
//fetchR对象内则为PHAsset对象
//根据PHAsset获取具体的资源图片，需使用到PHImageManager类的实例方法
- (PHImageRequestID)requestImageForAsset:(PHAsset *)asset targetSize:(CGSize)targetSize contentMode:(PHImageContentMode)contentMode options:(nullable PHImageRequestOptions *)options resultHandler:(void (^)(UIImage *__nullable result, NSDictionary *__nullable info))resultHandler;
```

注：PhotoKit的获取照片，使用`PHImageManager`的子类`PHImageCachingManager`优化，`PHImageCachingManagery`用于图像处理过程的缓存，例如在collectionView上加载图片，可以使用其提前缓存要展示的图片，增加流畅性。



1. 压缩！