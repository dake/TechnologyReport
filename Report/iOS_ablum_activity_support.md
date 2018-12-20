# iOS 图片、音视频存储方式调研结果

## 一、图片
### 1、Activity - 存储图像
*有无保存入口*

|系统\格式|jpg|png|gif|bmp|apng|webp|pcx|tga|icns|ico|tiff|svg|psd|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|iOS 9 |✓|✓|✓|✓|✘|✘|✘|✓|✓|✓|✓|✘|✘|
|iOS 10|✓|✓|✓|✓|✘|✘|✘|✓|✓|✓|✓|✘|✓|
|iOS 11|✓|✓|✓|✓|✘|✘|✘|✓|✓|✓|✓|✘|✓|
|iOS 12|✓|✓|✓|✓|✘|✓|✘|✓|✓|✓|✓|✘|✓|

> iOS 12 虽然有 webp 入口，但是会保存失败。 

### 2、利用 `Photos.framework` 和 `AssetsLibrary.framework` 
#### 1). Photos > creationRequestForAssetFromImageAtFileURL

|系统\格式|webp|icns|apng|svg|psd|pcx|tga、jpg、png、gif、bmp、ico、tiff|多帧|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|iOS 9   |✘|✘|✘|✘|✘|✘|✓|正常|
|iOS 10 |✘|✘|✘|✘|✓|✘|✓|正常|
|iOS 11 |✘|✓|✘|✘|✓|✘|✓|正常|
|iOS 12 |✘|✓|✘|✘|✓|✘|✓|正常|

```objc
- (void)saveImageAtURL:(NSURL *)URL
{
    __block BOOL hasChanges = YES;
    [PHPhotoLibrary.sharedPhotoLibrary performChanges:^{
        PHAssetChangeRequest *req = [PHAssetChangeRequest creationRequestForAssetFromImageAtFileURL:URL];
        if (nil == req) {
            hasChanges = NO;
        }
    } completionHandler:^(BOOL success, NSError * _Nullable error) {
    }];
}
```


#### 2). Photos > creationRequestForAssetFromImage

|系统\格式|webp|icns|svg|psd|pcx|tga、apng、jpg、png、gif、bmp、ico、tiff|多帧|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|iOS 9   |✘|✘|✘|✘|✘|✓|只有第一帧|
|iOS 10 |✘|✘|✘|✓|✘|✓|只有第一帧|
|iOS 11 |✘|✘|✘|✓|✘|✓|只有第一帧|
|iOS 12 |✘|✘|✘|✓|✘|✓|只有第一帧|

```objc
- (void)saveImageAtURL:(NSURL *)URL
{
    __block BOOL hasChanges = YES;
    [PHPhotoLibrary.sharedPhotoLibrary performChanges:^{
        UIImage *image = [UIImage imageWithContentsOfFile:URL.path];
        if (nil == image) {
            hasChanges = NO;
            return;
        }
        
        PHAssetChangeRequest *req = [PHAssetChangeRequest creationRequestForAssetFromImage:image];
        if (nil == req) {
            hasChanges = NO;
        }
    } completionHandler:^(BOOL success, NSError * _Nullable error) {
    }];
}
```

### ３). ALAssetsLibrary > writeImageDataToSavedPhotosAlbum

|系统\格式|webp|icns|svg|psd|pcx|tga、apng、jpg、png、gif、bmp、ico、tiff|多帧|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|iOS 9   |✘|✘|✘|✘|✘|✓|正常|
|iOS 10 |✘|✘|✘|✓|✘|✓|正常|
|iOS 11 |✘|✓|✘|✓|✘|✓|正常|
|iOS 12 |✘|✓|✘|✓|✘|✓|正常|

```objc
- (void)saveImageAtIndex:(NSUInteger)index
{
    if (index >= _URLs.count) {
        return;
    }
    
    __weak typeof(self) wSelf = self;
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        @autoreleasepool {
            NSURL *URL = wSelf.URLs[index];
            ALAssetsLibrary *library = [[ALAssetsLibrary alloc] init];
            NSData *data = [NSData dataWithContentsOfFile:URL.path];
            __strong typeof(wSelf) sSelf = wSelf;
            [library writeImageDataToSavedPhotosAlbum:data metadata:nil completionBlock:^(NSURL *assetURL, NSError *error) {
                [sSelf saveImageAtIndex:index + 1];
            }];
        }
    });
}
```
### 4、 YYImage > yy_saveToAlbumWithCompletionBlock

* 慎用 YYImage 的保存到相册功能，除了 `png、jpg、gif` 之外的图片格式，它都会做转换，有`alpha`通道的就会被转换成`png` data, 没有的就会被转换成`jpg` data。

```objc
- (void)yy_saveToAlbumWithCompletionBlock:(void(^)(NSURL *assetURL, NSError *error))completionBlock {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        @autoreleasepool {
            NSData *data = [self _yy_dataRepresentationForSystem:YES];
            ALAssetsLibrary *library = [[ALAssetsLibrary alloc] init];
            [library writeImageDataToSavedPhotosAlbum:data metadata:nil completionBlock:^(NSURL *assetURL, NSError *error){
                if (!completionBlock) return;
                if (pthread_main_np()) {
                    completionBlock(assetURL, error);
                } else {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        completionBlock(assetURL, error);
                    });
                }
            }];
        }
    });
}

/// @param forSystem YES: used for system album (PNG/JPEG/GIF), NO: used for YYImage (PNG/JPEG/GIF/WebP)
- (NSData *)_yy_dataRepresentationForSystem:(BOOL)forSystem {
- }
```

### 3、总结
* `webp` 直接用系统方法都无法保存成功，可以用 `YYImage` 解码之后再用方法 2 或者 方法 3 来存储，多帧 `webp` 保存之后只有第一帧。
* `icns` 在 iOS 9 和 iOS 10 无法读取和保存。
* `creationRequestForAssetFromImage` 对于多帧图片，保存之后只会保留第一帧。

## 二、视频
### 1、Activity - 存储视频
*是否可以原格式保存成功*

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

## 四、iOS 不同系统读取不同格式图片的差异
测试一下类型 `jpg`、`png`、`gif`、`bmp`、`apng`、`webp`、`icns`、`ico`、`tiff`
、`svg`、`psd`、`pcx`、`tga`

|系统\格式|icns|psd|webp|svg|pcx|
|:-:|:-:|:-:|:-:|:-:|:-:|
| iOS 9|✘|✘|✘|✘|✘|
| iOS 10|✘|✓|✘|✘|✘|
| iOS 11|✓|✓|✘|✘|✘|
| iOS 12|✓|✓|✘|✘|✘|

* `webp` 可以通过 `YYImage` 来解析。

