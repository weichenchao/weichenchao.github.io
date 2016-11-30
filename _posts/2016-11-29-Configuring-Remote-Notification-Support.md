---
layout: post
title: Configuring Remote Notification Support
tags: 
- Notification
---

## 配置支持远程通知

即使应用程序未在用户的设备上运行，您也可以通过远程通知向应用程序的用户提供最新信息。 大多数实现远程通知的工作都在您公司的服务器上，但您还必须执行以下操作来为应用程序配置远程通知的支持：

1. 启用您的应用程序的推送通知功能。
2. 在您的应用程序的启动时间码中，注册APN。
3. 实现处理传入的远程通知的支持。

有关如何配置您的服务器，提供远程通知用户设备的信息，请参阅[的APN概述](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1)。

### 启用推送通知功能

处理远程通知的应用必须有与APNs通信的权利。你这可以使用Xcode项目的“功能”窗格将这些权利添加到应用程序中。

**任务**

1. 在Project Navigator中，选择您的项目。In the Project Navigator, select your project.

2. 在编辑器中，选择您的iOS应用的target。In the editor, select your iOS app target.

3. 选择功能选项卡。Select the Capabilities tab.

4. 启用推送通知的能力。 Enable the Push Notifications capability. 

   ![](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Art/push_notification_capability_2x.png)

Xcode中增加了所需的权利到您的项目。

没有正确授权的应用将在App Store的审查过程中被拒绝。在测试过程中，在没有正确的权利情况下，试图APN注册将返回错误。

### 注册接收远程通知

每次应用程序启动时，它必须注册APNs。APN注册方法因平台而异，但高级别过程如下：

1. 使用特定平台的API，来获得APN的设备令牌`device token`。
2. 发送设备令牌`device token`到服务器。

`device token`对于特定应用和设备是唯一的。收到`device token`后，您需要打开与服务器的网络连接，并转发`device token`和其他相关数据。在您的服务器上，使用`device token`和其他数据会为特定设备生成远程通知。向设备发送远程通知时，必须始终将`device token`放在发送到APN的数据内。

不要缓存`device token`;当你需要它们，从系统中获取他们。尽管`device token`对于应用和设备是唯一的，但它们也可以随时间改变。`device token`可以随时更改，除了当用户从备份中还原其设备时，用户在新设备上安装应用程序时以及用户重新安装操作系统时，`device token`可以保持不同。从系统获取`device token`，可确保您始终拥有与APN通信所需的当前令牌。此外，如果`device token`未更改，提取速度会很快，不会产生任何大的性能开销。

> 重要
>
> 当`device token`更改时，用户必须启动您的应用程序一次，然后才能将远程通知再次传送到该设备。

在watchOS运行应用程序不注册远程通知。手表应用依靠其配对的iPhone，将远程通知转发给Apple Watch显示。当iPhone被锁定（或屏幕处于睡眠状态）并且Apple Watch在用户的手腕上并解锁时，发生远程通知的转发。

有关远程通知的数据格式、以及如何将数据发送到APNs的信息[Communicating with APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CommunicatingwithAPNs.html#//apple_ref/doc/uid/TP40008194-CH11-SW1)。

### 获取iOS和tvOS设备令牌

在iOS和tvOS上，您可以通过调用`UIApplication`对象的`registerForRemoteNotifications`方法，来获取应用程序的`device token`。推荐您在启动时调用此方法，作为正常启动序列的一部分。您的应用程序首次调用此方法时，app对象会从APNs请求`device token`。在初始调用之后，只有当`device token`改变时，app对象才会重新连接APNs;否则，它会快速返回现有令牌。

应用对象在是否成功接收到`device token`时异步地通知其委托。您可以使用这些委托回调来处理`device token`或处理错误。您必须实现以下委派方法来跟踪注册是否成功。

- 使用`application:didRegisterForRemoteNotificationsWithDeviceToken:`接收`device token`并将其转发给您的服务器。
- 使用`application:didFailToRegisterForRemoteNotificationsWithError:`响应错误。

> 注意
>
> 如果`device token`在应用程序运行时发生更改，则应用程序对象再次调用相应的代理方法来通知您更改。

Listing 4-1  显示了如何获取iOS或tvOS应用的`device token`。应用程序委托调用`registerForRemoteNotifications`方法，作为其常规启动时设置的部分。收到`device token`后，应用程序：`didRegisterForRemoteNotificationsWithDeviceToken：`方法使用自定义方法将其转发到应用程序的关联服务器。如果在注册期间发生错误，应用程序会暂时禁用与远程通知相关的任何功能。当接收到有效的`device token`时，将重新启用这些功能。

Listing 4-1

```objective-c
- (void)applicationDidFinishLaunching:(UIApplication *)app {
    // Configure the user interactions first.
    [self configureUserInteractions];
 
   // Register for remote notifications.
    [[UIApplication sharedApplication] registerForRemoteNotifications];
}
 
// Handle remote notification registration.
- (void)application:(UIApplication *)app
        didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)devToken {
    // Forward the token to your server.
    [self enableRemoteNotificationFeatures];
    [self forwardTokenToServer:devTokenBytes];
}
 
- (void)application:(UIApplication *)app
        didFailToRegisterForRemoteNotificationsWithError:(NSError *)err {
    // The token is not currently available.
    NSLog(@"Remote notification support is unavailable due to error: %@", err);
    [self disableRemoteNotificationFeatures];
}
```



如果一个蜂窝或Wi-Fi连接不可用，既不调用`application:didRegisterForRemoteNotificationsWithDeviceToken:`方法，也不调用`application:didFailToRegisterForRemoteNotificationsWithError:`方法。对于Wi-Fi连接，这有时会发生在与APNs连接时，设备无法通过配置的端口。如果发生这种情况，用户可以通过移动到不阻塞所需端口的另一个Wi-Fi网络。在具有蜂窝无线电的设备上，用户也可以等到蜂窝数据服务可用。

在`application:didFailToRegisterForRemoteNotificationsWithError:`实现中，使用错误对象来禁用远程通知功能。由于通知无法到达，最好是适度地降低，避免任何需要的本地工作来促进远程通知。如果远程通知变为可用后，应用对象通过调用您的委托的`application:didRegisterForRemoteNotificationsWithDeviceToken:`的方法，来通知你。

### 获取MacOS的设备令牌

在iOS和tvOS上，您可以通过调用`NSApplication`对象的`registerForRemoteNotifications`方法，来获取应用程序的`device token`。推荐您在启动时调用此方法，作为正常启动序列的一部分。您的应用程序首次调用此方法时，app对象会从APNs请求`device token`。在初始调用之后，只有当`device token`改变时，app对象才会重新连接APNs;否则，它会快速返回现有令牌。

应用对象在是否成功接收到`device token`时异步地通知其委托。您可以使用这些委托回调来处理`device token`或处理错误。您必须实现以下委派方法来跟踪注册是否成功

- 使用`application:didRegisterForRemoteNotificationsWithDeviceToken:`接收`device token`并将其转发给您的服务器。
- 使用`application:didFailToRegisterForRemoteNotificationsWithError:`响应错误。

> 注意
>
> 如果`device token`在应用程序运行时发生更改，则应用程序对象再次调用相应的代理方法来通知您更改。



清单4-2

```objective-c
- (void)applicationDidFinishLaunching:(NSNotification *)notification {
    // Configure the user interactions first.
    [self configureUserInteractions];
 
    [NSApp registerForRemoteNotificationTypes:(NSRemoteNotificationTypeAlert | NSRemoteNotificationTypeSound)];
}
 
- (void)application:(NSApplication *)application
        didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    // Forward the token to your server.
    [self forwardTokenToServer:deviceToken];
}
 
- (void)application:(NSApplication *)application
        didFailToRegisterForRemoteNotificationsWithError:(NSError *)error {
    NSLog(@"Remote notification support is unavailable due to error: %@", error);
    [self disableRemoteNotificationFeatures];
}
```



### 处理远程通知

用户通知框架提供了代码路径，用于处理本地和远程通知相关的许多任务。具体来说，您可以使用相同的技术处理下列任务：

- 如果您的应用程序在前台，您可以直接接收通知并将其置为静默。
- 如果您的应用程序在后台或未运行：
  - 当用户选择使用的通知有关的自定义操作，您可以响应。
  - 当用户关闭通知或启动您的应用，进行回应。

除了通过用户通知框架的方法处理通知，您的应用程序也可以通过应用程序委托接收到远程通知的整个有效负载。当远程通知到达，且通常应用程序在后台时，系统处理用户的交互。它还提供了通知的有效载荷到`application:didReceiveRemoteNotification:fetchCompletionHandler:`。在MacOS上，有效载荷被传递给应用程序的委托方法`application:didReceiveRemoteNotification:`。您可以使用这些方法来检查有效载荷和执行关的任务。例如，当接收到静默的远程通知，你可能会开始下载你的应用程序的新内容。

有关如何处理使用用户通知框架的方法通知的信息，请参阅[响应通知的交付](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW14)。有关如何在您的应用程序委托处理通知信息，请参见*UIApplicationDelegate协议参考*或*NSApplicationDelegate协议参考*参考。