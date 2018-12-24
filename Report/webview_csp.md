# iOS WebView 的 层次关系

[研究 wkwebview 的子 view 和 <body> 的关系](https://weibo.com/3321824014/H3vJukUdK )

接着这个话题，总结一下 iOS 中 关于 CSP 策略的一些知识点

# iOS WebView CSP

## iOS ATS

iOS 中启用 ATS 需要的前提条件：

+ 服务器TLS版本至少是1.2版本
+ 连接加密只允许几种先进的加密

```
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
```
+ 证书必须使用SHA256或者更好的哈希算法进行签名，要么是2048位或者更长的RSA密钥，要么就是256位或更长的ECC密钥。

server需要提供HTTPS（TLS 1.2）的接口，可以在终端使用下面的命令验证server的API是否符合ATS标准。

```
nscurl --ats-diagnostics https://www.google.com
```

+ HTTP/2 中只能使用 TLSv1.2+，还禁用了几百种 CipherSuite（[详见：TLS 1.2 Cipher Suite Black List](http://http2.github.io/http2-spec/#BadCipherSuites)）。实际上，HTTP/2 允许使用的 CipherSuite 必须采用具有前向安全性的密钥交换算法，不允许使用 RSA 密钥交换。这也是为什么 RSA Private Key 无法用于解密 HTTP/2 加密流量

### iOS9 以后 默认情况下非 HTTPS 的网络访问是被禁止的 ，考虑到兼容性 ，可以通过以下三个 KEY 来禁用 ATS 

+ NSAllowsArbitraryLoadsForMedia
+ NSAllowsArbitraryLoadsInWebContent
+ NSAllowsLocalNetworking

### PS：在 iOS9 上这三个 KEY 不生效 ， iOS10 如果设置了上述 KEY 则会忽略 NSAllowsArbitraryLoads

| KEY | 值 | 组件 | 系统版本 | 结果 |
| :------:| :------: | :------: | :------: | :------: | 
| NSAllowsArbitraryLoads | YES | Webview | iOS9/iOS10 | 彻底禁用 ATS |
| NSAllowsArbitraryLoads | YES | URLSession | iOS9/iOS10 | 彻底禁用 ATS |
| NSAllowsArbitraryLoads | NO | Webview| iOS9/iOS10 | 未禁用 ATS |
| NSAllowsArbitraryLoads | NO | URLSession | iOS9/iOS10 | 未禁用 ATS |
| NSAllowsArbitraryLoads: NO & NSAllowsArbitraryLoadsInWebContent: YES | YES | Webview | iOS9/iOS10 | iOS9 彻底禁用ATS/iOS10只对网页内容禁用 ATS |
| NSAllowsArbitraryLoads: NO & NSAllowsArbitraryLoadsInWebContent: YES | NO&YES | URLSession | iOS9/iOS10 | 彻底禁用ATS |
| NSAllowsArbitraryLoads: NO & NSAllowsArbitraryLoadsInWebContent: NO | NO&NO | Webview/URLSession | iOS9/iOS10 | 彻底禁用ATS |
| NSAllowsArbitraryLoads: YES & NSAllowsArbitraryLoadsInWebContent: NO | YES&NO | Webview | iOS9/iOS10 | iOS9彻底禁用ATS/iOS10会忽略 NSAllowsArbitraryLoads 的设置 |
| NSAllowsArbitraryLoads: YES & NSAllowsArbitraryLoadsInWebContent: NO | YES&NO | URLSession | iOS9/iOS10 | iOS9彻底禁用ATS/iOS10未禁用ATS |
| NSAllowsArbitraryLoads: YES & NSAllowsArbitraryLoadsInWebContent: YES | YES&YES | Webview | iOS9/iOS10 | iOS9彻底禁用ATS/iOS10会忽略 NSAllowsArbitraryLoads 的设置 |
| NSAllowsArbitraryLoads: YES & NSAllowsArbitraryLoadsInWebContent: YES | YES&YES | URLSession | iOS9/iOS10 | 彻底禁用ATS |

### iOS10 也支持对某一个域名禁用或者开启  ATS 

示例代码：

```
NSAppTransportSecurity
    NSExceptionDomains
        "domain-i-control.example.com"
            NSExceptionAllowsInsecureHTTPLoads = NO
            NSExceptionRequiresForwardSecrecy = YES
            NSExceptionMinimumTLSVersion = "TLSv1.2"
        "other-domain-i-control.example.com"
            NSExceptionAllowsInsecureHTTPLoads = NO
            NSExceptionRequiresForwardSecrecy = YES
            NSExceptionMinimumTLSVersion = "TLSv1.2"
    NSAllowsArbitraryLoads = YES

```

更详细的 ATS 配置示例：

```
NSAppTransportSecurity : Dictionary {
    NSAllowsArbitraryLoads : Boolean
    NSAllowsArbitraryLoadsForMedia : Boolean
    NSAllowsArbitraryLoadsInWebContent : Boolean
    NSAllowsLocalNetworking : Boolean
    NSExceptionDomains : Dictionary {
        <domain-name-string> : Dictionary {
            NSIncludesSubdomains : Boolean
            NSExceptionAllowsInsecureHTTPLoads : Boolean
            NSExceptionMinimumTLSVersion : String
            NSExceptionRequiresForwardSecrecy : Boolean   // Default value is YES
            NSRequiresCertificateTransparency : Boolean
        }
    }
}
```

+ NSAppTransportSecurity : 一个用于配置App Transport Security行为的属性，在Info.plist中是于Bundle Identifier同一级别的属性。
+ NSAllowsArbitraryLoads : 一个用于针对不在NSExceptionDomains中的域的配置项。如果设置成YES，则对于那些不在NSExceptionDomains的域则不需要通过ATS的验证。默认值是No，这时对于不在NSExceptionDomains里的域则是需要通过ATS的验证。
+ NSExceptionDomains : 用于配置例外的配置项。
+ < domain-name-for-exception-as-string > : 需要添加例外的域名字符串，如:www.baidu.com
+ NSExceptionMinimumTLSVerion : 用于指定例外域名的TSL的版本号，可用的配置有TLSv1.0、TLSv1.1以及TLSv1.2三个配置项。
+ NSExceptionRequiresForwardSecrecy ： 用于指定所配置的域协议是否使用ATS的所要求的证书加密算法签名，如果是NO， 则加密算法必须是以下这几种。默认值是YES。

```
TLS_RSA_WITH_AES_256_GCM_SHA384
TLS_RSA_WITH_AES_128_GCM_SHA256
TLS_RSA_WITH_AES_256_CBC_SHA256
TLS_RSA_WITH_AES_256_CBC_SHA
TLS_RSA_WITH_AES_128_CBC_SHA256
TLS_RSA_WITH_AES_128_CBC_SHA
```
+ NSExceptionAllowsInsecureHTTPLoads :用于指明所配置的域是否HTTPS的服务器。用这个配置可用访问那些没有证书、自签名证书、过期证书以及证书与域名匹配不上的服务器。默认值是NO。
+ NSIncludesSubdomains : 用于指明子域名是否使用同样的配置。默认值是NO。
+ NSThirdPartyExceptionMinimumTLSVersion : 该变量在域名为第三方域名时，且开发人员无法控制的情况下进行配置
+ NSThirdPartyExceptionRequiresForwardSecrecy : 该变量在域名为第三方域名时，且开发人员无法控制的情况下进行配置。

## iOS 访问 Mixed Content 网页

即便是升级成为了全站 HTTPS ，也有可能出现混合网页的情况，HTTPS 网页中加载的 HTTP 资源被称之为 Mixed Content（混合内容），不同浏览器对 Mixed Content 有不一样的处理规则。

W3C 的 Mixed Content 规范将资源分为了两大类:

| Optionally-blockable | Blockable |
| :------:| :------: |
| 一般指危险性较小即使被中间人篡改也无大碍的资源 例如：图片 | 一般指需要禁止加载的资源 例如： JavaScript、CSS |

### 现代浏览器大多数会默认加载 Optionally-blockable 禁止 Blockable

+ iOS 9 以下的 Safari，默认会加载混合网页内容；


## iOS 中默认的 CSP 策略

### upgrade-insecure-requests

+ 从 iOS 10 开始，iOS 请求头中支持了 upgrade-insecure-requests 字段

为了解决 http to https 的问题，即便全站升级了 https ，但一些遗留代码以及无人维护的代码中还残留着 http 的影子，upgrade-insecure-requests 的出现就是为了解决上述问题，浏览器在发现请求的资源是 http 时，会带上  upgrade-insecure-requests : 1
这样 就可以带不改动代码的情况下，完成 http to https 。

### PS：从测试情况来看，iOS 中的 upgrade-insecure-requests 策略 无法通过禁用 ATS 来关闭，所以建议全站升级 https 避免出现 Mixed Content 的情况，否则会导致资源无法访问

### 使用 HSTS 可以有效的避免 降级劫持


## 参考资料--> 
[https://www.w3.org/TR/upgrade-insecure-requests/#examples](https://www.w3.org/TR/upgrade-insecure-requests/#examples)
[https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html)  
[ATS详解](http://ifungfay.com/uncategorized/ATS详解/)
