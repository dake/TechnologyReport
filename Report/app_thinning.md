# App 瘦身
## 一、资源
### 1. 无用资源删除
#### 如何查找无用资源
* 无用图片 LSUnusedResources
* 重复文件 fdupes // 大小对比 > 部分 MD5 签名对比 > 完整 MD5 签名对比 > 逐字节对比

### 2. 图片
* 无损压缩 有损会影响设计 ImageOptim
* xcassets 管理 App Slicing
* 大图片可以选择 Bundle 管理

> Bundle 和 xcassets 的主要区别有：
>1. xcassets 里面的图片，只能通过 imageNamed 加载。Bundle 还可以通过 imageWithContentsOfFile 等方式。
>2. xcassets 里的 2x 和 3x，会根据具体设备分发，不会同时包含。而 Bundle 会都包含。（App Slicing）
>3. xcassets 内，可以对图片进行 Slicing，即裁剪和拉伸。Bundle 不支持。
>4. Bundle 内支持多语言，xcassets 不支持。

### 3. 其它（音视频、网页）
* 远端下载，有离线需求可以进行缓存

## 二、二进制文件
### 1. 编译选项优化
* Bitcode App Slicing
* Generate Debug Symbols // 是否生成 symbol 文件，默认开启。
* Asset Catalog Compiler // 默认选项，不做修改。
* Dead Code Stripping // 删除静态链接的可执行文件中未引用的代码。Debug 设置为 NO， Release 设置为 YES 可减少可执行文件大小。默认选项，不做修改。
* Apple Clang - Code Generation // 默认情况下，Debug 设定为 None[-O0] ，Release 设定为 Fastest,Smallest[-Os]。默认选项，不做修改。
* Swift Compiler - Code Generation // 如果没有特殊情况，使用默认的 Whole Module 优化即可。 它会牺牲部分编译性能，但的优化结果是最好的。故，在 Relese 模式下 -Osize 和 Whole Module 同时开启效果会最好！
* Strip Symbol Information
* Exceptions // 去掉异常支持，Enable C++ Exceptions 和 Enable Objective-C Exceptions 设为 NO，并且 Other C Flags 添加 -fno-exceptions，会影响异常捕获
* Link-Time Optimization // 苹果使用了新的优化方式 Incremental，大大减少了链接的时间。建议开启。开启这个优化后，一方面减少了汇编代码的体积，一方面提高了代码的运行效率。
* Make Strings Read-Only 

### 2. Linkmap
* 删除无用类

> 查找无用 oc 类有两种方式，一种是类似于查找无用资源，通过搜索 "[ClassName alloc/new"、"ClassName *"、"[ClassName class]" 等关键字在代码里是否出现。另一种是通过 otool 命令逆向 __DATA.__objc_classlist 段和 __DATA.__objc_classrefs 段来获取当前所有 oc 类和被引用的 oc 类，两个集合相减就是无用 oc 类。

* 删除无用代码
通过 AppCode 查找无用代码
> 以往 C++ 在链接时，没有被用到的类和方法是不会编进可执行文件里。但 Objctive-C 不同，由于它的动态性，它可以通过类名和方法名获取这个类和方法进行调用，所以编译器会把项目里所有 OC 源文件编进可执行文件里，哪怕该类和方法没有被使用到。
结合 LinkMap 文件的__TEXT.__text，通过正则表达式([+|-][.+\s(.+)])，我们可以提取当前可执行文件里所有 objc 类方法和实例方法（SelectorsAll）。再使用 otool 命令 otool -v -s __DATA __objc_selrefs逆向__DATA.__objc_selrefs 段，提取可执行文件里引用到的方法名（UsedSelectorsAll），我们可以大致分析出 SelectorsAll 里哪些方法是没有被引用的（SelectorsAll-UsedSelectorsAll）。注意，系统 API 的 Protocol 可能被列入无用方法名单里，如 UITableViewDelegate 的方法，我们只需要对这些 Protocol 里的方法加入白名单过滤即可。
另外第三方库的无用 selector 也可以这样扫出来的。

* 删除重复代码

> 可以利用第三方工具 simian 扫描。


### 3. 公共代码
* Frameworks 引用 代码复用 二进制复用

> 文件勾选是为代码复用，多 target 勾选就会产生多份文件。要达到二进制复用，就把需要复用的文件抽到新的 Framework 中，给多 target 引用。

* Bundle 资源复用

> 1. 把需要的资源打包成 Bundle，供多 target 引用。
> 2. extension 的国际化字符串复用 host app 的。// extension 中建立空白的对应语言的国际化字符串文件即可

### 三、其它
* tintColor 来减少部分图片资源
* 减少 swift、oc 混编
* 不支持 Bitcode 的三方动态库，在打包 ipa 时，删除不需要的架构

### 四、工具
* [LSUnusedResources](https://github.com/tinymind/LSUnusedResources) 查找未使用资源
* [fdupes](https://github.com/adrianlopezroche/fdupes) 查找重复文件
* [ImageOptim](https://github.com/ImageOptim/ImageOptim) 压缩图片
* [Asset Catalog Tinkerer](https://github.com/insidegui/AssetCatalogTinkerer) 查看 assets.car
* AppCode 无用代码
* FauxPas

### 五、参考资料
* [iOS 瘦包常见方式梳理](https://mp.weixin.qq.com/s/J_XYpIfDeeWJBlk9sRQMAA)
* [iOS 微信安装包瘦身](http://www.cocoachina.com/ios/20151211/14562.html)






