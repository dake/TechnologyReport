# iOS 图片、音视频存储方式调研结果

## 一、图片
### 1、Activity - 存储图像

|系统\格式|jpg|png|gif|bmp|apng|webp|icns|ico|tiff|
|:-------------:|:-------------:|:-------------:|:-------------|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
|iOS 9 |✓|✓|✓|✓|✘|✘|✓|✓|✓|
|iOS 10|✓|✓|✓|✓|✘|✘|✓|✓|✓|
|iOS 11|✓|✓|✓|✓|✘|✘|✓|✓|✓|
|iOS 12|✓|✓|✓|✓|✘|✓|✓|✓|✓|

> iOS 12 虽然有 webp 入口，但是会保存失败。 

### 2、利用 `Photos.framework` 保存图片

|系统\格式|jpg|png|gif|bmp|apng|webp|icns|ico|tiff|
|:-------------:|:-------------:|:-------------:|:-------------|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
|iOS 9 |✓|✓|✓|✓|✘|✘|✘|✓|✓|
|iOS 10|✓|✓|✓|✓|✘|✘|✘|✓|✓|
|iOS 11|✓|✓|✓|✓|✘|✘|✓|✓|✓|
|iOS 12|✓|✓|✓|✓|✘|✘|✓|✓|✓|

``` objectivec
[PHAssetChangeRequest creationRequestForAssetFromImageAtFileURL:imageURL];
```

### 3、利用 `AssetsLibrary.framework` 保存图片
|系统\格式|jpg|png|gif|bmp|apng|webp|icns|ico|tiff|
|:-------------:|:-------------:|:-------------:|:-------------|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
|iOS 9 |✓|✓|✓|✓|✓|✓|✘|✓|✓|
|iOS 10|✓|✓|✓|✓|✓|✓|✘|✓|✓|
|iOS 11|✓|✓|✓|✓|✓|✓|✘|✓|✓|
|iOS 12|✓|✓|✓|✓|✓|✓|✘|✓|✓|

> 会改变图片格式，有 `alpha` 通道的图会被保存成 `png`，反之 `jpg`

``` objectivec
ALAssetsLibrary *library = [[ALAssetsLibrary alloc] init];
[library writeImageDataToSavedPhotosAlbum:data metadata:nil completionBlock:nil];
```

### 4、 总结
* `icns` 不能用 `AssetsLibrary` 框架来存储。
* `apng`、`webp` 只能用 `AssetsLibrary` 框架来存储。
* `AssetsLibrary` 会改变图片格式，`Photos` 保存不了的时候再考虑用 `AssetsLibrary`。

## 二、视频
### 1、Activity - 存储视频

|系统\格式|mp4|3gp|avi|flv|mov|m4v|mkv|mpg|mpeg|wmv|
|:-------------:|:-------------:|:-------------:|:-------------|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|
|iOS 9 |✓|✓|✘|✘|✓|✓|✘|✘|✘|✘|
|iOS 10|✓|✘|✘|✘|✓|✓|✘|✘|✘|✘|
|iOS 11|✓|✘|✘|✘|✓|✓|✘|✘|✘|✘|
|iOS 12|✓|✓|✘|✘|✓|✓|✘|✘|✘|✘|

### 2、`Photos.framework` 和 `AssetsLibrary.framework`

经过反复验证，只有 `3gp`、`mov`、`m4v`、`mp4` 4 种格式支持用以上 2 个框架保存到相册，4 种格式在不同系统版本上略有差异。

|系统\框架|Photos|AssetsLibrary|
|:-------------:|:-------------:|:-------------:|
|iOS 9/10/12|3gp、mov、m4v、mp4|mov m4v mp4|
|iOS 11|mov、m4v、mp4|mov m4v mp4|

### 3、总结
* 系统相册对视频文件的保存仅支持 `3gp`、`mov`、`m4v`、`mp4` 4 种格式。
* 存储视频到相册 `Photos.framework` 可完全替代 `AssetsLibrary.framework`。
* `iOS 11` 无法保存 `3gp` 视频文件到相册。

## 三、音频
|系统\格式|`aac`、`mp3`、`amr`、`flac`、`wav`、`wma`、`aiff`、`ogg`|
|:-------------:|:-------------:|
|Activity|无入口|
|Photos.framework|无保存接口|
|AssetsLibrary.framework|无保存接口|

