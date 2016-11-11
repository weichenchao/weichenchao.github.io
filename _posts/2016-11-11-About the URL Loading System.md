---
layout: post
title: About the URL Loading System
tags: 
- WebP


---

# About the URL Loading System

# 关于URL加载系统

本指南介绍了这个框架，可用于URL交互，并使用标准的互联网协议与服务器进行通信。这些类一起被称为*URL Loading System*。    *URL Loading System*

*URL Loading System*是一套class和protocol，使您的应用程序通过URL访问内容。在这项技术的核心是`NSURL`class，它可以让你的应用程序处理URL和resource。

为了支持`NSURL`class，Foundation框架提供一系列丰富的class，让您加载URL的内容，上传数据到服务器，管理cookie存储，控制响应缓存，处理凭证存储和应用程序特定的认证方式，并写自定义协议的扩展。

   *URL Loading System*提供了以下协议进行访问资源的支持：

- 文件传输协议（`ftp://`）
- 超文本传输协议（`http://`）
- 加密超文本传输协议（`https://`）
- 本地文件URL（ `file:///`）
- 数据的网址（`data://`）

它还透明地支持代理服务器和SOCKS网关，使用用户的系统偏好。

> **重要提示：**  除了*URL Loading System*，Mac OS X和iOS在其他应用程序，如Safari打开URL提供的API。这些API未在本文档中描述。
>
> 有关启动服务在OS X的更多信息，请阅读 *Launch Services Programming Guide*。
>
> 有关详细信息，`openURL:`该方法`NSWorkSpace`在OS X类，阅读 *NSWorkspace Class Reference*。
>
> 有关详细信息，`openURL:`该方法`UIApplication`在iOS中类，阅读*的UIApplication类参考*。

## 总览

*URL Loading System*包括了一些URL加载的重要辅助类。主要辅助类分为五类：协议支持，认证和证书，cookie存储，配置管理和缓存管理。

![该网址加载系统类层次结构](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Art/nsobject_hierarchy_2x.png)

### URL加载

最常用的类让你的应用,从源头上检索的URL的内容。您可以通过`NSURLSession`检索内容。您使用的具体方法在很大程度上取决于您是否希望将数据读取到内存中或将其下载到磁盘。

#### 获取数据（In Menory）

在较高的水平，有获取URL数据的两个基本方法：

- 对于简单的请求，直接使用`NSURLSession`API来检索内容，通过`NSURL`对象，要么是作为一个`NSData`对象，要么是磁盘上的文件。
- 对于更复杂的请求，例如上传数据，提供了`NSURLRequest`对象（或它的子类`NSMutableURLRequest`）。

无论你选择哪种方法，你的应用程序可以通过两种方式获得响应数据：

- 提供*a completion handler block*。当它从服务器完成接收数据，URL loading class调用该block。
- 提供自定义[delegate](undefined)。URL loading class定期调用你的delegate方法，当它从始发源接收数据。您的应用程序负责积累数据。

除了数据本身，URL loading class为delegate或block，提供response对象（此对象encapsulates封装了request元数据），如MIME类型和 content length。

**相关章节：** [使用NSURLSession](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)

#### 下载文件

更深层地，有两个下载文件内容的基本方法：

- 对于简单的请求，使用`NSURLSession`API来检索从内容`NSURL`直接对象，无论是作为一个`NSData`对象或磁盘上的文件。
- 对于上传数据，例如，提供了一个更复杂的要求，请求`NSURLRequest`对象（或它的子类可变，`NSMutableURLRequest`）来`NSURLSession`。

**注：**  由发起的下载`NSURLSession`实例不缓存。如果你需要缓存的结果，您的应用程序必须使用`NSURLSession`和将数据写入到磁盘本身。

**相关章节：** [使用NSURLSession](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)

### Helper Classes

URL Loading classes使用两个辅助类，提供额外的元数据，一个用于请求本身（`NSURLRequest`），一个用于服务器的响应（`NSURLResponse`）。

#### URL Requests

一个`NSURLRequest`对象封装了URL和协议property，与协议独立的方式。

**注意：**  当客户端应用程序初始化使用实例的连接或下载`NSMutableURLRequest`，一个[深拷贝](undefined)被提出的要求。到发起请求做出的改变下载初始化后没有效果。

某些协议支持特定协议[特性](undefined)。例如，HTTP协议将方法添加`NSURLRequest`该返回HTTP请求的身体，头和传输方法。它还增加了方法`NSMutableURLRequest`来设置这些值。

与URL请求对象工作的细节在这本书中描述。

#### 响应元

服务器的响应可被当做两个部分：描述内容的元数据和内容数据本身。元Metadata是大多数协议（由`NSURLResponse`类封装，包含由MIME type，expected content length,文本编码text encoding , URL 。`NSURLResponse`的其他子类可以提供额外的元数据。例如，`NSHTTPURLResponse`存储了headers和由Web服务器返回的状态代码status code 。

**重要：**  只有在响应的元数据存储在`NSURLResponse`对象。不同的URL loading class，要么通过delegate，要么通过completion block，为app提供response数据。

NSCagcgURLResponse实例封装了NSURLResponse实例，the URL content data，以及其他额外信息。详见 [Cache Management](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/20001834-155585) 。

### 重定向和其他请求更改

某些协议，如HTTP，会提供一个方法，让服务器传达app，内容被移动到不同URL。当这种情况发生，URL loading class可以通知其delegate。如果您的应用程序提供了一个合理的delegate方法，您的应用程序可以再决定是否重定向，通过重定向之后返回响应主体，或是返回一个错误。

**相关章节：**[Handling Redirects and Other Request Changes](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/RequestChanges.html#//apple_ref/doc/uid/TP40009506-SW1)

### 认证和证书

一些服务器限制访问某些内容，要求用户提供某种凭证（客户端证书，用户名和密码等等）通过验证，以获得访问权。在Web服务器的情况下，限制内容被分成需要一系列凭据的领域（组成，部分）。证书也被用来确定信任关系，另一个方向来评估您的应用程序是否应该信任这台服务器。

URL loading system提供了类，来提供模型的证书和保护区，安全凭证持久化。您的应用程序可以指定这些证书坚持唯一一个请求，要么永久在用户的钥匙串，要么在应用程序的启动时。

**注：**  证书（存储在persistent stroage）保存在用户的钥匙串，共享其他所有应用程序之间。

`NSURLCredential`封装了证书，包括验证信息（例如用户名和密码）和持久性的行为。`NSURLProtectionSpace`表示需要特定证书的区域。一个保护空间可以限定在一个单一的URL，包含一个Web服务器上的领域，或代理。

一个共享实例`NSURLCredentialStorage`管理证书存储，和提供`NSURLCredential`对象和相应`NSURLProtectionSpace`的映射。

由`NSURLProtocol`认证request的实现规定，`NSURLAuthenticationChallenge`封装信息：a proposed credential, the protection space involved, the error or response that the protocol used to determine that authentication is required, and the number of authentication attempts that have been made。一个`NSURLAuthenticationChallenge`实例还需指定发起认证的对象。初始的对象，称为*sender*，必须遵守`NSURLAuthenticationChallengeSender`协议。

`NSURLAuthenticationChallenge`实例被`NSURLProtocol`子类所使用，通知URL loading system 需要认证。他们还提供给[委托](undefined)的方式`NSURLSession`有利于定制认证处理。

**相关章节：**[Authentication Challenges and TLS Chain Validation](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)

### Cache管理

URL loading system提供了一个磁盘和内存的综合缓存方案，让应用减少对网络连接的依赖，并为以前缓存的响应提供更快的周转。缓存是每个应用程序的基础。根据由初始`NSURLRequest`和`NSURLSessionConfiguration`对象指定的高速缓存策略，缓存由`NSURLSession`查询。

`NSURLCache`类提供了配置缓存大小和磁盘上的位置的方法。它还提供了方法，来管理的集合`NSCachedURLResponse`（包含了已缓存的响应对象）。

一个`NSCachedURLResponse`对象封装`NSURLResponse`对象和URL的内容数据。`NSCachedURLResponse`还提供了用户信息的字典，你的应用程序可以缓存任何自定义数据。

并非所有的协议实现支持响应缓存。目前只有`http`和`https`支持请求缓存。

一个`NSURLSession`对象可以控制响应是否缓存，以及响应是否应该只缓存在内存中`URLSession:dataTask:willCacheResponse:completionHandler:` [的委托](undefined)方法。

**相关章节：** [Understanding Cache Access](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html#//apple_ref/doc/uid/20001843-BAJEAIEE)

### Cookie的存储

由于HTTP协议的无状态特性，客户经常使用cookies来提供请求数据的在不同URL之间持久化存储。URL loading system提供接口来创建和管理cookies，送饼干作为HTTP请求的一部分，并且解释Web服务器的响应时，接收cookies。

OS X和iOS提供`NSHTTPCookieStorage`类，这又提供了一种接口，来管理集合`NSHTTPCookie`的对象。在OS X中存储的cookie可以在所有应用程序共享; 在iOS中，cookie的存储是仅限每个应用程序。

**相关章节：** [Cookie Storage](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW1)

### 协议支持

URL loading system本身支持`http`，`https`，`file`，`ftp`，和`data`协议。然而，URL loading system还可以让你的应用程序来注册自己的类来支持其他的应用层网络协议。您还可以添加特定协议的属性，URL请求和URL响应对象。

**相关章节：**[Cookies and Custom Protocols](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW3)

## 如何使用本文档

通过阅读开始[使用NSURLSession](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW1)，了解URL loading system的概述。然后阅读[的URL会话生命周期](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1)详细了解`NSURLSession`如何与它delegate进行交互。在下面的章节中提供了有关URL loading system等方面的详细信息：

- [Encodeing URL Date](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/WorkingwithURLEncoding/WorkingwithURLEncoding.html#//apple_ref/doc/uid/10000165i-CH12-SW1)说明如何编码任意字符串，使他们安全的在URL中使用。
- [Handling Redicrecta and Other Request Changes](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/RequestChanges.html#//apple_ref/doc/uid/TP40009506-SW1)说明回应重定向URL请求的选项。
- [Authentication Challenges and TLS Chain Validation](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)说明安全服务器的连接的验证过程。
- [Understanding Cache Access](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html#//apple_ref/doc/uid/20001843-BAJEAIEE)描述了一个连接的请求时如何使用缓存。
- [Cookies and Custom Protocols](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html#//apple_ref/doc/uid/10000165i-CH10-SW3)解释了可用于管理存储的cookie和支持自定义的应用层协议的类。