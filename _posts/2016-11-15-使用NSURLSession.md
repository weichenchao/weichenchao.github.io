---
layout: post
title: 使用NSURLSession
tags: 
- NSURLSession

---

# 使用NSURLSession

`NSURLSession`及相关类提供了通过HTTP下载内容的API。这个API提供了一套丰富的委托方法，支持认证，后台下载（在app没有运行或是app被挂起的时候）。

使用`NSURLSession`API，您的应用程序创建一系列的会话session，协调一系列相关的数据传输任务。例如，如果你正在编写一个Web浏览器，应用程序可以为每个tab或window创建一个会话。在每个会话中，你的应用程序加了一系列任务，其中每一个都代表特定URL的请求（及如果原始URL返回的HTTP重定向URL）。

类似很多网络API，`NSURLSession`API是高度同步。如果使用默认，系统提供的delegate时，你必须提供completion handler block，数据传输到app是成功还是错误。另外，如果你提供自己的自定义delegate对象，当从服务器接收数据，task对象回去调用这些delegate方法（或者是文件下载，当传输完成）。

**注意：**  completion回调主要是作为替代，来使用自定义委托。如果您使用需要完成completion callback的方法，创建task，那么响应和数据交付delegate方法不会被调用。

`NSURLSession`API提供了状态和进度属性，除了提供这些信息给delegate。它可以支持取消，重新启动，以及暂停任务，恢复暂停，取消和失败的下载。

## 了解URL会话概念

会话中的任务行为取决于三件事：会话的类型（由用于创建它配置对象的类型确定），任务的类型，以及该应用是否是在前台时创建任务。

### 会话类型

`NSURLSession`API支持三种类型的会话，由（用于创建会话）configuration对象的类型确定：

- *Default sessions*默认会话，行为类似于其他用于下载的URL的Foundation方法。他们利用在用户的钥匙串中的持久化磁盘的缓存和存储证书。
- *Ephemeral sessions*临时会话不存储任何数据到磁盘; 所有高速缓存，证书存储等等被保存在RAM和绑在session。因此，当你的应用程序使session失效，它们会自动清除。
- *Background sessions*后台会话类似于默认会话，除了一个单独的进程可以处理所有的数据传输。背景会话有一些其他限制，所描述[Background Transfer Considerations](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW44)。

### 任务的类型

在一个会议上，`NSURLSession`类支持三种类型的任务：任务数据，下载任务，上传任务。

- *data task数据任务，用*`NSData`的对象来发送和接收数据。data task意为短暂，活跃地从您的应用程序服务器的请求。数据的任务可以接收到每个数据块之后返回给app，也可以一次性下载完成，然后通过 completion block返回。
- *download task下载任务*，以一个文件的形式接收数据，并且支持后台下载。
- *upload task上传任务*以一个文件的形式，并支持后台上传。

### 后台传输的注意事项

该`NSURLSession`支持后台传输，同时您的应用程序将被暂停。由session，使用background session configuration对象（可通过调用`backgroundSessionConfiguration:`返回）来提供后台传输。

后台会话，因为实际的传输是由一个独立的进程执行，因为重新启动应用程序的过程是相对昂贵，一些功能将无法使用，导致以下限制：

- session*必须*提供事件传递一个委托。（对于上载和下载，delegate在传输过程中的表现是一样的）
- 只有HTTP和HTTPS协议的支持（不定制协议）。
- 重定向始终遵循。
- 只有上传文件任务支持（当app退出后，上传数据或stream会失败）。
- 如果后台传输启动，而应用程序是在后台运行，配置对象的`discretionary`属性被视作为`true`。

**注：**  iOS 8和OS X 10.10之前版本，数据任务不支持后台会话。

您的应用程序的行为方式，在iOS和OS X启动之间略有不同。

在iOS中，当后台传输完成或者需要证书，如果您的应用程序不再运行，iOS后台会自动重新启动app，并调用`application:handleEventsForBackgroundURLSession:completionHandler:`。此调用提供了启动app的session标识符。您的应用程序应该存储completion block，创建有相同标识符的background configuration对象，并使用该configuration对象的会话。新session会自动与正在进行的后台活动重新关联。后来，当session结束的最后一个后台下载任务时，它会给session发送`URLSessionDidFinishEventsForBackgroundURLSession:`消息。在该delegate方法，在主线程调用之前存储的completion block，通知操作系统可以安全地再次暂停app。

在iOS和OS X中，当用户重启app，app应立即创建和app上一次运行时的session（具有大量任务的session）相同标识符的background configuration对象，然后为每个配置对象创建session。这些新session也同样会自动与正在进行的后台活动重新关联。

**注意：**  您必须根据每一个标识创建*完全相同*会话。多个会话共享相同标识符的行为是不确定的。

如果app挂起，而任务完成后，delegate的`URLSession:downloadTask:didFinishDownloadingToURL:`方法，会被任务及相关的新下载文件的URL调用。

同样，如果任务需要证书，则`NSURLSession`对象调用delegate的`URLSession:task:didReceiveChallenge:completionHandler:`方法或`URLSession:didReceiveChallenge:completionHandler:`方法。

网络错误后，后台会话的上传或下载任务，是通过URL loading system自动重试。它不需要使用reachability的API来确定何时重试失败任务。

有关如何使用一个例子`NSURLSession`后台传输，看*Simple Background Transfer*。

### 生命周期和代表互动

根据您使用`NSURLSession`来做什么，充分理解会话生命周期很有帮助，包括会话如何与委托互动，委托的调用顺序，当服务器返回一个重定向，当你的应用程序恢复失败的下载等等。

对于URL会话生命周期的完整说明，请参阅[Life Cycle of a URL Session](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1)。

### NSCopying行为

会话和任务对象符合`NSCopying`协议如下：

- 当您的应用程序copy会话或任务的对象，你会得到相同的对象回来。
- 当您的应用程序copy配置对象，你会得到一个新的副本，你可以独立修改。



## 创建和配置session

`NSURLSession`API提供了丰富的配置选项：

- 私人存储，支持cache，cookies，证书和协议可自定制于单个会话
- 身份验证，依赖于特定请求（task），或请求组（session）
- 通过URL上传和下载文件，鼓励元数据（网址和设置）进行数据分离（该文件的内容）
- 每个主机的最大连接数的配置
- 已触发的资源超时，如果整个资源不能在一定的时间下载
- 最小和最大的TLS版本的支持
- 自定义proxy词典
- 控制cookie策略
- 控制HTTP流水线行为

因为大多数设置都包含在一个独立的配置对象，可以重用常用的设置。当你实例化一个会话对象，您指定以下内容：

- configuration对象管辖该session的行为和其中的任务

- 可选的，delegate对象处理输入数据和处理针对session的其他事件和的任务，如服务器认证，确定资源负载请求是否应当被转换成下载等

  如果你不提供一个委托，`NSURLSession`对象使用系统提供的委托。通过这种方式，你可以很容易地使用`NSURLSession`代替使用现有代码`sendAsynchronousRequest:queue:completionHandler:`的便捷方法。

  **注意：**  如果您的应用程序需要执行后台传输，它必须提供自定义委托。

当您实例化session对象，则不能更改配置或委托，除非创建新的session。

清单1-2展示了如何创建常见暂时的背景会话的例子。

**清单1-2**   创建和配置会话



```objective-c
// Creating session configurations
NSURLSessionConfiguration *defaultConfiguration = [NSURLSessionConfiguration defaultSessionConfiguration];
NSURLSessionConfiguration *ephemeralConfiguration = [NSURLSessionConfiguration ephemeralSessionConfiguration];
NSURLSessionConfiguration *backgroundConfiguration = [NSURLSessionConfiguration backgroundSessionConfigurationWithIdentifier: @"com.myapp.networking.background"];
```

```objective-c
// Configuring caching behavior for the default session
NSString *cachesDirectory = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject;
NSString *cachePath = [cachesDirectory stringByAppendingPathComponent:@"MyCache"];
```

```objective-c
/* Note:
 iOS requires the cache path to be
 a path relative to the ~/Library/Caches directory,
 but OS X expects an absolute path.
 */
#if TARGET_OS_OSX
cachePath = [cachePath stringByStandardizingPath];
#endif
NSURLCache *cache = [[NSURLCache alloc] initWithMemoryCapacity:16384 diskCapacity:268435456 diskPath:cachePath];
defaultConfiguration.URLCache = cache;
defaultConfiguration.requestCachePolicy = NSURLRequestUseProtocolCachePolicy;

// Creating sessions
id <NSURLSessionDelegate> delegate = [[MySessionDelegate alloc] init];
NSOperationQueue *operationQueue = [NSOperationQueue mainQueue];
 
NSURLSession *defaultSession = [NSURLSession sessionWithConfiguration:defaultConfiguration delegate:delegate operationQueue:operationQueue];
NSURLSession *ephemeralSession = [NSURLSession sessionWithConfiguration:ephemeralConfiguration delegate:delegate delegateQueue:operationQueue];
NSURLSession *backgroundSession = [NSURLSession sessionWithConfiguration:backgroundConfiguration delegate:delegate delegateQueue:operationQueue];
```

随着背景的配置外，你可以重复使用会话配置对象来创建其他session。（不能重用background configuration，因为两个后台会话对象共享相同标识符的行为是未定义）

您也可以在任何时间安全地修改配置对象。当你创建一个会话，会话执行配置对象的深层副本，因此修改只影响新会话，不影响旧会话。例如，您可以创建新的session，只有当你使用的是Wi-Fi才检索内容，如清单1-3所示。

**清单1-3**   创建具有相同的配置对象的第二个session

```objective-c
ephemeralConfiguration.allowsCellularAccess = NO;
NSURLSession *ephemeralSessionWiFiOnly = [NSURLSession sessionWithConfiguration:ephemeralConfiguration delegate:delegate delegateQueue:operationQueue];
```

## 使用系统自带的delegate获取资源

`NSURLSession`用最直接的方法是使用系统提供的delegate请求资源。使用这种方法，你只需要在app提供两类代码

- 创建configuration对象，以及基于该configuration的session
- completion 处理路线

使用系统提供的delegate，您可以用几行代码就可以获取某个URL。list 1-4所示示例

**注：**  系统提供的委托提供的网络行为只进行了有限的定制。如果您的应用程序已经超出基本的URL抓取特殊需求，如自定义身份验证或后台下载，这种技术是不恰当的。对于必须在其中实现一个完整的代表，看到情况的完整列表的URL会话生命周期。

**清单1-4**   使用系统提供的delegate请求资源

```objective-c
NSURLSession *sessionWithoutADelegate = [NSURLSession sessionWithConfiguration:defaultConfiguration];
NSURL *url = [NSURL URLWithString:@"https://www.example.com/"];
 
[[sessionWithoutADelegate dataTaskWithURL:url completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
    NSLog(@"Got response %@ with error %@.\n", response, error);
    NSLog(@"DATA:\n%@\nEND DATA\n", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
}] resume];
```



## 使用自定义代理获取数据

如果您使用的是自定义delegate，委托必须至少实现以下方法：

- `URLSession:dataTask:didReceiveData:` 分块接收数据。
- `URLSession:task:didCompleteWithError:`表明任务，该数据传输完成。

如果当`URLSession:dataTask:didReceiveData:`方法返回后，您的应用程序需要使用数据时，你的代码是负责存储这些数据。

例如，网络浏览器可能需要呈现数据，因为它跟随着先前已接收到的数据。要做到这一点，它可能会使用该任务对象映射到一个字典`NSMutableData`对象，用于存储结果，然后使用`appendData:`该对象上的方法来追加新接收的数据。

清单1-5显示了如何创建和启动一个数据的任务。

**清单1-5**   数据任务的例子

```objective-c
NSURL *url = [NSURL URLWithString: @"https://www.example.com/"];
NSURLSessionDataTask *dataTask = [defaultSession dataTaskWithURL:url];
[dataTask resume];
```

## 下载文件

更深层，下载文件是类似于接收数据。APP应实现以下的委托方法：

- `URLSession:downloadTask:didFinishDownloadingToURL:` 提供URL的下载内容，临时存储的文件。

  **重要说明：**  此方法返回之前，它必须是打开文件进行读取或将其移动到永久位置。当这个方法返回，如果临时文件仍然在原来的位置，会被删除。

- `URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:` 提供有关下载进度的状态信息。

- `URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:` 通知你的应用程序，试图恢复以前失败的下载成功了。

- `URLSession:task:didCompleteWithError:` 通知应用程序下载失败。

如果计划后台下载，当你的应用程序没有运行时下载会继续。如果下载是标准或临时会话，当您的应用程序重新启动时下载必须重新开始。

当从服务器传输，如果用户告诉应用程序暂停下载，APP可以通过调用`cancelByProducingResumeData:`的方法，取消该任务。后来，你的应用程序可以将返回的恢复数据传递给`downloadTaskWithResumeData:`或`downloadTaskWithResumeData:completionHandler:`方法，来创建新的下载任务，重试之前的下载。

如果传输失败，你委托的`URLSession:task:didCompleteWithError:`方法会被`NSError`对象调用。如果任务是可恢复，该对象的`userInfo`字典包含一个值`NSURLSessionDownloadTaskResumeData`密钥; 

list 1-6提供下载一个中等大小文件的例子。list 1-7提供的下载任务委托方法的例子。

**清单1-6**   下载任务的例子

```objective-c
NSURL *url = [NSURL URLWithString:@"https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/ObjC_classic/FoundationObjC.pdf"];
NSURLSessionDownloadTask *downloadTask = [backgroundSession downloadTaskWithURL:url];
[downloadTask resume];
```



**清单1-7**   为下载任务委托方法

```objective-c
- (void)URLSession:(NSURLSession *)session
      downloadTask:(NSURLSessionDownloadTask *)downloadTask
      didWriteData:(int64_t)bytesWritten
 totalBytesWritten:(int64_t)totalBytesWritten
totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
{
    NSLog(@"Session %@ download task %@ wrote an additional %lld bytes (total %lld bytes) out of an expected %lld bytes.\n", session, downloadTask, bytesWritten, totalBytesWritten, totalBytesExpectedToWrite);
}
 
- (void)URLSession:(NSURLSession *)session
      downloadTask:(NSURLSessionDownloadTask *)downloadTask
 didResumeAtOffset:(int64_t)fileOffset
expectedTotalBytes:(int64_t)expectedTotalBytes
{
    NSLog(@"Session %@ download task %@ resumed at offset %lld bytes out of an expected %lld bytes.\n", session, downloadTask, fileOffset, expectedTotalBytes);
}
 
- (void)URLSession:(NSURLSession *)session
      downloadTask:(NSURLSessionDownloadTask *)downloadTask
didFinishDownloadingToURL:(NSURL *)location
{
    NSLog(@"Session %@ download task %@ finished downloading to URL %@\n", session, downloadTask, location);
 
    // Perform the completion handler for the current session
    self.completionHandlers[session.configuration.identifier]();
 
   // Open the downloaded file for reading
    NSError *readError = nil;
    NSFileHandle *fileHandle = [NSFileHandle fileHandleForReadingFromURL:location error:readError];
    // ...
 
   // Move the file to a new URL
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSURL *cacheDirectory = [[fileManager URLsForDirectory:NSCachesDirectory inDomains:NSUserDomainMask] firstObject];
    NSError *moveError = nil;
    if ([fileManager moveItemAtURL:location toURL:cacheDirectory error:moveError]) {
        // ...
    }
}
```



## 上传主体内容

您的应用程序可以以三种方式提供HTTP POST请求主体内容：作为一个`NSData`对象，一个文件，或流。一般情况下，你的应用程序应该：

- 使用一个`NSData`对象，如果你的应用程序在内存是data，也没有理由去处理它。
- 使用文件，如果你上传的内容是存在磁盘上的文件，如果你正在做后台传输，或者如果将其写入到磁盘比较傲，以便它可以释放与数据相关的内存。
- 如果你正在通过网络接收数据使用的流。

不管你选择哪一种风格，如果你的应用程序提供了一个自定义session delegate，delegate应该实现`URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:`方法获取上传进度信息。

此外，如果您的应用程序使用流提供请求主体，它必须提供一个实现了一个自定义会话的委托`URLSession:task:needNewBodyStream:`方法，更详细地描述 [Uploading Body Content Using a Stream](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW23)。

### 上传正文内容使用的NSData对象

用`NSData`对象来上传主体内容，应用程序调用要么`uploadTaskWithRequest:fromData:`或者`uploadTaskWithRequest:fromData:completionHandler:`方法来创建一个上传任务，并通过`fromData`参数提供请求体数据。

session对象计算`Content-Length`基于数据对象的大小标头。

你的应用必须提供一些头部信息，例如该服务器可能需要content type。

**清单1-8**   上传data

```objective-c
NSURL *textFileURL = [NSURL fileURLWithPath:@"/path/to/file.txt"];
NSData *data = [NSData dataWithContentsOfURL:textFileURL];
 
NSURL *url = [NSURL URLWithString:@"https://www.example.com/"];
NSMutableURLRequest *mutableRequest = [NSMutableURLRequest requestWithURL:url];
mutableRequest.HTTPMethod = @"POST";
[mutableRequest setValue:[NSString stringWithFormat:@"%lld", data.length] forHTTPHeaderField:@"Content-Type"];
[mutableRequest setValue:@"text/plain" forHTTPHeaderField:@"Content-Type"];
 
NSURLSessionUploadTask *uploadTask = [defaultSession uploadTaskWithRequest:mutableRequest fromData:data];
[uploadTask resume];
```



### 使用文件上传主体内容

要上传文件的主体内容，你的应用程序调用要么`uploadTaskWithRequest:fromFile:`或者`uploadTaskWithRequest:fromFile:completionHandler:`方法来创建一个上传任务，并会提供一个文件的URL，task可读取其内容。

session对象根据data对象的大小来计算`Content-Length`。如果您的应用程序不为`Content-Type`头部提供值，session也会提供一个。

您的应用程序可以提供任何附加头信息，服务器可能需要作为URL请求对象的一部分。

**清单1-9**   从流请求示例上传任务

```objective-c
NSURL *textFileURL = [NSURL fileURLWithPath:@"/path/to/file.txt"];
 
NSURL *url = [NSURL URLWithString:@"https://www.example.com/"];
NSMutableURLRequest *mutableRequest = [NSMutableURLRequest requestWithURL:url];
mutableRequest.HTTPMethod = @"POST";
 
NSURLSessionUploadTask *uploadTask = [defaultSession uploadTaskWithRequest:mutableRequest fromFile:textFileURL];
[uploadTask resume];
```



### 上传正文内容使用流

要上传使用流的主体内容，您的应用程序调用`uploadTaskWithStreamedRequest:`创建上传任务的方法。您的应用程序提供了与相关的流从该任务读取体内容的请求对象。

你的应用必须提供的任何附加头部信息，该服务器可能需要内容类型和长度，例如，作为URL请求对象的一部分。

此外，由于会话不能重放流以重新读取数据，应用程式是负责在该会话必须重试的请求（例如，如果身份验证失败）的情况下，提供一个新的流。要做到这一点，您的应用程序提供了一种`URLSession:task:needNewBodyStream:`方法。当该方法被调用时，您的应用程序应该执行任何需要的行动来获得或创建一个新的主体流，然后调用与新的流提供完成completion block。

**注：**  由于您的应用程序必须提供一个`URLSession:task:needNewBodyStream:`委托方法，如果它通过流提供了身体，这种技术是使用系统提供的委托不兼容。

**清单1-10**   上传流

```objective-c
NSURL *textFileURL = [NSURL fileURLWithPath:@"/path/to/file.txt"];
 
NSURL *url = [NSURL URLWithString:@"https://www.example.com/"];
NSMutableURLRequest *mutableRequest = [NSMutableURLRequest requestWithURL:url];
mutableRequest.HTTPMethod = @"POST";
mutableRequest.HTTPBodyStream = [NSInputStream inputStreamWithFileAtPath:textFileURL.path];
[mutableRequest setValue:@"text/plain" forHTTPHeaderField:@"Content-Type"];
[mutableRequest setValue:[NSString stringWithFormat:@"%lld", data.length] forHTTPHeaderField:@"Content-Type"];
 
NSURLSessionUploadTask *uploadTask = [defaultSession uploadTaskWithStreamedRequest:mutableRequest];
[uploadTask resume];
```



### 使用一个下载任务上传文件

要上传download task的主体内容，您的应用程序必须提供一个`NSData`物体或主体流作为`NSURLRequest`的一部分时，它会创建下载请求提供的对象。

如果您使用流数据，应用程序必须提供一个`URLSession:task:needNewBodyStream:`委托方法，在身份验证失败的情况下，提供一个新主体流。这种方法进一步描述于[Uploading Body Content Using a Stream](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/UsingNSURLSession.html#//apple_ref/doc/uid/TP40013509-SW23)。

下载任务行为就像data task，除了在该数据被返回到应用程序的方式。

## 处理身份验证和自定义TLS链验证

如果远程服务器返回一个状态代码,表示认证是必需的，如果认证需要一个连接盘问（如SSL客户端证书），`NSURLSession`会调用验证盘问delegate方法。

- 对于会话级challenges- `NSURLAuthenticationMethodNTLM`，`NSURLAuthenticationMethodNegotiate`，`NSURLAuthenticationMethodClientCertificate`，或`NSURLAuthenticationMethodServerTrust`     -     `NSURLSession`对象调用session delegate的`URLSession:didReceiveChallenge:completionHandler:`方法。如果您的应用程序不提供session delegate方法，`NSURLSession`对象调用task delegate的`URLSession:task:didReceiveChallenge:completionHandler:`方法来处理challenge。
- 对于非会话级的challenges（所有其他），`NSURLSession`对象调用task delegate的`URLSession:task:didReceiveChallenge:completionHandler:`方法来处理的挑战。如果您的应用程序提供了一个session delegate和需要处理的身份验证，那么您必须要么在task级别处理验证，要明确么提供每个会话处理的一个task level的处理程序。session delegate的`URLSession:didReceiveChallenge:completionHandler:`方法不会调用非会话级的challenge。

**注：**  Kerberos身份验证是透明的处理。

当有一个基于流的上传主体任务，身份验证失败，任务不一定能够重放和安全地重用该流。取而代之的是，`NSURLSession`对象调用委托的`URLSession:task:needNewBodyStream:`委托方法以获得新的`NSInputStream`对象，它提供了主体数据的新的请求。

有关编写的认证委托方法的更多信息`NSURLSession`，请阅读[Authentication Challenges and TLS Chain Validation](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1).。

## 处理的iOS后台活动

如果您使用的是`NSURLSession`，下载完成后您的应用程序会自动重新启动。您的应用程序的`application:handleEventsForBackgroundURLSession:completionHandler:`委托方法，负责重新创建会话，存储completion，并调用 handler，当会话调用`URLSessionDidFinishEventsForBackgroundURLSession:`方法。

list 1-11提供了在后台创建和启动下载任务的一个例子。list 1-12和list 1-13展示的这些会议和应用程序的委托方法的例子。

**list 1-11**   会话后台下载任务

```objective-c
NSURL *url = [NSURL URLWithString:@"https://www.example.com/"];
 
NSURLSessionDownloadTask *backgroundDownloadTask = [backgroundSession downloadTaskWithURL:url];
[backgroundDownloadTask resume];
```



**清单1-12**   后台下载，会话代理方法

```objective-c
- (void)URLSessionDidFinishEventsForBackgroundURLSession:(NSURLSession *)session {
    AppDelegate *appDelegate = (AppDelegate *)[[[UIApplication sharedApplication] delegate];
    if (appDelegate.backgroundSessionCompletionHandler) {
        CompletionHandler completionHandler = appDelegate.backgroundSessionCompletionHandler;
        appDelegate.backgroundSessionCompletionHandler = nil;
        completionHandler();
    }
 
    NSLog(@"All tasks are finished");
}
```



**清单1-13**   后台下载，App delegate 方法

```objective-c
@interface AppDelegate : UIResponder <UIApplicationDelegate>
@property (strong, nonatomic) UIWindow *window;
@property (copy) CompletionHandler backgroundSessionCompletionHandler;
 
@end
 
@implementation AppDelegate
 
- (void)application:(UIApplication *)application
handleEventsForBackgroundURLSession:(NSString *)identifier
  completionHandler:(void (^)())completionHandler
{
    self.backgroundSessionCompletionHandler = completionHandler;
}
 
@end
```

