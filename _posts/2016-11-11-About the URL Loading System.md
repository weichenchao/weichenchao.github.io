---
layout: post
title: About the URL Loading System
tags: 
- WebP


---

# 关于URL加载系统

本指南介绍了这个框架可用于URL交互，并使用标准的互联网协议与服务器进行通信。这些类被总称为*URL Loading System*。 

*URL Loading System*是一套*class*和*protocol*的集合，使您的应用程序通过URL访问内容。此技术的核心是`NSURL`类，它可以让应用程序处理指定URL和resource。

为了支持`NSURL`类，Foundation框架提供一系列丰富的类，让你加载URL的内容，上传数据到服务器，管理cookie存储，控制响应缓存，处理证书存储和应用程序特定的身份验证方式，并自定义协议的扩展。

   *URL Loading System*提供了以下协议进行访问资源的支持：

- 文件传输协议（`ftp://`）
- 超文本传输协议（`http://`）
- 加密超文本传输协议（`https://`）
- 本地文件URL（ `file:///`）
- 数据的网址（`data://`）

使用用户的系统偏好，还透明地支持代理服务器和SOCKS网关。

> **重要提示：**  除了*URL Loading System*，在其他应用程序，Mac OS X和iOS也提供API打开URL，如Safari。这些API并未在本文档中描述。
>
> 有关在OS X启动服务的更多信息，请阅读 *Launch Services Programming Guide*。
>
> 在OS X类，有关`NSWorkSpace`的`openURL:`方法的详细信息，请阅读 *NSWorkspace Class Reference*。
>
> 在iOS中类，有关`UIApplication`的`openURL:`方法的详细信息，请阅读*UIApplication Class Reference*。

## 总览

*URL Loading System*包含了一些URL加载的重要辅助类。主要辅助类分为五种：协议支持，认证和证书，cookie存储，配置管理和缓存管理。

![该网址加载系统类层次结构](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Art/nsobject_hierarchy_2x.png)

### URL加载

*URL loading system*最常用的类，让你的应用从源上获取URL的内容。您可以使用`NSURLSession`获取内容。具体的使用方法在很大程度上取决于是将数据读取到内存中，还是将其下载到磁盘。

#### 获取数据（In Menory）

高层次的说，有两个基本的方法来获取URL数据：

- 对于简单的请求，直接使用`NSURLSession`API，通过`NSURL`对象来获取内容，要么是作为一个`NSData`对象，要么是磁盘上的文件。
- 对于复杂些的请求，例如上传数据，提供了`NSURLRequest`对象（或它的子类`NSMutableURLRequest`）。

无论你选择哪种方法，你的应用程序可以通过两种方式获得响应数据：

- 提供*completion handler block*。当从服务器接收数据完成之后，*URL loading class*会调用该block。
- 提供自定义[delegate](undefined)。当它从始发源接收数据，*URL loading class*会定期调用你的delegate方法。您的应用程序负责堆叠这些数据。

除了数据本身，*URL loading class*会为delegate或block，提供response对象，此对象封装了request元数据，如MIME类型和 content length。

**相关章节：** [使用NSURLSession](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)

#### 下载文件

更深层地，有两个基本的方法来下载文件内容：

- 对于简单的请求，直接使用`NSURLSession`API，通过`NSURL`对象来获取内容，要么是作为一个`NSData`对象，要么是磁盘上的文件。
- 对于复杂些的请求，例如上传数据，提供了`NSURLRequest`对象（或它的子类`NSMutableURLRequest`）。

**注：**  由`NSURLSession`实例发起的下载不缓存。如果你需要缓存结果，您的应用程序必须使用`NSURLSession`，并自己将数据写入到磁盘。

**相关章节：** [使用NSURLSession](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)

### Helper Classes

*URL Loading classes*使用两个辅助类，提供额外的元数据，一个是请求本身（`NSURLRequest`），另一个是服务器的响应（`NSURLResponse`）。

> 注：元数据（**Metadata**），又称**中介数据**、**中继数据**，为描述数据的数据（data about data），主要是描述数据[属性]()的信息，用来支持如指示存储位置、历史数据、资源查找、文件记录等功能。

#### URL Requests

一个`NSURLRequest`对象封装了URL和协议相关的属性。

**注意：**  当客户端应用程序发起连接，或使用`NSMutableURLRequest`示例下载，[深拷贝](undefined)这个请求。原始请求的任何改变是无效的，直到下载完成。

某些协议还支持特定的属性。例如，HTTP协议将方法添加到`NSURLRequest`，可以返回HTTP请求的请求提，请求头和传输方法。同理`NSMutableURLRequest`。

URL请求对象更多细节，会在这本书中描述。

#### 响应元

服务器的响应可被分成两个部分：描述内容的元数据和内容数据本身。在大多数协议内很常见，Metadata是由`NSURLResponse`类封装，包含MIME type，expected content length,文本编码text encoding , URL等 。当然，`NSURLResponse`的其他子类可以提供其他元数据。例如，`NSHTTPURLResponse`存储了请求提和由Web服务器返回的状态代码status code 。

**重要：**  只有在响应的元数据被存储在`NSURLResponse`对象。不同的URL loading class，要么通过delegate，要么通过completion block，为app提供响应数据。

NSCagcgURLResponse实例封装了NSURLResponse对象和the URL content data，以及其他app提供的信息。详见 [Cache Management](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/20001834-155585) 。

### 重定向和其他请求更改

某些协议，如HTTP，会提供一个方法，让服务器通知app，内容被移动到不同URL。当这种情况发生，*URL loading class*可以通知其delegate。如果您的应用程序提供了一个合理的delegate方法，您的应用程序可以再决定是否跟随重定向，是通过重定向之后返回响应主体，还是返回一个错误。

**相关章节：**[Handling Redirects and Other Request Changes](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/RequestChanges.html#//apple_ref/doc/uid/TP40009506-SW1)

### 身份验证和证书

> 身份验证（Authentication）指客户端身份验证，要求用户提供证书（客户端证书，用户名和密码等等）。

一些服务器限制访问某些内容，要求用户提供证书（客户端证书，用户名和密码等等）进行身份验证，以获得访问权。在Web服务器的情况下，限制内容被分成需要不同证书的组成（部分）。证书也被用来确定信任关系，反过来说，也可以评估您的应用程序是否应该信任这台服务器。

*URL loading system*提供了类，来建模证书和保护区，以及安全持久化证书。您的应用程序可以指定这些证书只坚持唯一的一个请求，要么永久在用户的钥匙串，要么在应用程序的启动时。

**注：**  证书（持久化存储）保存在用户的钥匙串，在其他所有应用程序之间是共享的。

`NSURLCredential`封装了证书，包括验证信息（例如用户名和密码）和持久性的行为。`NSURLProtectionSpace`表示需要特定证书的区域。一个保护空间可以限定在单一的URL，包含一个Web服务器上的领域，或代理。

一个共享实例`NSURLCredentialStorage`管理证书存储，和提供`NSURLCredential`对象和相应`NSURLProtectionSpace`的映射。

`NSURLAuthenticationChallenge`封装了信息，由`NSURLProtocol`身份验证请求的实现规定：a proposed credential, the protection space involved, the error or response that the protocol used to determine that authentication is required, and the number of authentication attempts that have been made。一个`NSURLAuthenticationChallenge`实例还需指定发起身份验证的对象。这个发起的对象，称为*sender*，必须遵守`NSURLAuthenticationChallengeSender`协议。

`NSURLAuthenticationChallenge`实例被`NSURLProtocol`子类所使用，通知URL loading system 需要身份验证。还有`NSURLSession`的委托方式，方便定制身份验证处理。

**相关章节：**[Authentication Challenges and TLS Chain Validation](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)

### Cache管理

URL loading system提供了一个磁盘和内存的综合缓存方案，让应用减少对网络连接的依赖，并为以前缓存的响应提供更快的周转。缓存是每个应用程序的基础。根据初始`NSURLRequest`和`NSURLSessionConfiguration`对象指定的高速缓存策略，缓存由`NSURLSession`查询。

`NSURLCache`类提供了配置缓存大小和磁盘缓存位置的方法。它还提供了方法，来管理`NSCachedURLResponse`集合（包含了已缓存的响应对象）。

一个`NSCachedURLResponse`对象封装`NSURLResponse`对象和URL的内容数据。`NSCachedURLResponse`还提供了用户信息的字典，你的应用程序可以缓存任何自定义数据。

并非所有的协议实现支持响应缓存。目前只有`http`和`https`支持请求缓存。

一个`NSURLSession`对象可以控制响应是否缓存，以及响应是否只缓存在内存中，通过实现`URLSession:dataTask:willCacheResponse:completionHandler:` 的委托方法。

**相关章节：** [Understanding Cache Access](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html#//apple_ref/doc/uid/20001843-BAJEAIEE)

### Cookie的存储

由于HTTP协议的无状态特性，客户经常使用cookies，提供请求数据在不同URL之间的持久化存储。*URL loading system*提供接口，来创建和管理cookies，作为HTTP请求的一部分发送cookies，并且在解析Web服务器的响应时，接收cookies。

OS X和iOS提供`NSHTTPCookieStorage`类，这又提供了一种接口，来管理集合`NSHTTPCookie`对象。在OS X中存储的cookie可以在所有应用程序之间共享; 在iOS中，cookie的存储是仅限每个应用程序内。

**相关章节：** [Cookie Storage](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW1)

### 协议支持

URL loading system本身支持`http`，`https`，`file`，`ftp`，和`data`协议。然而，URL loading system还可以让你的应用程序注册自己的类，来支持其他的应用层网络协议。您还可以添加特定协议的属性，URL请求和URL响应对象。

**相关章节：**[Cookies and Custom Protocols](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW3)

## 如何使用本文档

通过阅读[使用NSURLSession](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)，了解*URL loading system*的概述。然后阅读 [Life Cycle of a URL Session](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1) 详细了解`NSURLSession`如何与它的delegate进行交互。在下面的章节中提供了URL loading system其他方面的详细信息：

- [Encodeing URL Date](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/WorkingwithURLEncoding/WorkingwithURLEncoding.html#//apple_ref/doc/uid/10000165i-CH12-SW1)说明如何编码字符串，使他们在URL中安全使用。
- [Handling Redicrecta and Other Request Changes](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/RequestChanges.html#//apple_ref/doc/uid/TP40009506-SW1)说明回应重定向URL请求的选项。
- [Authentication Challenges and TLS Chain Validation](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)说明安全服务器的连接的身份验证过程。
- [Understanding Cache Access](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html#//apple_ref/doc/uid/20001843-BAJEAIEE)描述了一个连接的请求时，如何使用缓存。
- [Cookies and Custom Protocols](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW3)解释了可用于管理存储的cookie和支持自定义的应用层协议的类。