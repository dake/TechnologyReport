# iOS 图片存储方式调研结果

## 一、Activity - 存储图像

| 系统\格式 | jpg| png| gif| bmp|apng|webp|icns|ico|tiff|
|:-------------:|:-------------:|:-------------:|:-------------|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
|iOS 9 |✓|✓|✓|✓|✘|✘|✓|✓|✓|
|iOS 10|✓|✓|✓|✓|✘|✘|✓|✓|✓|
|iOS 11|✓|✓|✓|✓|✘|✘|✓|✓|✓|
|iOS 12|✓|✓|✓|✓|✘|✓|✓|✓|✓|

> iOS 12 虽然有 webp 入口，但是会保存失败。 

## 二、利用 `Photos.framework` 保存图片

| 系统\格式 | jpg| png| gif| bmp|apng|webp|icns|ico|tiff|
|:-------------:|:-------------:|:-------------:|:-------------|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
|iOS 9 |✓|✓|✓|✓|✘|✘|✘|✓|✓|
|iOS 10|✓|✓|✓|✓|✘|✘|✘|✓|✓|
|iOS 11|✓|✓|✓|✓|✘|✘|✓|✓|✓|
|iOS 12|✓|✓|✓|✓|✘|✘|✓|✓|✓|

``` objectivec
[PHAssetChangeRequest creationRequestForAssetFromImageAtFileURL:imageURL];
```

## 三、利用 `AssetsLibrary.framework` 保存图片
| 系统\格式 | jpg| png| gif| bmp|apng|webp|icns|ico|tiff|
|:-------------:|:-------------:|:-------------:|:-------------|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
|iOS 9 |✓|✓|✓|✓|✓|✓|✘|✓|✓|
|iOS 10|✓|✓|✓|✓|✓|✓|✘|✓|✓|
|iOS 11|✓|✓|✓|✓|✓|✓|✘|✓|✓|
|iOS 12|✓|✓|✓|✓|✓|✓|✘|✓|✓|

> `webp` 可以保存成功，但是会变成 `png` 图片

``` objectivec
ALAssetsLibrary *library = [[ALAssetsLibrary alloc] init];
[library writeImageDataToSavedPhotosAlbum:data metadata:nil completionBlock:nil];
```

## 四、 总结
* `icns` 不能用 `AssestsLibray` 框架来存储。
* `apng`、`webp` 只能用 `AssestsLibray` 框架来存储。
* `jpg`、`png`、`bmp`、`gif`、`ico`、`tiff` 2 种框架都支持。

