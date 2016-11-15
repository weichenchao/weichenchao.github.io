---
layout: post
title: Life Cycle of a URL Session
tags: 
- NSURLSession

---

# 一个URL会话生命周期

您可以使用`NSURLSession`API有两种方式：使用系统提供的委托，或使用自定义委托。一般情况下，你必须使用自定义委托，如果你的应用程序需要做下列处理：

- 使用后台会话上传或下载内容，当您的应用程序没有运行。
- 执行自定义验证。
- 执行自定义SSL证书验证。
- 决定传输是否应被下载到磁盘，或基于由服务器返回的MIME类型显示。
- 从body stream上载数据（而不是一个`NSData`对象）。
- 代码限制缓存。
- 代码限制HTTP重定向。

如果您的应用程序并不需要做上述事件，你的应用程序可以使用系统提供的委托。根据您选择哪种技术，你应该阅读以下各节之一：

- [Life Cycle of a URL Session with System-Provided Delegates](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW2)提供如何创建和使用URL会话的一个轻量级视图。即使你打算自定义delegate，你也应该阅读这部分，因为它给一个完整的概论和全貌，你的代码如何配置对象并使用。
- [Life Cycle of a URL Session with Custom Delegates](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW42) 提供了一个URL会话的每一步操作的完整视图。你应该参考本节以帮助您了解会话如何与delegate互动。特别是，将说明每个委托方法何时被调用。

## 用系统提供的代理，URL会话生命周期

如果您使用的是`NSURLSession`，不需要提供任何delegate对象，该系统提供的delegate会自动为您处理许多细节。这是方法和completion handler的调用基本顺序，方法是你的app制造的，completion handler是你的app接收到的：

1. 创建session的configuration。如果是后台会话，该配置必须包含一个唯一的标识符。存储这个标识，如果你的应用程序崩溃，被终止或暂停，可用它与会话重新关联。

2. 创建一个会话，指定configuration对象和并将delegate属性设置为`nil`。

3. 在会话中，创建task对象，一个task对象代表一个资源请求。

   每个任务都开始于挂起状态。您的应用程序调用对某个任务进行`resume`后，应用程序开始下载指定的资源。

   任务对象 ，是子类`NSURLSessionTask` ，`NSURLSessionDataTask`，`NSURLSessionUploadTask`或`NSURLSessionDownloadTask`的实例，取决于你正在实现的行为。

   你的应用程序可以（而且通常应该）添加了多个任务到session中，简单起见，剩下的步骤描述的是一个task的生命周期。

   **重要提示：**  如果您使用的是`NSURLSession`，并不提供自定义delegate，您的应用程序必须用含有`completionHandler`参数的方法创建task，否则就不能获取下载的数据。

4. 对于下载任务，在服务器的传输过程中，如果用户需要暂停下载，通过调用`cancelByProducingResumeData:`的方法来取消任务。之后，返回的恢复数据，传递给要么`downloadTaskWithResumeData:`要么`downloadTaskWithResumeData:completionHandler:`方法来创建新的下载任务（新的下载任务是继续之前的下载）。

5. 当任务完成后，`NSURLSession`对象调用completion handler。

   **注：** `NSURLSession`无法通过error参数来报告服务器错误。您的应用程序接收的、通过error参数的唯一错误指的是客户端的错误，如暂时无法解析主机名或连接到主机。详见`URL Loading System Error Codes`。

   服务器端错误通过`NSHTTPURLResponse`对象的HTTP状态代码反馈。欲了解更多信息，请阅读的文档`NSHTTPURLResponse`和`NSURLResponse`。

6. 当你的应用程序不再需要一个会话，要么调用`invalidateAndCancel`（取消未完成的任务），要么调用`finishTasksAndInvalidate`（允许未完成的任务完成，然后取消session），来取消session。

## 使用自定义代表一个URL会话生命周期

通常，您可以使用`NSURLSession`API也不提供委托。但是，如果您使用`NSURLSession`API去后台下载和上传，或者如果你需要用非默认的方式验证或缓存处理，您必须提供一个delegate，这个delegate必须遵守遵守session delegate协议，一个及以上的task delegate协议，或前两个协议的某种组合。这个delegate有许多用途：

- 当下载任务时，`NSURLSession`对象使用delegate来给你的应用程序来提供文件URL，通过file URL可以获取下载的数据。

  所有后台下载和上传需要这些delegates。这些delegates必须提供所有`NSURLSessionDownloadDelegate`的协议内的委托。

- delegate可以处理某些认证的challenge。

- delegate为数据上传到远程服务器提供body stream。

- delegate们可以决定是否遵循HTTP重定向。

- `NSURLSession`对象使用delegate来提供应用程序每次传输的状态。data task delegate接收的不仅仅是一个初始呼叫，您可以在其中将请求转换为下载和后续调用，因为他们从远程服务器到达后是提供数据块。

- delegates是一种方式，`NSURLSession`对象通知你的应用程序时，一个传输完成。

如果您正在使用自定义代理，一个URL会话的整个生命周期比较复杂。这是方法和delegate的基本调用顺序：

1. 创建session的configuration。对于后台会话，该configuration必须包含一个唯一的标识符。存储此标识，如果你的应用程序崩溃、被终止或暂停，可用它与会话重新关联。

2. 创建会话，指定的配置对象和代理，代理是可选选项。

3. 在会话中，创建task对象，一个task对象代表一个资源请求。

   每个任务都开始于挂起状态。您的应用程序调用对某个任务进行`resume`后，应用程序开始下载指定的资源。

   任务对象 ，是子类`NSURLSessionTask` ，`NSURLSessionDataTask`，`NSURLSessionUploadTask`或`NSURLSessionDownloadTask`的实例，取决于你正在实现的行为。

   你的应用程序可以（而且通常应该）添加了多个任务到session中，简单起见，剩下的步骤描述的是一个task的生命周期。

4. 如果远程服务器返回一个状态代码表示认证是必需的，如果认证需要一个连接级别的挑战（如SSL客户端证书），`NSURLSession`呼叫的验证挑战委托方法。

   - 对于会话级challenges- `NSURLAuthenticationMethodNTLM`，`NSURLAuthenticationMethodNegotiate`，`NSURLAuthenticationMethodClientCertificate`，`NSURLAuthenticationMethodServerTrust`- `NSURLSession`对象调用session delegate的`URLSession:didReceiveChallenge:completionHandler:`方法。如果您的应用程序不提供session delegate方法，该`NSURLSession`对象则会调用task delegate的`URLSession:task:didReceiveChallenge:completionHandler:`方法来处理challenge。
   - 对于非会话级的挑战（所有其他），`NSURLSession`对象调用session delegate的`URLSession:task:didReceiveChallenge:completionHandler:`方法来处理challenge。如果您的应用程序提供了一个session delegate和需要处理的身份验证，那么您必须在task level处理验证，或明确给每个会话handler提供一个task level handler。会议委托的`URLSession:didReceiveChallenge:completionHandler:`方法不处理非会话级challenge。

   **注：**  Kerberos身份验证是透明的处理。

   如果上传任务的身份验证失败，如果任务的数据来源于stream，`NSURLSession`对象调用委托的`URLSession:task:needNewBodyStream:`委托方法。然后delegate必须提供一个新的`NSInputStream`，目的是为新的请求提供所述主体数据。

   有关编写的认证委托方法的更多信息`NSURLSession`，请阅读[Authentication Challenges and TLS Chain Validation](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)。

5. 在接收HTTP重定向的响应，`NSURLSession`对象调用委托的`URLSession:task:willPerformHTTPRedirection:newRequest:completionHandler:`方法。该委托方法会调用completion handler，由`NSURLRequest`对象（跟随重定向），或一个新的`NSURLRequest`对象（重定向到一个不同的URL），或`nil`（将重定向的响应体作为一个有效的响应，并作为结果返回）提供。

   - 如果重定向之后，回到步骤4（认证质询处理）。
   - 如果委托没有实现这个方法，重定向随访至重定向的最大数量。

6. 通过调用创建的（重新）下载任务`downloadTaskWithResumeData:`或者`downloadTaskWithResumeData:completionHandler:`，`NSURLSession`调用委托的`URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:`。

7. 对于数据的任务，该`NSURLSession`对象调用委托的`URLSession:dataTask:didReceiveResponse:completionHandler:`方法。决定是否将数据任务转换成下载任务，然后调用completion callback继续接收数据或下载数据。

   如果您的应用程序选择数据的任务转化为一个下载任务，`NSURLSession`调用委托的`URLSession:dataTask:didBecomeDownloadTask:`。这个调用后，委托不会再收到从数据任务的回调，而是从下载任务开始接收回调。

8. 如果用`uploadTaskWithStreamedRequest:`创建任务，`NSURLSession`调用委托的`URLSession:task:needNewBodyStream:`方法来提供body data。

9. 在体内容服务器初始上传期间，委托会定期收到`URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:`回调，报告上传进度。

10. 服务器传输期间，task delegate定期会接收一个回调来报告传输的进度。对于一个下载任务时，会调用委托的`URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:`方法，报告成功写入到磁盘的字节数。对于一个data task，session调用delegate的`URLSession:dataTask:didReceiveData:`，报告接收的数据的实际体积。

    对于下载任务，在服务器的传输过程中，如果用户告诉您的应用程序暂停下载，通过调用`cancelByProducingResumeData:`来取消该任务。

    稍后，如果用户请求您的应用程序继续下载，返回的恢复数据传递给，要么是`downloadTaskWithResumeData:`要么`downloadTaskWithResumeData:completionHandler:`方法，来创建继续下载新的下载任务，然后转到步骤3（创建和恢复任务的对象）。

11. 对于数据的任务，该`NSURLSession`对象调用委托的`URLSession:dataTask:willCacheResponse:completionHandler:`方法。然后，您的应用程序应该决定是否允许缓存。如果不实现此方法，默认行为是使用session的configuration对象指定的缓存策略。

12. 如果下载任务成功完成，则该`NSURLSession`对象用一个位置的*临时*文件，调用任务的`URLSession:downloadTask:didFinishDownloadingToURL:`。在委托方法返回之前，您的应用程序必须要么从该文件中读取响应数据，要么将其移动到应用程序的沙盒容器目录中的永久位置。

13. 当所有任务完成后，`NSURLSession`对象调用委托的`URLSession:task:didCompleteWithError:`方法，要么是错误对象，要么是`nil`（如果任务成功完成）。

    如果任务失败，除非用户取消下载或服务器返回表示请求绝不会成功的错误，大多数应用程序应该重试请求。但是，您的应用程序不应该立即重试。相反它应该使用reachablity APIs来确定服务器是否通，并应该收到reachabilty改变的通知的时候，才创建新的要求。

    如果下载任务就可以恢复，`NSError`对象的`userInfo`字典包含的值`NSURLSessionDownloadTaskResumeData`键。您的应用程序应该通过这个值来调用`downloadTaskWithResumeData:`或`downloadTaskWithResumeData:completionHandler:`创建一个延续了现有下载的新下载任务。

    如果任务不能恢复，你的应用程序应该创建一个新的下载任务，然后从头开始重新启动事务。

    在任一情况下，如果传输失败，除了一个服务器错误以外的任何原因，就应该转到步骤3（创建和恢复任务对象）。

    **注：** `NSURLSession`通过误差参数不报告服务器错误。您的代理接收通过错误的参数的唯一错误是客户端的错误，如暂时无法解析主机名或连接到主机。错误代码中描述`URL Loading System Error Codes`。服务器端错误都在HTTP状态代码报道`NSHTTPURLResponse`对象。欲了解更多信息，请阅读的文档`NSHTTPURLResponse`和`NSURLResponse`类。

14. 如果响应是多部分编码，会话可能会再次调用的`didReceiveResponse`方法，跟随零个或多个其他`didReceiveData`电话。如果出现这种情况，则转到步骤7（处理`didReceiveResponse`调用）。

15. 当你不再需要一个会话，通过调用，要么`invalidateAndCancel`（取消未完成的任务）或`finishTasksAndInvalidate`（允许未完成的任务完成下载，再使session无效），使session无效。

    会话无效后，当所有未完成的任务已取消或已经完成，会话发送`URLSession:didBecomeInvalidWithError:`消息给委托。当该委托方法返回，session会移除对delegate的强引用。

    **重要提示：**  会话对象对delegate强引用，直到您的应用程序明确使会话失效。如果你不使会话失效，应用程序会内存泄漏。

如果您的应用程序取消正在进行的下载，`NSURLSession`对象会调用委托的`URLSession:task:didCompleteWithError:`方式，就好像发生了错误。