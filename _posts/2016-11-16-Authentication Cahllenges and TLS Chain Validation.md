---
layout: post
title: Authentication Callenges and TLS Chain Validation
tags: 
- NSURLSession

---

# 身份验证质询和传输层安全连接验证
> authentication challenge：身份验证质询
>
> TLS：传输层安全

一个`NSURLRequest`对象常常遇到一个**身份验证质询**，或者从它连接到服务器的请求证书。`NSURLSession`类通知其委托 ，当请求遇到一个身份验证质询，使他们可以采取相应的行动。

**重要提示：**  URL加载系统类并不调用他们的委托来处理请求，除非挑战服务器响应包含了一个`WWW-Authenticate`响应头。其他身份验证类型，如代理身份验证和TLS信任验证不需要这个响应头。

## 决定如何应对身份验证质询

如果会话任务需要验证，并且没有有效的证书提供，要么作为所请求的URL的一部分，要么在共享`NSURLCredentialStorage`时，它创建一个身份验证质询。它首先发送`URLSession:task:didReceiveChallenge:completionHandler:`消息给task delegate来处理身份验证质询。如果任务委托不给消息做出响应，任务发送`URLSession:task:didReceiveChallenge:completionHandler:`消息给session的委托来处理身份验证质询。

为了继续连接，委托有三个选项：

- 提供身份验证证书。
- 尝试继续无证书。
- 取消身份验证质询。

为了帮助确定正确操作，`NSURLAuthenticationChallenge`传递给方法的实例，这包含有关触发身份验证质询的信息，多少次质询，所有之前的证书，`NSURLProtectionSpace`需要的证书，以及challenge的发件人。

如果身份验证质询以前试图验证，但是失败了（例如，如果用户在服务器上改变了他密码），您可以通过调用身份验证质询的`proposedCredential`，来获取之前的证书。然后，委托可以使用这些凭据，来填充它呈现给用户的一个对话框。

调用`previousFailureCount`的认证挑战，返回先前的认证尝试，包括来自不同的认证协议的总数。委托可以提供此信息给用户，以确定它先前提供的凭证是否失败的，或是因为限制认证尝试的最大次数。

## 响应身份验证质询

以下是你可响应`URLSession:didReceiveChallenge:completionHandler:`或`URLSession:task:didReceiveChallenge:completionHandler:`委托方法的三种方法。

### 提供证书

要尝试进行验证，应用程序应该创建一个`NSURLCredential`对象，包含由服务器所期望的格式的认证信息。你可以通过调用确定服务器的身份验证方法`authenticationMethod`上提供的验证挑战的保护空间。`NSURLCredential`支持的一些身份验证方法是：

- HTTP基本验证（`NSURLAuthenticationMethodHTTPBasic`）需要用户名和密码。提示用户输入必要的信息，并用对象`credentialWithUser:password:persistence:`创建一个`NSURLCredential`。
- HTTP摘要认证（`NSURLAuthenticationMethodHTTPDigest`），类似基本身份验证需要用户名和密码。（摘要是自动生成的）提示用户输入必要的信息，并用对象`credentialWithUser:password:persistence:`创建一个`NSURLCredential`。
- 客户端证书认证（`NSURLAuthenticationMethodClientCertificate`）要求系统的身份，并与服务器进行验证所需的所有证书。用`credentialWithIdentity:certificates:persistence:`创建`NSURLCredential`。
- 服务器trust认证（`NSURLAuthenticationMethodServerTrust`）要求通过认证挑战的保护空间中的信任。用`credentialForTrust:`创建一个`NSURLCredential`对象。

你创建了之后`NSURLCredential`的对象，使用completion handler block，传递对象给身份验证质询的发件人。

### 无证书继续

如果delegate不提供认证挑战的凭证，它可以只能尝试继续。将以下值来给completion handler block：

- `NSURLSessionAuthChallengePerformDefaultHandling` 处理该请求，就好像代表没有提供委托方法来处理的挑战。
- `NSURLSessionAuthChallengeRejectProtectionSpace`拒绝challenge。根据服务器的响应所允许的身份验证类型，该URL装载类可以调用此委托的方法不止一次，对于额外的保护空间。

### 取消连接

委托还可以取消身份验证质询，通过传递`NSURLSessionAuthChallengeCancelAuthenticationChallenge`到所提供的completion handler block。

### 身份验证示例

清单4-1  通过创建响应挑战`NSURLCredential`与由应用程序的首选项提供的用户名和密码的实例。如果认证以前失败了，它将取消认证质询，并通知用户。

**清单4-1**   使用的一个例子`URLSession:didReceiveChallenge:completionHandler:`委托方法

```objective-c
- (void)URLSession:(NSURLSession *)session
didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
  completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential * __nullable credential))completionHandler
{
    if ([challenge previousFailureCount] == 0) {
        NSURLCredential *newCredential = [NSURLCredential credentialWithUser:[self preferencesName]
                                                                    password:[self preferencesPassword]
                                                                 persistence:NSURLCredentialPersistenceNone];
        completionHandler(NSURLSessionAuthChallengeUseCredential, newCredential);
    } else {
        // Inform the user that the user name and password are incorrect
        completionHandler(NSURLSessionAuthChallengeCancelAuthenticationChallenge, nil);
    }
}
```



## 执行自定义TLS链验证

在NSURL系列API中，TLS链验证是由您的应用程序的认证委托方法处理，但不是提供凭证给用户（或应用程序）的服务器进行身份验证，您的应用程序，而不是检查服务器的TLS期间提供的凭证握手，然后告诉URL装系统是否应该接受或拒绝这些凭据。

如果你需要在一个非标准的方式进行链验证（如接受测试特定的自签名证书），您的应用程序必须实现的`URLSession:didReceiveChallenge:completionHandler:`或`URLSession:task:didReceiveChallenge:completionHandler:`委托方法。如果你同时实现，会话级的方法是负责处理身份验证。

在您的验证处理程序委托方法，你应该检查，看看是否攻击的保护空间具有认证类型`NSURLAuthenticationMethodServerTrust`，如果是这样，获得`serverTrust`从保护空间信息。

[下一页](https://developer.apple.com/library/prerelease/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html)[上一页](https://developer.apple.com/library/prerelease/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/RequestChanges.html)

------

版权所有©2003年，2016年苹果公司保留所有权利。 [使用条款](http://www.apple.com/legal/internet-services/terms/site.html) | [隐私政策](http://www.apple.com/privacy/) | 更新日期：2016年9月13日

反馈

****
