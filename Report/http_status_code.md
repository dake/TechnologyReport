# HTTP 状态码一览

> HTTP 状态码（HTTP Status Code）是用以表示网页服务器 HTTP 响应状态的 3 位数字代码。它由  [RFC 2616](https://tools.ietf.org/html/rfc2616) 规范定义的，并得到 [RFC 2518](https://tools.ietf.org/html/rfc2518)、[RFC 2817](https://tools.ietf.org/html/rfc2817)、[RFC 2295](https://tools.ietf.org/html/rfc2295)、[RFC 2774](https://tools.ietf.org/html/rfc2774)、[RFC 4918](https://tools.ietf.org/html/rfc4918) 等规范扩展。

## 一、==消息==

>这一类型的状态码，代表请求已被接受，需要继续处理。这类响应是临时响应，只包含状态行和某些可选的响应头信息，并以空行结束。由于 HTTP/1.0 协议中没有定义任何 1xx 状态码，所以除非在某些试验条件下，服务器禁止向此类客户端发送 1xx 响应。这些状态码代表的响应都是信息性的，标示客户应该采取的其他行动。

### 100 Continue (继续)

The server has received the request headers and the client should proceed to send the request body (in the case of a request for which a body needs to be sent; for example, a POST request). Sending a large request body to a server after a request has been rejected for inappropriate headers would be inefficient. To have a server check the request's headers, a client must send Expect: 100-continue as a header in its initial request and receive a 100 Continue status code in response before sending the body. If the client receives an error code such as 403 (Forbidden) or 405 (Method Not Allowed) then it shouldn't send the request's body. The response 417 Expectation Failed indicates that the request should be repeated without the Expect header as it indicates that the server doesn't support expectations (this is the case, for example, of HTTP/1.0 servers).

服务器已经接收到请求头，并且客户端应继续发送请求主体（在需要发送身体的请求的情况下：例如，POST请求），或者如果请求已经完成，忽略这个响应。服务器必须在请求完成后向客户端发送一个最终响应。要使服务器检查请求的头部，客户端必须在其初始请求中发送 Expect: 100-continue 作为头部，并在发送正文之前接收 100 Continue 状态代码。响应代码 417 期望失败表示请求不应继续。

### 101 Switching Protocols (切换协议)

The requester has asked the server to switch protocols and the server has agreed to do so.

服务器已经理解了客户端的请求，并将通过 Upgrade 消息头通知客户端采用不同的协议来完成这个请求。在发送完这个响应最后的空行后，服务器将会切换到在 Upgrade 消息头中定义的那些协议。
只有在切换新的协议更有好处的时候才应该采取类似措施。例如，切换到新的HTTP版本（如 HTTP/2 ）比旧版本更有优势，或者切换到一个实时且同步的协议（如 WebSocket ）以传送利用此类特性的资源。

### 102 Processing (WebDAV ; RFC 2518 ) (正在处理)

A WebDAV request may contain many sub-requests involving file operations, requiring a long time to complete the request. This code indicates that the server has received and is processing the request, but no response is available yet. This prevents the client from timing out and assuming the request was lost.

WebDAV 请求可能包含许多涉及文件操作的子请求，需要很长时间才能完成请求。该代码表示​​服务器已经收到并正在处理请求，但无响应可用。这样可以防止客户端超时，并假设请求丢失。

### 103 Early Hints (RFC 8297 )

Used to return some response headers before final HTTP message.

## 二、==成功==

> 这一类型的状态码，代表请求已成功被服务器接收、理解、并接受 。

### 200 OK (成功)

Standard response for successful HTTP requests. The actual response will depend on the request method used. In a GET request, the response will contain an entity corresponding to the requested resource. In a POST request, the response will contain an entity describing or containing the result of the action.

请求已成功，请求所希望的响应头或数据体将随此响应返回。实际的响应将取决于所使用的请求方法。在 GET 请求中，响应将包含与请求的资源相对应的实体。在 POST 请求中，响应将包含描述或操作结果的实体。

### 201 Created (已创建)

The request has been fulfilled, resulting in the creation of a new resource.

请求已经被实现，而且有一个新的资源已经依据请求的需要而建立，且其 URI 已经随 Location 头信息返回。假如需要的资源无法及时创建的话，应当返回 '202 Accepted'。

### 202 Accepted (已接受)

The request has been accepted for processing, but the processing has not been completed. The request might or might not be eventually acted upon, and may be disallowed when processing occurs.

服务器已接受请求，但尚未处理。最终该请求可能会也可能不会被执行，并且可能在处理发生时被禁止。

### 203 Non-Authoritative Information (since HTTP/1.1) (非授权信息)

The server is a transforming proxy (e.g. a Web accelerator ) that received a 200 OK from its origin, but is returning a modified version of the origin's response.

服务器是一个转换代理服务器（transforming proxy，例如网络加速器 ），以 200 OK 状态码为起源，但回应了原始响应的修改版本。

### 204 No Content (无内容)

The server successfully processed the request and is not returning any content.

服务器成功处理了请求，没有返回任何内容。

### 205 Reset Content (重置内容)

The server successfully processed the request, but is not returning any content. Unlike a 204 response, this response requires that the requester reset the document view.

服务器成功处理了请求，但没有返回任何内容。与 204 响应不同，此响应要求请求者重置文档视图。

### 206 Partial Content (RFC 7233 ) (部分内容)

The server is delivering only part of the resource (byte serving ) due to a range header sent by the client. The range header is used by HTTP clients to enable resuming of interrupted downloads, or split a download into multiple simultaneous streams.

服务器已经成功处理了部分 GET 请求。类似于 FlashGet 或者迅雷 这类的 HTTP 下载工具 都是使用此类响应实现断点续传或者将一个大文档分解为多个下载段同时下载。

### 207 Multi-Status (WebDAV; RFC 4918 )

The message body that follows is by default an XML message and can contain a number of separate response codes, depending on how many sub-requests were made.

代表之后的消息体将是一个 XML 消息，并且可能依照之前子请求数量的不同，包含一系列独立的响应代码。

### 208 Already Reported (WebDAV; RFC 5842 )

The members of a DAV binding have already been enumerated in a preceding part of the (multistatus) response, and are not being included again.

DAV 绑定的成员已经在（多状态）响应之前的部分被列举，且未被再次包含。

### 226 IM Used (RFC 3229 )

The server has fulfilled a request for the resource, and the response is a representation of the result of one or more instance-manipulations applied to the current instance.

服务器已经满足了对资源的请求，对实体请求的一个或多个实体操作的结果表示。

## 三、==重定向==

> 这类状态码代表需要客户端采取进一步的操作才能完成请求。通常，这些状态码用来重定向，后续的请求地址（重定向目标）在本次响应的 Location 域中指明。
> 当且仅当后续的请求所使用的方法是 GET 或者 HEAD 时，用户浏览器才可以在没有用户介入的情况下自动提交所需要的后续请求。客户端应当自动监测无限循环重定向（例如：A→B→C→……→A或A→A），因为这会导致服务器和客户端大量不必要的资源消耗。按照 HTTP/1.0 版规范的建议，浏览器不应自动访问超过 5 次的重定向。

### 300 Multiple Choices (多种选择)

Indicates multiple options for the resource from which the client may choose (via agent-driven content negotiation). For example, this code could be used to present multiple video format options, to list files with different filename extensions , or to suggest word-sense disambiguation .

被请求的资源有一系列可供选择的回馈信息，每个都有自己特定的地址和浏览器驱动的商议信息。用户或浏览器能够自行选择一个首选的地址进行重定向。
除非这是一个 HEAD 请求，否则该响应应当包括一个资源特性及地址的列表的实体，以便用户或浏览器从中选择最合适的重定向地址。这个实体的格式由 Content-Type 定义的格式所决定。浏览器可能根据响应的格式以及浏览器自身能力，自动作出最合适的选择。当然，RFC 2616 规范并没有规定这样的自动选择该如何进行。
如果服务器本身已经有了首选的回馈选择，那么在 Location 中应当指明这个回馈的 URI ；浏览器可能会将这个 Location 值作为自动重定向的地址。此外，除非额外指定，否则这个响应也是可缓存的。

### 301 Moved Permanently (永久移动)

This and all future requests should be directed to the given URI .

被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个 URI 之一。如果可能，拥有链接编辑功能的客户端应当自动把请求的地址修改为从服务器反馈回来的地址。除非额外指定，否则这个响应也是可缓存的。
新的永久性的URI应当在响应的 Location 域中返回。除非这是一个 HEAD 请求，否则响应的实体中应当包含指向新的 URI 的超链接 及简短说明。
如果这不是一个 GET 或者 HEAD 请求，那么浏览器禁止自动进行重定向，除非得到用户的确认，因为请求的条件可能因此发生变化。
注意：对于某些使用 HTTP/1.0 协议的浏览器，当它们发送的 POST 请求得到了一个 301 响应的话，接下来的重定向请求将会变成 GET 方式。

### 302 Found (Previously "Moved temporarily")

Tells the client to look at (browse to) another url. 302 has been superseded by 303 and 307. This is an example of industry practice contradicting the standard. The HTTP/1.0 specification (RFC 1945) required the client to perform a temporary redirect (the original describing phrase was "Moved Temporarily"), but popular browsers implemented 302 with the functionality of a 303 See Other. Therefore, HTTP/1.1 added status codes 303 and 307 to distinguish between the two behaviours. However, some Web applications and frameworks use the 302 status code as if it were the 303.

要求客户端执行临时重定向（原始描述短语为 “Moved Temporarily”）。由于这样的重定向是临时的，客户端应当继续向原有地址发送以后的请求。只有在 Cache-Control 或 Expires 中进行了指定的情况下，这个响应才是可缓存的。
新的临时性的 URI 应当在响应的 Location 域中返回。除非这是一个 HEAD 请求，否则响应的实体中应当包含指向新的 URI 的超链接及简短说明。
如果这不是一个 GET 或者 HEAD 请求，那么浏览器禁止自动进行重定向，除非得到用户的确认，因为请求的条件可能因此发生变化。
注意：虽然 RFC 1945 和 RFC 2068 规范不允许客户端在重定向时改变请求的方法，但是很多现存的浏览器将 302 响应视作为 303 响应，并且使用 GET 方式访问在 Location 中规定的 URI，而无视原先请求的方法。因此状态码 303 和 307 被添加了进来，用以明确服务器期待客户端进行何种反应。

### 303 See Other (since HTTP/1.1) (查看其他位置)

The response to the request can be found under another URI using the GET method. When received in response to a POST (or PUT/DELETE), the client should presume that the server has received the data and should issue a new GET request to the given URI.

对应当前请求的响应可以在另一个 URI 上被找到，当响应于 POST（或 PUT / DELETE ）接收到响应时，客户端应该假定服务器已经收到数据，并且应该使用单独的 GET 消息发出重定向。这个方法的存在主要是为了允许由脚本激活的 POST 请求输出重定向到一个新的资源。这个新的 URI 不是原始资源的替代引用。同时，303 响应禁止被缓存。当然，第二个请求（重定向）可能被缓存。
新的 URI 应当在响应的 Location 域中返回。除非这是一个 HEAD 请求，否则响应的实体中应当包含指向新的 URI 的超链接及简短说明。
注意：许多 HTTP/1.1 版以前的浏览器不能正确理解 303 状态。如果需要考虑与这些浏览器之间的互动，302 状态码应该可以胜任，因为大多数的浏览器处理 302 响应时的方式恰恰就是上述规范要求客户端处理 303 响应时应当做的。

### 304 Not Modified (RFC 7232 ) (未修改)

Indicates that the resource has not been modified since the version specified by the request headers If-Modified-Since or If-None-Match. In such case, there is no need to retransmit the resource since the client still has a previously-downloaded copy.

表示资源在由请求头中的 If-Modified-Since 或 If-None-Match 参数指定的这一版本之后，未曾被修改。在这种情况下，由于客户端仍然具有以前下载的副本，因此不需要重新传输资源。

### 305 Use Proxy (since HTTP/1.1) (使用代理)

The requested resource is available only through a proxy, the address for which is provided in the response. Many HTTP clients (such as Mozilla  and Internet Explorer ) do not correctly handle responses with this status code, primarily for security reasons.

被请求的资源必须通过指定的代理才能被访问。Location 域中将给出指定的代理所在的 URI 信息，接收者需要重复发送一个单独的请求，通过这个代理才能访问相应资源。只有原始服务器才能创建 305 响应。许多 HTTP 客户端（像是 Mozilla  和Internet Explorer  ）都没有正确处理这种状态代码的响应，主要是出于安全考虑。
注意：RFC 2068 中没有明确 305 响应是为了重定向一个单独的请求，而且只能被原始服务器建立。忽视这些限制可能导致严重的安全后果。

### 306 Switch Proxy (切换代理)

No longer used. Originally meant "Subsequent requests should use the specified proxy."

在最新版的规范中，306 状态码已经不再被使用。最初是指“后续请求应使用指定的代理”。

### 307 Temporary Redirect (since HTTP/1.1) (临时重定向)

In this case, the request should be repeated with another URI; however, future requests should still use the original URI. In contrast to how 302 was historically implemented, the request method is not allowed to be changed when reissuing the original request. For example, a POST request should be repeated using another POST request.

在这种情况下，请求应该与另一个 URI 重复，但后续的请求应仍使用原始的 URI。 与 302 相反，当重新发出原始请求时，不允许更改请求方法。 例如，应该使用另一个 POST 请求来重复 POST 请求。

### 308 Permanent Redirect (RFC 7538 )

The request and all future requests should be repeated using another URI. 307 and 308 parallel the behaviors of 302 and 301, but do not allow the HTTP method to change. So, for example, submitting a form to a permanently redirected resource may continue smoothly.

请求和所有将来的请求应该使用另一个 URI  重复。 307 和 308 重复 302 和 301 的行为，但不允许 HTTP 方法更改。 例如，将表单提交给永久重定向的资源可能会顺利进行。

## 四、==请求错误==

> 这类的状态码代表了客户端看起来可能发生了错误，妨碍了服务器的处理。除非响应的是一个 HEAD 请求，否则服务器就应该返回一个解释当前错误状况的实体，以及这是临时的还是永久性的状况。这些状态码适用于任何请求方法。浏览器应当向用户显示任何包含在此类错误响应中的实体内容。
> 如果错误发生时客户端正在传送数据，那么使用 TCP 的服务器实现应当仔细确保在关闭客户端与服务器之间的连接之前，客户端已经收到了包含错误信息的数据包。如果客户端在收到错误信息后继续向服务器发送数据，服务器的 TCP 栈将向客户端发送一个重置数据包，以清除该客户端所有还未识别的输入缓冲，以免这些数据被服务器上的应用程序读取并干扰后者

### 400 Bad Request (错误请求)

The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

由于明显的客户端错误（例如，格式错误的请求语法，太大的大小，无效的请求消息或欺骗性路由请求），服务器不能或不会处理该请求。

### 401 Unauthorized (RFC 7235 ) (未授权)

Similar to 403 Forbidden, but specifically for use when authentication is required and has failed or has not yet been provided. The response must include a WWW-Authenticate header field containing a challenge applicable to the requested resource. See Basic access authentication and Digest access authentication . 401 semantically means "unauthenticated" , i.e. the user does not have the necessary credentials.
Note: Some sites incorrectly issue HTTP 401 when an IP address is banned from the website (usually the website domain) and that specific address is refused permission to access a website.

类似于 403 Forbidden ，401 语义即“未认证 ”，即用户没有必要的凭据。该状态码表示当前请求需要用户验证。该响应必须包含一个适用于被请求资源的 WWW-Authenticate 信息头用以询问用户信息。客户端可以重复提交一个包含恰当的 Authorization 头信息的请求。如果当前请求已经包含了 Authorization 证书，那么 401 响应代表着服务器验证已经拒绝了那些证书。如果 401 响应包含了与前一个响应相同的身份验证询问，且浏览器已经至少尝试了一次验证，那么浏览器应当向用户展示响应中包含的实体信息，因为这个实体信息中可能包含了相关诊断信息。
注意：当网站（通常是网站域名）禁止 IP 地址时，有些网站状态码显示的 401 ，表示该特定地址被拒绝访问网站。

### 402 Payment Required (预留)

Reserved for future use. The original intention was that this code might be used as part of some form of digital cash or micropayment scheme, as proposed for example by GNU Taler, but that has not yet happened, and this code is not usually used. Google Developers API uses this status if a particular developer has exceeded the daily limit on requests. Sipgate uses this code if an account does not have sufficient funds to start a call. Shopify uses this code when the store has not paid their fees and is temporarily disabled. 

该状态码是为了将来可能的需求而预留的。该状态码最初的意图可能被用作某种形式的数字现金或在线支付方案的一部分，但几乎没有哪家服务商使用，而且这个状态码通常不被使用。如果特定开发人员已超过请求的每日限制，Google Developers API 会使用此状态码。

### 403 Forbidden (禁止)

The request was valid, but the server is refusing action. The user might not have the necessary permissions for a resource, or may need an account of some sort.

服务器已经理解请求，但是拒绝执行它。与 401 响应不同的是，身份验证并不能提供任何帮助，而且这个请求也不应该被重复提交。如果这不是一个 HEAD 请求，而且服务器希望能够讲清楚为何请求不能被执行，那么就应该在实体内描述拒绝的原因。当然服务器也可以返回一个 404 响应，假如它不希望让客户端获得任何信息。

### 404 Not Found (未找到)

The requested resource could not be found but may be available in the future. Subsequent requests by the client are permissible.

请求失败，请求所希望得到的资源未被在服务器上发现，但允许用户的后续请求。没有信息能够告诉用户这个状况到底是暂时的还是永久的。假如服务器知道情况的话，应当使用 410 状态码来告知旧资源因为某些内部的配置机制问题，已经永久的不可用，而且没有任何可以跳转的地址。404 这个状态码被广泛应用于当服务器不想揭示到底为何请求被拒绝或者没有其他适合的响应可用的情况下。

### 405 Method Not Allowed (方法禁止)

A request method is not supported for the requested resource; for example, a GET request on a form that requires data to be presented via POST , or a PUT request on a read-only resource.

请求行中指定的请求方法不能被用于请求相应的资源。该响应必须返回一个 Allow 头信息用以表示出当前资源能够接受的请求方法的列表。例如，需要通过  POST  呈现数据的表单上的 GET 请求，或只读资源上的 PUT 请求。
鉴于 PUT ，DELETE 方法会对服务器上的资源进行写操作，因而绝大部分的网页服务器 都不支持或者在默认配置下不允许上述请求方法，对于此类请求均会返回 405 错误。

### 406 Not Acceptable (不接受)

The requested resource is capable of generating only content not acceptable according to the Accept headers sent in the request. See Content negotiation .

请求的资源的内容特性无法满足请求头中的条件，因而无法生成响应实体，该请求不可接受。
除非这是一个 HEAD 请求，否则该响应就应当返回一个包含可以让用户或者浏览器从中选择最合适的实体特性以及地址栏表的实体。实体的格式由 Content-Type 头中定义的媒体类型决定。浏览器可以根据格式及自身能力自行作出最佳选择。但是，规范中并没有定义任何作出此类自动选择的标准。

### 407 Proxy Authentication Required (RFC 7235 ) (需要代理授权)

The client must first authenticate itself with the proxy .

与 401 响应类似，只不过客户端必须在代理服务器上进行身份验证。代理服务器必须返回一个 Proxy-Authenticate 用以进行身份询问。客户端可以返回一个 Proxy-Authorization 信息头用以验证。

### 408 Request Timeout (请求超时)

The server timed out waiting for the request. According to HTTP specifications: "The client did not produce a request within the time that the server was prepared to wait. The client MAY repeat the request without modifications at any later time."

请求超时。根据 HTTP 规范，客户端没有在服务器预备等待的时间内完成一个请求的发送，客户端可以随时再次提交这一请求而无需进行任何更改。

### 409 Conflict (冲突)

Indicates that the request could not be processed because of conflict in the current state of the resource, such as an edit conflict between multiple simultaneous updates.

表示因为请求存在冲突无法处理该请求，例如多个同步更新之间的编辑冲突 。

### 410 Gone (已删除)

Indicates that the resource requested is no longer available and will not be available again. This should be used when a resource has been intentionally removed and the resource should be purged. Upon receiving a 410 status code, the client should not request the resource in the future. Clients such as search engines should remove the resource from their indices. Most use cases do not require clients and search engines to purge the resource, and a "404 Not Found" may be used instead.

表示所请求的资源不再可用，将不再可用。当资源被有意地删除并且资源应被清除时，应该使用这个。在收到 410 状态码后，用户应停止再次请求资源。但大多数服务端不会使用此状态码，而是直接使用 404 状态码。

### 411 Length Required (需要有效长度)

The request did not specify the length of its content, which is required by the requested resource.

服务器拒绝在没有定义 Content-Length 头的情况下接受请求。在添加了表明请求消息体长度的有效 Content-Length 头之后，客户端可以再次提交该请求。

### 412 Precondition Failed (RFC 7232 ) (未满足前提条件)

The server does not meet one of the preconditions that the requester put on the request.

服务器在验证在请求的头字段中给出先决条件时，没能满足其中的一个或多个。这个状态码允许客户端在获取资源时在请求的元信息（请求头字段数据）中设置先决条件，以此避免该请求方法被应用到其希望的内容以外的资源上。

### 413 Payload Too Large (RFC 7231 ) (请求实体太大)

The request is larger than the server is willing or able to process. Previously called "Request Entity Too Large".

前称 “Request Entity Too Large”，表示服务器拒绝处理当前请求，因为该请求提交的实体数据大小超过了服务器愿意或者能够处理的范围。此种情况下，服务器可以关闭连接以免客户端继续发送此请求。
如果这个状况是临时的，服务器应当返回一个 Retry-After 的响应头，以告知客户端可以在多少时间以后重新尝试。

### 414 URI Too Long (RFC 7231 ) (请求的 URI 过长)

The URI provided was too long for the server to process. Often the result of too much data being encoded as a query-string of a GET request, in which case it should be converted to a POST request. Called "Request-URI Too Long" previously.

表示请求的 URI 长度超过了服务器能够解释的长度，因此服务器拒绝对该请求提供服务。通常将太多数据的结果编码为 GET 请求的查询字符串，在这种情况下，应将其转换为 POST 请求。这比较少见，通常的情况包括：
本应使用 POST 方法的表单提交变成了 GET 方法，导致查询字符串 过长。
重定向 URI “黑洞”，例如每次重定向把旧的 URI 作为新的 URI 的一部分，导致在若干次重定向后 URI 超长。
客户端正在尝试利用某些服务器中存在的安全漏洞 攻击服务器。这类服务器使用固定长度的缓冲读取或操作请求的 URI ，当 GET 后的参数超过某个数值后，可能会产生缓冲区溢出，导致任意代码被执行。没有此类漏洞的服务器，应当返回 414 状态码。

### 415 Unsupported Media Type (不支持的媒体类型)

The request entity has a media type which the server or resource does not support. For example, the client uploads an image as image/svg+xml , but the server requires that images use a different format.

对于当前请求的方法和所请求的资源，请求中提交的互联网媒体类型 并不是服务器中所支持的格式，因此请求被拒绝。例如，客户端将图像上传格式为 svg，但服务器要求图像使用上传格式为 jpg。

### 416 Range Not Satisfiable (RFC 7233 ) (请求范围不符合要求)

The client has asked for a portion of the file (byte serving ), but the server cannot supply that portion. For example, if the client asked for a part of the file that lies beyond the end of the file. Called "Requested Range Not Satisfiable" previously.

前称 “Requested Range Not Satisfiable”。客户端已经要求文件的一部分（Byte serving ），但服务器不能提供该部分。例如，如果客户端要求文件的一部分超出文件尾端。

### 417 Expectation Failed (未满足期望值)

The server cannot meet the requirements of the Expect request-header field.

在请求头 Expect 中指定的预期内容无法被服务器满足，或者这个服务器是一个代理服显的证据证明在当前路由 的下一个节点上，Expect 的内容无法被满足。

### 418 I'm a teapot (RFC 2324 , RFC 7168 )

This code was defined in 1998 as one of the traditional IETF April Fools' jokes , in RFC 2324 , Hyper Text Coffee Pot Control Protocol , and is not expected to be implemented by actual HTTP servers. The RFC specifies this code should be returned by teapots requested to brew coffee. This HTTP status is used as an Easter egg in some websites, including Google.com .

本操作码是在 1998 年作为 IETF  的传统愚人节笑话 , 在 RFC 2324 超文本咖啡壶控制协议 '中定义的，并不需要在真实的 HTTP 服务器中定义。当一个控制茶壶的 HTCPCP  收到 BREW 或 POST 指令要求其煮咖啡时应当回传此错误。这个 HTTP 状态码在某些网站（包括 Google.com ）与项目（如 Node.js  、ASP.NET  和 Go 语言 ）中用作彩蛋 。

### 421 Misdirected Request (RFC 7540 )

The request was directed at a server that is not able to produce a response (for example because of connection reuse).

该请求针对的是无法产生响应的服务器（例如因为连接重用）。

### 422 Unprocessable Entity (WebDAV; RFC 4918 )

The request was well-formed but was unable to be followed due to semantic errors.

请求格式正确，但是由于含有语义 错误，无法响应。

### 423 Locked (WebDAV; RFC 4918 )

The resource that is being accessed is locked.

当前资源被锁定。

### 424 Failed Dependency (WebDAV; RFC 4918 )

The request failed because it depended on another request and that request failed (e.g., a PROPPATCH).

由于之前的某个请求发生的错误，导致当前请求失败，例如 PROPPATCH。

### 425 Unordered Collection
Defined in drafts of “WebDAV Advanced Collections Protocol”, but not present in “Web Distributed Authoring and Versioning (WebDAV) Ordered Collections Protocol”

在 WebDAV Advanced Collections Protocol 中定义，但 Web Distributed Authoring and Versioning (WebDAV) Ordered Collections Protocol 中并不存在。

### 426 Upgrade Required

The client should switch to a different protocol such as TLS/1.0 , given in the Upgrade header field.

客户端应当切换到 TLS/1.0 ，并在 HTTP/1.1 Upgrade header 中给出。

### 428 Precondition Required (RFC 6585 )

The origin server requires the request to be conditional. Intended to prevent the 'lost update' problem, where a client GETs a resource's state, modifies it, and PUTs it back to the server, when meanwhile a third party has modified the state on the server, leading to a conflict."

原服务器要求该请求满足一定条件。这是为了防止“‘未更新’问题，即客户端读取（GET）一个资源的状态，更改它，并将它写（PUT）回服务器，但这期间第三方已经在服务器上更改了该资源的状态，因此导致了冲突。”

### 429 Too Many Requests (RFC 6585 )

The user has sent too many requests in a given amount of time. Intended for use with rate-limiting schemes.

用户在给定的时间内发送了太多的请求。旨在用于网络限速 。

### 431 Request Header Fields Too Large (RFC 6585 )

The server is unwilling to process the request because either an individual header field, or all the header fields collectively, are too large.

服务器不愿处理请求，因为一个或多个头字段过大。

### 451 Unavailable For Legal Reasons (RFC 7725 )

A server operator has received a legal demand to deny access to a resource or to a set of resources that includes the requested resource. The code 451 was chosen as a reference to the novel Fahrenheit 451 (see the Acknowledgements in the RFC).

该访问因法律 的要求而被拒绝，由 IETF 在 2015 核准后新增加。

## 五、==服务器错误==

> 表示服务器无法完成明显有效的请求。这类状态码代表了服务器在处理请求的过程中有错误或者异常状态发生，也有可能是服务器意识到以当前的软硬件资源无法完成对请求的处理。除非这是一个  HEAD 请求，否则服务器应当包含一个解释当前错误状态以及这个状况是临时的还是永久的解释信息实体。浏览器应当向用户展示任何在当前响应中被包含的实体。这些状态码适用于任何响应方法。

### 500 Internal Server Error (服务器内部错误)

A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

通用错误消息，服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。没有给出具体错误信息。

### 501 Not Implemented (尚未实施)

The server either does not recognize the request method, or it lacks the ability to fulfil the request. Usually this implies future availability (e.g., a new feature of a web-service API).

服务器不支持当前请求所需要的某个功能。当服务器无法识别请求的方法，并且无法支持其对任何资源的请求。（例如，网络服务 API 的新功能）

### 502 Bad Gateway (错误网关)

The server was acting as a gateway or proxy and received an invalid response from the upstream server.

作为网关 或者代理 工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。

### 503 Service Unavailable (服务不可用)

The server is currently unavailable (because it is overloaded or down for maintenance). Generally, this is a temporary state.

由于临时的服务器维护或者过载 ，服务器当前无法处理请求。这个状况是暂时的，并且将在一段时间以后恢复。如果能够预计延迟时间，那么响应中可以包含一个 Retry-After 头用以标明这个延迟时间。如果没有给出这个 Retry-After 信息，那么客户端应当以处理 500 响应的方式处理它。

### 504 Gateway Timeout (网关超时)

The server was acting as a gateway or proxy and did not receive a timely response from the upstream server.

作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器（URI 标识出的服务器，例如HTTP 、FTP 、LDAP ）或者辅助服务器（例如 DNS ）收到响应。
注意：某些代理服务器在 DNS 查询超时 时会返回 400 或者 500 错误。

### 505 HTTP Version Not Supported (HTTP 版本不受支持)

The server does not support the HTTP protocol version used in the request.

服务器不支持，或者拒绝支持在请求中使用的 HTTP 版本。这暗示着服务器不能或不愿使用与客户端相同的版本。响应中应当包含一个描述了为何版本不被支持以及服务器支持哪些协议的实体。

### 506 Variant Also Negotiates (RFC 2295 )

Transparent content negotiation for the request results in a circular reference .

由《透明内容协商协议》（RFC 2295 ）扩展，代表服务器存在内部配置错误，被请求的协商变元资源被配置为在透明内容协商中使用自己，因此在一个协商处理中不是一个合适的重点。

### 507 Insufficient Storage (WebDAV; RFC 4918 )

The server is unable to store the representation needed to complete the request.

服务器无法存储完成请求所必须的内容。这个状况被认为是临时的。

### 508 Loop Detected (WebDAV; RFC 5842 )

The server detected an infinite loop while processing the request (sent in lieu of 208 Already Reported).

服务器在处理请求时陷入死循环。 （可代替 208 状态码）

### 510 Not Extended (RFC 2774 )

Further extensions to the request are required for the server to fulfil it.

获取资源所需要的策略并没有被满足。

### 511 Network Authentication Required (RFC 6585 )

The client needs to authenticate to gain network access. Intended for use by intercepting proxies used to control access to the network (e.g., "captive portals " used to require agreement to Terms of Service before granting full Internet access via a Wi-Fi hotspot ).

客户端需要进行身份验证才能获得网络访问权限，旨在限制用户群访问特定网络。（例如连接WiFi热点 时的强制网络门户 ）

## 六、 ==非官方状态码==

### 103 Checkpoint

Used in the resumable requests proposal to resume aborted PUT or POST requests.

### 218 This is fine (Apache Web Server )

Used as a catch-all error condition for allowing response bodies to flow through Apache when ProxyErrorOverride is enabled. When ProxyErrorOverride is enabled in Apache, response bodies that contain a status code of 4xx or 5xx are automatically discarded by Apache in favor of a generic response or a custom response specified by the ErrorDocument directive.

### 419 Page Expired (Laravel Framework )

Used by the Laravel Framework when a CSRF Token is missing or expired.

### 420 Method Failure (Spring Framework )

A deprecated response used by the Spring Framework when a method has failed.

### 420 Enhance Your Calm (Twitter )

Returned by version 1 of the Twitter Search and Trends API when the client is being rate limited; versions 1.1 and later use the 429 Too Many Requests response code instead.

Twitter Search 与 Trends API 在客户端被限速的情况下返回。

### 450 Blocked by Windows Parental Controls (Microsoft)

The Microsoft extension code indicated when Windows Parental Controls are turned on and are blocking access to the requested webpage.

这是一个由 Windows 家庭控制（Microsoft）HTTP 阻止的 450 状态代码的示例，用于信息和测试。

### 498 Invalid Token (Esri)

Returned by ArcGIS for Server . Code 498 indicates an expired or otherwise invalid token.

### 499 Token Required (Esri)

Returned by ArcGIS for Server . Code 499 indicates that a token is required but was not submitted.

### 509 Bandwidth Limit Exceeded (Apache Web Server /cPanel )

The server has exceeded the bandwidth specified by the server administrator; this is often used by shared hosting providers to limit the bandwidth of customers.

### 526 Invalid SSL Certificate

Used by Cloudflare and Cloud Foundry 's gorouter to indicate failure to validate the SSL/TLS certificate that the origin server presented.

### 530 Site is frozen

Used by the Pantheon web platform to indicate a site that has been frozen due to inactivity.

### 598 (Informal convention) Network read timeout error

Used by some HTTP proxies to signal a network read timeout behind the proxy to a client in front of the proxy.

### ==Internet Information Services==  

### 440 Login Time-out

The client's session has expired and must log in again.

### 449 Retry With

The server cannot honour the request because the user has not provided the required information.

### 451 Redirect

Used in Exchange ActiveSync when either a more efficient server is available or the server cannot access the users' mailbox. The client is expected to re-run the HTTP AutoDiscover operation to find a more appropriate server.

### ==nginx==

### 444 No Response

Used internally to instruct the server to return no information to the client and close the connection immediately.

Nginx上HTTP服务器扩展。服务器不向客户端返回任何信息，并关闭连接（有助于阻止恶意软件）。

### 494 Request header too large

Client sent too large request or too long header line.

在错误代码 431 提出之前 Nginx上使用的扩展 HTTP 代码。

### 495 SSL Certificate Error

An expansion of the 400 Bad Request response code, used when the client has provided an invalid client certificate .

### 496 SSL Certificate Required

An expansion of the 400 Bad Request response code, used when a client certificate is required but not provided.

### 497 HTTP Request Sent to HTTPS Port

An expansion of the 400 Bad Request response code, used when the client has made a HTTP request to a port listening for HTTPS requests.

### 499 Client Closed Request

Used when the client has closed the request before the server could send a response.

这两个是以前一个叫做 ArcGIS for Server 的系统会回应的 Status Code。一般来说验证信息错误还是会回传 401 Unathorized。

### ==Cloudflare==

### 520 Unknown Error

The 520 error is used as a "catch-all response for when the origin server returns something unexpected", listing connection resets, large headers, and empty or invalid responses as common triggers.

Cloudflare 会用的未知错误。

### 521 Web Server Is Down

The origin server has refused the connection from Cloudflare.

指目标服务器挂了

### 522 Connection Timed Out

Cloudflare could not negotiate a TCP handshake with the origin server.

### 523 Origin Is Unreachable

Cloudflare could not reach the origin server; for example, if the DNS records for the origin server are incorrect.

### 524 A Timeout Occurred

Cloudflare was able to complete a TCP connection to the origin server, but did not receive a timely HTTP response.

### 525 SSL Handshake Failed

Cloudflare could not negotiate a SSL/TLS handshake with the origin server.

### 526 Invalid SSL Certificate

Cloudflare could not validate the SSL certificate on the origin web server.

### 527 Railgun Error

Error 527 indicates that the request timed out or failed after the WAN connection had been established.

### 530 Origin DNS Error

Error 530 indicates that the requested host name could not be resolved on the Cloudflare network to an origin server.

## 相关链接
1. [HTTP Status Codes](http://networksmart.wikidot.com/status-code-http)
2. [List of HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
3. [HTTP STATUS DOGS](https://httpstatusdogs.com/)

