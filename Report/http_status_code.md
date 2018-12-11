

# HTTP状态码一览

> HTTP状态码（HTTP Status Code）是用以表示网页服务器HTTP响应状态的3位数字代码。它由 RFC 2616 规范定义的，并得到RFC 2518、RFC 2817、RFC 2295、RFC 2774、RFC 4918等规范扩展。

## 消息

>这一类型的状态码，代表请求已被接受，需要继续处理。这类响应是临时响应，只包含状态行和某些可选的响应头信息，并以空行结束。由于 HTTP/1.0 协议中没有定义任何 1xx 状态码，所以除非在某些试验条件下，服务器禁止向此类客户端发送 1xx 响应。

| 状态码|英文 |中文 |
| :---: | --- | --- |
| 100 |  Continue | 继续 |
| 101 |  Switching Protocols | 切换协议 |
| 102 |  Processing (WebDAV ; RFC 2518 ) |  处理将被继续执行|
| 103 |  Early Hints (RFC 8297 ) |  |

## 成功

> 这一类型的状态码，代表请求已成功被服务器接收、理解、并接受 。

| 状态码|英文 |中文 |
| :---: | --- | --- |
| 200 |  OK | 请求成功 |
| 201 |  Created | 已创建 |
| 202 |  Accepted | 已接受 |
| 203 |  Non-Authoritative Information (since HTTP/1.1) | 非授权信息 |
| 204 |  No Content | 无内容 |
| 205 |  Reset Content | 重置内容 |
| 206 |  Partial Content (RFC 7233 ) | 部分内容 |
| 207 |  Multi-Status (WebDAV; RFC 4918 ) |  |
| 208 |  Already Reported (WebDAV; RFC 5842 ) |  |
| 226 |  IM Used (RFC 3229 ) |  |

## 重定向

> 这类状态码代表需要客户端采取进一步的操作才能完成请求。通常，这些状态码用来重定向，后续的请求地址（重定向目标）在本次响应的 Location 域中指明。

| 状态码|英文 |中文 |
| :---: | --- | --- |
| 300 |  Multiple Choices | 多种选择 |
| 301 |  Moved Permanently | 永久移动 |
| 302 |  Found (Previously "Moved temporarily") | 临时移动 |
| 303 |  See Other (since HTTP/1.1) | 查看其它地址 |
| 304 |  Not Modified (RFC 7232 ) | 未修改 |
| 305 |  Use Proxy (since HTTP/1.1) | 使用代理 |
| 306 |  Switch Proxy | 在最新版的规范中，306状态码已经不再被使用 |
| 307 |  Temporary Redirect (since HTTP/1.1) | 请求的资源临时从不同的URI 响应请求 |
| 308 |  Permanent Redirect (RFC 7538 ) | 永久重定向 |

## 请求错误

> 这类的状态码代表了客户端看起来可能发生了错误，妨碍了服务器的处理。除非响应的是一个 HEAD 请求，否则服务器就应该返回一个解释当前错误状况的实体，以及这是临时的还是永久性的状况。这些状态码适用于任何请求方法。浏览器应当向用户显示任何包含在此类错误响应中的实体内容。

| 状态码|英文 |中文 |
| :---: | --- | --- |
| 401 |  Bad Request | 客户端请求的语法错误，服务器无法理解 |
| 401 |  Unauthorized (RFC 7235 ) | 当前请求需要用户验证|
| 402 |  Payment Required | 该状态码是为了将来可能的需求而预留的 |
| 403 |  Forbidden | 服务器已经理解请求，但是拒绝执行它|
| 404 |  Not Found | 请求失败，请求所希望得到的资源未被在服务器上发现 |
| 405 |  Method Not Allowed | 请求行中指定的请求方法不能被用于请求相应的资源|
| 406 |  Not Acceptable | 服务器无法根据客户端请求的内容特性完成请求 |
| 407 |  Proxy Authentication Required (RFC 7235 ) | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权 |
| 408 |  Request Timeout | 服务器等待客户端发送的请求时间过长，超时 |
| 409 |  Conflict | 服务器完成客户端的PUT请求是可能返回此代码，服务器处理请求时发生了冲突 |
| 410 |  Gone | 客户端请求的资源已经不存在 |
| 411 |  Length Required | 服务器无法处理客户端发送的不带Content-Length的请求信息 |
| 412 |  Precondition Failed (RFC 7232 ) | 客户端请求信息的先决条件错误 |
| 413 |  Payload Too Large (RFC 7231 ) |服务器拒绝处理当前请求，因为该请求提交的实体数据大小超过了服务器愿意或者能够处理的范围  |
| 414 |  URI Too Long (RFC 7231 ) | 请求的URI过长 |
| 415 |  Unsupported Media Type | 服务器无法处理请求附带的媒体格式 |
| 416 |  Range Not Satisfiable (RFC 7233 ) | 客户端请求的范围无效 |
| 417 |  Expectation Failed | 服务器无法满足Expect的请求头信息 |
| 418 |  I'm a teapot (RFC 2324 , RFC 7168 ) |  |
| 421 |  Misdirected Request (RFC 7540 ) |  |
| 422 |  Unprocessable Entity (WebDAV; RFC 4918 ) |  |
| 423 |  Locked (WebDAV; RFC 4918 ) |  |
| 424 |  Failed Dependency (WebDAV; RFC 4918 ) |  |
| 426 |  Upgrade Required |  |
| 428 |  Precondition Required (RFC 6585 ) |  |
| 429 |  Too Many Requests (RFC 6585 ) |  |
| 431 |  Request Header Fields Too Large (RFC 6585 ) |  |
| 451 |  Unavailable For Legal Reasons (RFC 7725 ) |  |

## 服务器错误

> 这类状态码代表了服务器在处理请求的过程中有错误或者异常状态发生，也有可能是服务器意识到以当前的软硬件资源无法完成对请求的处理。除非这是一个HEAD 请求，否则服务器应当包含一个解释当前错误状态以及这个状况是临时的还是永久的解释信息实体。浏览器应当向用户展示任何在当前响应中被包含的实体。

| 状态码|英文 |中文 |
| :---: | --- | --- |
| 500 |  Internal Server Error | 服务器内部错误 |
| 501 |  Not Implemented | 未实现 |
| 502 |  Bad Gateway | 网关错误 |
| 503 |  Service Unavailable | 由于超载或系统维护，服务器暂时的无法处理客户端的请求 |
| 504 |  Gateway Timeout | 充当网关或代理的服务器，未及时从远端服务器获取请求 |
| 505 |  HTTP Version Not Supported | 服务器不支持，或者拒绝支持在请求中使用的 HTTP 版本 |
| 506 |  Variant Also Negotiates (RFC 2295 ) | 由《透明内容协商协议》（RFC 2295）扩展，代表服务器存在内部配置错误 |
| 507 |  Insufficient Storage (WebDAV; RFC 4918 ) | 服务器无法存储完成请求所必须的内容。 |
| 508 |  Loop Detected (WebDAV; RFC 5842 ) |  |
| 510 |  Not Extended (RFC 2774 ) | 获取资源所需要的策略并没有被满足 |
| 511 |  Network Authentication Required (RFC 6585 ) |  |

## 非官方

| 状态码|英文 |中文 |
| :---: | --- | --- |
| 218 |  This is fine (Apache Web Server ) |  |
| 419 |  Page Expired (Laravel Framework ) |  |
| 420 |  Method Failure (Spring Framework ) |  |
| 420 |  Enhance Your Calm (Twitter ) |  |
| 450 |  Blocked by Windows Parental Controls (Microsoft) |  |
| 498 |  Invalid Token (Esri) |  |
| 499 |  Token Required (Esri) |  |
| 509 |  Bandwidth Limit Exceeded (Apache Web Server /cPanel ) | 服务器达到带宽限制 |
| 526 |  Invalid SSL Certificate |  |
| 530 |  Site is frozen |  |
| 598 |  (Informal convention) Network read timeout error |  |
| 449 |  Retry With |  |
| 451 |  Redirect |  |
| 494 |  Request header too large |  |
| 495 |  SSL Certificate Error |  |
| 496 |  SSL Certificate Required |  |
| 497 |  HTTP Request Sent to HTTPS Port |  |
| 499 |  Client Closed Request |  |
| 521 |  Web Server Is Down |  |
| 522 |  Connection Timed Out |  |
| 523 |  Origin Is Unreachable |  |
| 524 |  A Timeout Occurred |  |
| 525 |  SSL Handshake Failed |  |
| 526 |  Invalid SSL Certificate |  |
| 527 |  Railgun Error |  |
| 530 |  Origin DNS Error |  |
## 详细解释
* [HTTP状态码](https://baike.baidu.com/item/HTTP%E7%8A%B6%E6%80%81%E7%A0%81/5053660?fr=aladdin)
* [List of HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)


