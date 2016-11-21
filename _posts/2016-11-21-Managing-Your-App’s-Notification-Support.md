---
layout: post
title: Managing Your App’s Notification Support
tags: 
- Notification

---

## 管理你的应用程序的通知支持

应用必须在启动时，被配置为支持本地和远程通知。具体而言，您必须事先配置你的应用程序，如果这样做以下任何一项：

- 显示alert，badge和声音。
- 显示自定义操作按钮。

通常情况下，您的应用程序完成启动前完成所有的配置。在iOS和tvOS，这意味着配置您的通知支持，在`application:didFinishLaunchingWithOptions:`委托之前。在watchOS，最迟配置支持比`applicationDidFinishLaunching`委托。您可以在以后执行此配置，但你必须避免调度任何本地或远程通知，直到配置完成。

支持远程通知的应用需要额外的配置，详见[Configuring Remote Notification Support](https://developer.apple.com/library/prerelease/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1)。

### 请求授权与用户进行互动

在iOS中，tvOS和watchOS，应用程序必须有授权，以显示alert，播放声音，或badge。请求授权的权力在用户手中，用户可以授予或拒绝你的授权请求。用户还可以之后在系统设置，更改应用的权限设置，。

请求授权，调用`requestAuthorizationWithOptions:completionHandler:`方法，来共享`UNUserNotificationCenter`对象。如果您的应用授权已获得所有请求类型的授权，系统将调用completion handler block ，并将`granted`参数设置为`YES`。如果一个或多个交互类型被拒绝，参数`NO`。清单2-1显示了如何请求授权播放声音和显示alert。使用完成处理程序块根据交互类型是否被允许或拒绝，来更新应用程序的行为。。

**清单2-1**  请求用户交互授权

```objective-c
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert + UNAuthorizationOptionSound)
   completionHandler:^(BOOL granted, NSError * _Nullable error) {
      // Enable or disable features based on authorization.
}];
```



您的应用程序启动并调用了第一次`requestAuthorizationWithOptions:completionHandler:`的方法，系统会提示用户授予或拒绝请求的相互作用。由于系统节省了用户的响应，调用此方法在随后的发布会不要再提示用户。

注意用户可以使用系统设置随时更改授权交互类型为您的应用程序。为了准确确定哪些类型，你可以使用互动，调用`getNotificationSettingsWithCompletionHandler:`的方法`UNUserNotificationCenter`。

### 配置categories和actionable通知

*Actionalbe Notifications*（可交互的通知）给用户提供快速简便的方法执行有关方法，以响应通知。一个可操作的通知界面，显示可点击的自定义操作按钮，代替强迫用户启动应用程序。当用户点击时，每个按钮会移除通知的界面，并转发选择的动作给你的应用程序，方便立即处理。转发动作给你的应用，避免了需要用户深层浏览您的应用程序来执行操作，从而节省时间。

应用程序必须明确地添加对*Actionalbe Notifications*支持。在启动时，应用程序必须注册一个或多个catogories，它定义了您的应用程序发出通知的类型。与每个类别相关联的是action，当该类型的一个通知被发送的用户可以执行的操作。每个类别最多可以有4个相关的动作，虽然实际显示动作的数量取决于如何以及在何处显示通知。例如，横幅显示不超过两个action。

注意 *Actionalbe Notifications* 只在iOS和watchOS支持。

### 注册 Categories

Categories定义您的应用支持的通知类型，并与系统沟通要如何呈现一个通知。使用类别，可以关联与通知自定义action，并指定如何处理该类型通知的选项。例如，您可以使用类别选项，指定通知是否在CarPlay环境中显示。

在启动时，使用`setNotificationCategories:`，一次性注册你的应用所需的所有category。调用该方法之前，您需要创建一个或多个`UNNotificationCategory`实例，并指定category的名称和选项，这是在显示该类型的通知时使用。类别名称是在您的应用程序的内部，且永远不会被用户看到。调度通知时，您可以在通知的有效内容中包含类别名称，系统随后会使用该名称检索选项并显示通知。。

清单 2-2  展示了如何创建一个简单的`UNNotificationCategory`对象，并使用它注册系统。这个类别的名称为“GENERAL”，并配置了一个自定义的消失动作的选项，这将导致系统在用户关闭通知界面时通知应用程序，并且不采取任何其他动作。

**清单2-2**

```objective-c
UNNotificationCategory* generalCategory = [UNNotificationCategory
     categoryWithIdentifier:@"GENERAL"
     actions:@[]
     intentIdentifiers:@[]
     options:UNNotificationCategoryOptionCustomDismissAction];
 
// Register the notification categories.
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center setNotificationCategories:[NSSet setWithObjects:generalCategory, nil]];
```



你无需为每个通知安排一个category。但是，如果不使用类别来显示您的通知，通知没有任何自定义操作或配置选项。

### category添加自定义操作

您注册的每个category最多包含四个自定义action。当一个category包含自定义action时，系统会在通知界面中添加按钮，每个按钮都有一个自定义action的标题。如果用户点击某个您的自定义action，系统会发送相应的action标识符到你的应用程序中，根据需要启动您的应用。

要定义自定义操作，创建`UNNotificationAction`对象，并将其添加到您的一个category对象。每个action包含相应按钮的标题字符串，并为如何显示按钮和处理相关任务的选项。当用户选择一个action，该系统提供了该action的标识符字符串，然后您可以使用该字符串来标识要执行的任务。 清单2-3 扩展示例清单2-2，添加有两个自定义操作的新类别。

**清单2-3** 

```objective-c
UNNotificationCategory* generalCategory = [UNNotificationCategory
      categoryWithIdentifier:@"GENERAL"
      actions:@[]
      intentIdentifiers:@[]
      options:UNNotificationCategoryOptionCustomDismissAction];
 
// Create the custom actions for expired timer notifications.
UNNotificationAction* snoozeAction = [UNNotificationAction
      actionWithIdentifier:@"SNOOZE_ACTION"
      title:@"Snooze"
      options:UNNotificationActionOptionNone];
 
UNNotificationAction* stopAction = [UNNotificationAction
      actionWithIdentifier:@"STOP_ACTION"
      title:@"Stop"
      options:UNNotificationActionOptionForeground];
 
// Create the category with the custom actions.
UNNotificationCategory* expiredCategory = [UNNotificationCategory
      categoryWithIdentifier:@"TIMER_EXPIRED"
      actions:@[snoozeAction, stopAction]
      intentIdentifiers:@[]
      options:UNNotificationCategoryOptionNone];
 
// Register the notification categories.
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center setNotificationCategories:[NSSet setWithObjects:generalCategory, expiredCategory,
      nil]];
```



虽然你可以为每个类别指定了四种自定义action，系统可能会在某些情况下只显示前两个action。例如，系统显示在横幅的通知时仅显示两个动作。当你初始化`UNNotificationCategory`的对象，始终配置action数组，以便最相关的action在数组前列。

如果使用`UNTextInputNotificationAction`配置操作，系统为配置操作提供了一种方法，输入文字作为通知响应的一部分。文字输入操作是对于从用户收集自由形式的文本很有用。例如，一个社交应用可以允许用户提供对消息的定制响应。当文本输入动作被传递到您的应用程序进行处理时，系统将用户的响应封装到了`UNTextInputNotificationResponse`对象。

有关如何处理自定义操作的选择信息，请参阅[Responding to the Selection of a Custom Action](https://developer.apple.com/library/prerelease/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW2)。

### 准备自定义警告声音

本地和远程通知，可以指定在发送通知时播放自定义警报声音。 您可以将音频数据打包到aiff，wav或caf文件中。 由于它们由系统声音设施播放，自定义声音必须采用其中的音频数据格式：

- 线性PCM
- MA4（IMA / ADPCM）
- μLaw
- aLaw

将自定义的声音文件在您的应用程序软件包或应用程序的容器目录的`Library/Sounds`文件夹中。播放时自定义声音必须在30秒以下。如果自定义声音超过该限制，则会播放系统默认的声音。

您可以使用`afconvert`工具转换声音。例如，在CAF文件，将16位线性PCM系统声音`Submarine.aiff`，转换为IMA4音频，在终端应用程序中使用下面的命令：

$ `afconvert /System/Library/Sounds/Submarine.aiff ~/Desktop/sub.caf -d ima4 -f caff -v`

有关如何将声音文件与通知相关联的信息，请参阅[Adding a Sound to the Notification Content](https://developer.apple.com/library/prerelease/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW3)。

### 管理你的应用程序的通知设置

因为用户可以随时改变你的应用程序的通知设置，您可以使用`getNotificationSettingsWithCompletionHandler:`方法，随时您的应用程序的授权状态。该方法返回一个`UNNotificationSettings`对象，其内容反映了您的应用程序的当前授权状态和当前的通知环境。

使用`UNNotificationSettings`对象中的信息，来调整应用通知的相关代码。你可以传输你的应用程序的alert，badge，和声音授权设置到您的服务器，以便您的服务器可以调整它包括任何远程通知的选项。您可以使用其他设置来调整你如何调度和配置通知。有关可用设置的详细信息，请参阅*UNNotificationSettings Class Rerference*。

### 管理发送的通知

当本地和远程通知没有被你的应用程序或用户立即处理，它们会显示在通知中心，使他们能够在以后查看。使用`getDeliveredNotificationsWithCompletionHandler:`来获得仍显示在通知中心的通知列表。如果你发现已经过时，不应显示给用户的通知，您可以使用`removeDeliveredNotificationsWithIdentifiers:`方法删除它们。