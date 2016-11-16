---
layout: post
title: Understand Cache Accesss
tags: 
- NSURLSession

---

# 了解高速缓存访问

URL Loading system给请求和响应提供一个复合的磁盘和内存缓存方式。此缓存允许应用程序减少对网络连接的依赖，并提高其性能。

## 请求使用缓存

一个`NSURLRequest`实例可以说明本地缓存是如何使用的，通过将缓存策略设置为`NSURLRequestCachePolicy`枚举中的一个值：`NSURLRequestUseProtocolCachePolicy`，`NSURLRequestReloadIgnoringCacheData`，`NSURLRequestReturnCacheDataElseLoad`，或`NSURLRequestReturnCacheDataDontLoad`。

`NSURLRequest`实例的默认缓存策略是`NSURLRequestUseProtocolCachePolicy`。的`NSURLRequestUseProtocolCachePolicy`行为是特定的协议和被定义为符合协议最好的策略。

设置高速缓存策略为`NSURLRequestReloadIgnoringCacheData`，会使URL加载系统从始发源地址加载数据，完全忽略缓存。

`NSURLRequestReturnCacheDataElseLoad`高速缓存策略，导致URL加载系统使用高速缓存的数据，而忽略其年龄或到期日期，并且仅当不存在缓存版本，才会从始发源加载数据。

`NSURLRequestReturnCacheDataDontLoad`策略允许应用程序指定，只有高速缓存数据应被返回。如果响应不是在本地缓存，用此高速缓存策略创建一个`NSURLSessionTask`与，会立即返回`nil`。这是在功能上的“离线”模式相似，并且永远不会启动了网络连接。

**注：**  目前，只有HTTP和HTTPS请求的响应可被缓存。根据所允许的高速缓存策略，FTP和`file`协议尝试访问原始源。自定义`NSURLProtocol`类可以选择性地提供缓存。

## HTTP协议的缓存使用场景

最复杂的高速缓存的使用情况是，当一个请求使用HTTP协议，并已设置缓存政策为`NSURLRequestUseProtocolCachePolicy`。

如果对应请求的`NSCachedURLResponse`不存在，则该URL加载系统从始发源获得数据。

如果对应请求的高速缓存的响应存在，则URL加载系统进行检查响应，以确定它是否指定该内容必须被重新验证。

如果内容必须被重新验证，该URL加载系统发起HEAD请求到始发源，以查看该资源是否已经改变。如果没有改变的话，网址加载系统返回缓存的响应。如果它已经改变了，URL装载系统则取出从始发源的数据。

如果缓存响应内容不需要重新验证，网址加载系统会检查缓存响应中规定的最大年龄或期限。如果缓存的响应足够近，则URL加载系统返回缓存的响应。如果响应是过期的，则URL加载系统发送HEAD请求到始发源，以确定资源是否已改变。如果是改变，则URL装载系统取出从始发源的资源。否则，它返回缓存的响应。

RFC 2616，第13章（[http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13)）指定涉及详细的语义。

## 代码控制缓存

默认情况下，该请求的数据基于所述请求的高速缓存策略进行缓存，由`NSURLProtocol`子类处理该请求。

如果您的应用需要在高速缓存更精确的代码控制（如果该协议支持缓存），可以实现委托方法，使您的应用程序，以确定在每个请求对应的响应是否应该被缓存。

对于`NSURLSession`数据和上载的任务，执行`URLSession:dataTask:willCacheResponse:completionHandler:`方法。此委托方法仅用于数据和上传的task。下载任务的缓存策略由专门指定的缓存策略决定。

对于`NSURLSession`，您的委托方法调用completion handler block告诉会话需要缓存。委托通常提供以下的值：

- 所提供的响应对象，允许缓存
- 新创建的响应对象，为了缓存一个修改的响应，例如，具有存储策略的响应，允许高速缓存到内存而不是磁盘
- `nil` ，以防止缓存

您的委托方法也能将对象插入`userInfo`字典（这个字典也与`NSCachedURLResponse`对象有关联），从而导致存储在缓存中的那些对象作为响应的一部分。

**重要提示：**  您的委托方法*必须始终*调用completion handler。否则，您的应用程序泄漏内存。

在该示例清单5-1   HTTPS响应阻止磁盘上的缓存。它还增加了当前日期到user info字典。

**清单5-1**   示例`URLSession:dataTask:willCacheResponse:completionHandler:`实施

```objective-c
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
 willCacheResponse:(NSCachedURLResponse *)proposedResponse
 completionHandler:(void (^)(NSCachedURLResponse * __nullable cachedResponse))completionHandler {
    NSCachedURLResponse *newCachedResponse = proposedResponse;
    NSDictionary *newUserInfo;
    newUserInfo = [NSDictionary dictionaryWithObject:[NSDate date]
                                              forKey:@"Cached Date"];
    if ([proposedResponse.response.URL.scheme isEqualToString:@"https"]) {
#if ALLOW_IN_MEMORY_CACHING
        newCachedResponse = [[NSCachedURLResponse alloc]
                             initWithResponse:proposedResponse.response
                             data:proposedResponse.data
                             userInfo:newUserInfo
                             storagePolicy:NSURLCacheStorageAllowedInMemoryOnly];
#else // !ALLOW_IN_MEMORY_CACHING
        newCachedResponse = nil;
#endif // ALLOW_IN_MEMORY_CACHING
    } else {
        newCachedResponse = [[NSCachedURLResponse alloc]
                             initWithResponse:[proposedResponse response]
                             data:[proposedResponse data]
                             userInfo:newUserInfo
                             storagePolicy:[proposedResponse storagePolicy]];
    }
   completionHandler(newCachedResponse);
}
```





[下一页](https://developer.apple.com/library/prerelease/content/documentation/Cocoa/Conceptual/URLLoadingSystem/CookiesandCustomProtocols/CookiesandCustomProtocols.html)[上一页](https://developer.apple.com/library/prerelease/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html)

------

版权所有©2003年，2016年苹果公司保留所有权利。 [使用条款](http://www.apple.com/legal/internet-services/terms/site.html) | [隐私政策](http://www.apple.com/privacy/) | 更新日期：2016年9月13日

反馈

****

![img](https://www.gstatic.com/images/branding/product/1x/translate_24dp.png)