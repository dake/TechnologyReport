# iOS WebView 的 层次关系

[研究 wkwebview 的子 view 和 <body> 的关系](https://weibo.com/3321824014/H3vJukUdK )

接着这个话题，总结一下 iOS 中 关于 CSP 策略的一些知识点

# iOS WebView CSP

## iOS ATS

### iOS9 以后 默认情况下非 HTTPS 的网络访问是被禁止的 ，考虑到兼容性 ，可以通过以下三个 KEY 来禁用 ATS 

+ NSAllowsArbitraryLoadsForMedia
+ NSAllowsArbitraryLoadsInWebContent
+ NSAllowsLocalNetworking

### PS：在 iOS9 上这三个 KEY 不生效 ， iOS10 入如果设置了上述 KEY 则会忽略 NSAllowsArbitraryLoads

| KEY | 值 | 组件 | 系统版本 | 结果 |
| :------:| :------: | :------: | :------: | :------: | 
| NSAllowsArbitraryLoads | YES | Webview | iOS9/iOS10 | 彻底禁用 ATS |
| NSAllowsArbitraryLoads | YES | URLSession | iOS9/iOS10 | 彻底禁用 ATS |
| NSAllowsArbitraryLoads | NO | Webview| iOS9/iOS10 | 未禁用 ATS |
| NSAllowsArbitraryLoads | NO | URLSession | iOS9/iOS10 | 未禁用 ATS |
| NSAllowsArbitraryLoads: NO & NSAllowsArbitraryLoadsInWebContent: YES | YES | Webview | iOS9/iOS10 | iOS9 彻底禁用ATS/iOS9以上只对网页内容禁用 ATS |
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