---
layout: post
title: APNs Overview
tags: 
- Apple Push Notification Service
---

## APNs概述

苹果推送通知服务（APNs）是远程通知功能的核心。它是一种强大和高效的服务，用于将信息传播到iOS（以及间接传递到watchOS），tvOS和macOS设备。 在首次启动时，您的应用程序将通过用户设备与APNs建立认证和加密的IP连接。 随着时间的推移，APNs使用这个持久连接来提供通知。 如果在您的应用未运行时收到通知，设备会收到通知，并在适当的时间处理给应用的通知投放。

除了APNs和您的应用程序，还需要远程通知的推送。 您必须配置自己的服务器，才能发出这些通知。 您的服务器，称为提供者provider，具有以下职责：

- 它从您的应用程序接收`device token`设备令牌和相关数据。
- 它确定远程通知发送到设备的时间。
- 它与APNs通信通知的数据，这样APNs可以处理将通知传送到设备。

对于每个远程通知，您的提供者：

1. 用通知有效载荷`payload`来构造一个JSON字典;详见 [Creating the Remote Notification Payload](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW1)所述。
2. 将有效负载`payload`和相应的设备令牌链接到`HTTP/2`请求。
3. 通过使用`HTTP/2`网络协议的持久和安全通道将请求发送到APNs。

对于每个成功的`HTTP/2`请求，APN将相应的通知有效载荷转发到用户的设备。该设备处理收到的有效负载，并管理与用户的交互和有效载荷交付到您的应用程序。有关`HTTP/2`请求格式以及来自APNs的可能响应和错误的信息，请参阅[Communicating with APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW1)。

### 远程通知的路径

提供者负责发送与您的应用相关的远程通知。当您决定此时发送通知，您的提供者会将通知的数据打包到JSON字典中，该字典是构成要传送到设备的有效数据`payload`（通知数据→json字典→payload）。您将该有效数据打包到HTTP / 2请求中，该请求包含了用于传递通知的设备令牌和其他信息。然后，您的提供商将该请求转发给APNs，APNs将有效数据`payload`传到相应的用户设备。图6-1显示了传送过程的路径。

payload：HTTP请求的有效数据，核心数据，请求header后面的数据，是一次请求的主要数据，

**图6-1**  从提供者到应用程序推送通知![图片：../Art/remote_notif_simple.jpg](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/remote_notif_simple_2x.png)

每个请求都会包含设备令牌`device token`，代表接收通知的设备标识。 APNs使用设备令牌`device token`来标识每个唯一的应用和设备组合。 它还使用它们来验证发送到设备的远程通知的路径。 每当应用在设备上运行时，它都会从APNs获取此设备令牌`device token`并将其转发给您的提供商。 您的提供商存储设备令牌`device token`，并在向特定应用和设备发送通知时使用它。 设备令牌`device token`本身是不透明和持久的，只有当设备的数据和设置被擦除时才会改变。 只有APNs可以解码和读取`device token`。

图6-2 描述了APNs在多个提供商和设备之间启用的虚拟网络的类型。 要处理应用程序的通知负载，您通常会使用多个提供程序，每个提供程序都有自己与APNs的持久和安全连接。 然后，每个提供商可以将通知按路线发送到其具有有效`device token`的设备。

**图6-2**  来自多个供应商的多个设备推远程通知![图片：../Art/remote_notif_multiple.jpg](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/remote_notif_multiple_2x.png)



有关如何请求设备令牌信息，请参阅 [Device Token Generation and Dispersal](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW14)。有关如何创建通知有效载荷的信息，请参阅[Creating the Remote Notification Payload](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW1)。

### 服务器性能，存储转发，和聚合推送

Apple推送通知服务，默认包括执行存储转发功能的服务质量（QoS）组件。如果APNs尝试推送通知，但是目标设备处于离线，APN会在一定时间内存储通知，并在设备再次连接时传递通知。此组件仅存储每个设备和每个应用程序的最新通知。如果设备离线，发送新通知会导致先前的通知被丢弃。如果设备长时间保持离线状态，则所有存储的通知都将被丢弃。

要允许聚合同类通知，您可以在通知请求中添加聚合标识符`collapse identifier`。通常，当设备处于联网状态时，您发送的每个通知都会传送到设备。但是，当`apns-collapse-id`键存在于您的`HTTP/2`请求头中时，APNs将合并键值相同的请求。例如，发送相同标题两次的新闻服务，可以对两个通知请求使用相同的聚合标识符值。 APNs然后将这两个请求合并，作为推送到设备的单个通知。

### 安全架构

为了确保通信安全，APNs服务器使用连接证书，证书颁发机构（CA）证书和加密密钥（私人和公共）,来验证与提供商和设备的连接和身份。 APNs使用两个信任级别来调节提供者和设备之间的入口点：连接信任`connection trust`和设备令牌信任`device token trust`。

连接信任确定了：APNs连接到公司所拥有的授权提供商，是由Apple已同意为其传送通知。您必须采取措施，以确保您的提供商服务器和APNs之间存在连接信任`connection trust`，如本节所述。 APN还使用与每个设备的连接信任`connection trust`，以确保设备的合法性。与设备的连接信任由APNs自动处理。

设备令牌信任`device token trust`确保：通知仅在合法的开始点和结束点之间推送。设备令牌是分配给特定设备上的特定应用程序的不透明的唯一标识符。每个应用实例在注册APNs时接收其唯一令牌。应用程序必须与其提供者共享此令牌，以便提供者可以将令牌加入到与APNs的通信中。设备令牌在通知请求中的存在确保：通知仅被递送到其预期的唯一应用设备组合。

> 重要
>
> 为了保护用户隐私，请勿使用设备令牌来标识用户设备。 当用户更新操作系统以及擦除设备的数据和设置时，设备令牌会更改。 因此，应用应始终在启动时请求当前设备令牌。 有关详细信息，请参阅[Device Token Generation and Dispersal](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW14)。

### Provider-to-APNs连接信任

有两种方案，供您提供商的服务器和苹果推送通知服务之间协商连接信任：

- **基于token的信任连接**  ：`provider`，通过基于`HTTP/2`的API，使用一个*JSON web tokens*（JWT），来验证`provider`和APNs连接。在这个方案中，`provider`不需要证书加私钥建立连接。 相反，您提供由Apple保留的公钥和自己存储的私钥。 然后，`provider`使用私钥生成和签名`JWT authentication tokens`。每个推送请求中都必须包含`authentication tokens`。
- **基于证书的信任连接**  ：另外，`provider`可以采用了唯一的*供应商认证和私有密钥*。建立推送服务时，提供者证书标识了提供者支持的主题。 每个主题是与您的某个应用程序相关联的捆绑ID。

### Token-Based Provider-to-APNs信托

[图6-3](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW3)  说明了使用基于`HTTP/2`架构的APN提供者API，来建立信任，并且使用JWT提供者认证令牌，来发送通知。其工作原理如下：

有关此步骤的提供程序可以接收的响应的详细信息，请参阅来自APN的`HTTP/2`响应。

1. 提供商要求使用传输层安全性（TLS）与APNs安全连接，如图中标记为“TLS initiation”的箭头所示。

2. 然后，APNs向您的提供商提供APNs证书，由图中下一个箭头（标记为“APNs certificate”）表示，您的提供商随后会验证该证书。

   此时，建立了连接信任，并且您的提供商服务器已启用向APN发送远程通知

3. 您的提供商发送的每个通知都必须附带JWT身份验证令牌，在图中表示为标记为“通知推送”的箭头。

4. APNs回复每个推送，在图中表示为标记为“`HTTP/2 response`”的箭头。

   关于你的供应商可以接收此步骤的响应细节，请参阅[HTTP/2 Response from APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW2).。

**图6-3**  建立和使用提供者和APNs之间的基于令牌的信任连接![图片：../Art/service_provider_ct.jpg](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/service_provider_ct_2x.png)

> 重要
>
> 为了与APN建立TLS会话，您必须确保在每个提供程序上安装了GeoTrust Global CA根证书。 如果程序运行在macOS，则此根证书默认位于钥匙串中。 在其他系统上，此证书需要显式安装。 您可以从这下载此证书[GeoTrust Root Certificates website](https://www.geotrust.com/resources/root-certificates/)。

基于`HTTP/2`的`provider`连接适用于传递到由证书中指定的主题（应用程序包ID）标识的特定应用程序。 根据您如何配置和配置APNs传输层安全（TLS）证书，受信任的连接也可以有效地将远程通知传递到与您的应用程序相关联的其他项目，包括Apple Watch复杂性和互联网语音协议（VoIP ） 服务。 即使这些项目在后台运行，APNs也会传递这些通知。 有关详细信息，请参阅[Communicating with APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW1)，请参阅 [Voice Over IP (VoIP) Best Practices](https://developer.apple.com/library/content/documentation/Performance/Conceptual/EnergyGuide-iOS/OptimizeVoIP.html#//apple_ref/doc/uid/TP40015243-CH30)。

APNs维护证书废除列表; 如果提供商的证书在废除列表上，APNs可以撤销提供商信任（即，APN可以拒绝TLS发起连接）。

### Certificate-Based Provider-to-APNs信托

[图6-4](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW12)显示了使用苹果公司颁发的证书，让提供者和的APNs之间建立信任。不同于[图6-3](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW3)，图中没有显示通知推送本身，而是在建立一个传输层安全（TLS）的连接停止。在基于证书的信任方案中，在基于证书的信任方案中，推送不被认证，但每个推送被验证。

基于证书的供应商到的APNs信任的工作原理如下：

1. `provider`要求使用传输层安全性（TLS）进行APNs安全链接，表示为图中的标有“TLS initiation”的箭头。

2. 随后，APNs给`provider`APNs证书，由图中（标有“APNs certificate”）的下箭头，然后验证。

3. 然后，您的提供者必须发送其苹果供应商证书回给APNs，图中为箭头标有“Provider certificate”。

4. 随后APNs验证您的供应商证书，从而确认连接请求来自合法的供应商，并建立您的TLS连接。

   此时，连接建立信任和您提供商服务器已启用发送推送通知到的APNs。

**图6-4**  建立供应商和APNs之间的基于证书的信任连接![图片：../Art/service_provider_ct_certificate_2x.png](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/service_provider_ct_certificate_2x.png)

与基于令牌的信任，基于证书的供应商到的APN连接有效交付只有一个特定的应用程序。该应用程序是由供应商证书中规定的话题（包ID）标识。

APNs维护一个证书撤销列表; 如果一个提供商的证书撤销列表上，APNs可以撤销提供商信任（即，APNs可以拒绝所述TLS起始连接）。

### APNs到设备连接信托

APNs以及各设备之间的信任作为本节所述的设备的操作系统在最初的设备激活自动建立（无需通过您的应用程序的参与）。

每个设备具有一个加密证书和私有加密密钥，在初始设备激活获得并存储在设备的钥匙串。在激活过程中，使用的APN证书和密钥使用对等网络身份验证，验证设备的连接，如图图6-5。

当设备操作系统启动用的APNs TLS连接，它返回它的服务器证书信任协商开始。操作系统验证该证书，然后发送设备的证书。最后的APN验证设备证书，建立信任。

**图6-5**建立设备和APNs之间的连接信任![图片：../Art/service_device_ct.jpg](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/service_device_ct_2x.png)

随着设备和APNs之间建立TLS连接，可以APNs然后提供一个应用程序特定的设备令牌给注册了远程通知每个应用程序。

### 设备令牌生成和扩散

每一个应用程序启动时，它必须与系统中注册中所述接收远程通知，[注册接收远程通知](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW3)。注册成功后，应用程序收到设备令牌其供应商发送通知到设备时使用。

当需要一个新的设备令牌的APNs利用包含该装置的证书中的信息该令牌。然后，它加密用记号密钥的令牌，并将其返回到设备，如图图6-6。该系统提供设备令牌返回到您的应用程序作为一个`NSData`对象。在收到此令牌，你的应用程序必须二进制或十六进制格式转发给您的提供商。您的供应商不能将通知发送到设备没有这个令牌。

**图6-6**管理设备令牌![图片：../Art/token_generation.jpg](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/token_generation_2x.png)

重要的APNs设备令牌是可变长度的。不要硬编码它们的大小。

### 通知的设备令牌信任

您的供应商发送到APNs的每个通知，必须包含设备令牌来识别接收设备。APNs使用令牌密钥解密`device token`，以确保通知源(提供者)的有效性。APNs使用包含在设备令牌的设备ID，来确定目标设备的身份。然后，它发送通知给该设备，如图图6-7。

**图6-7**识别使用该设备令牌的设备![图片：../Art/token_trust.jpg](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/token_trust_2x.png)

### 配置程序

通过iOS应用商店，tvOS的App Store和Mac App Store的发布的应用程序以及企业应用程序，APNs都是可用的。程序必须是配置和代码签名，才能使用APNs。如果你在公司工作，大部分的配置步骤只能由团队代理人或管理员进行。

有关如何配置你的推送通知的支持，搜索“Configure push notifications” 中的[Xcode help](http://help.apple.com/xcode/mac)。
