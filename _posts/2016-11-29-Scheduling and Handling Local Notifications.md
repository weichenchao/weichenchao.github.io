---
layout: post
title: Scheduling and Handling Local Notifications
tags: 
- Notification
---
## 调度和处理本地通知

本地通知可以让你在应用未执行时提醒用户。您可以在应用程序前台或后台运行时调度本地通知。 在调度通知之后，系统负责在适当的时间将通知传递给用户。 即使您的应用程序未运行，系统也可传递通知。

如果您的应用程序未运行或者处于后台，系统会直接向用户显示本地通知。 系统可以通过警报面板或横幅，声音或通过标记应用程序的图标来提醒用户。 如果您的应用提供了通知内容应用扩展程序，系统甚至可以使用您的自定义界面来提醒用户。 如果您的应用程序在通知到达时处于前台，则系统会为您的应用程序提供在内部处理通知的机会。

> 注意
>
> 本地通知只在iOS上，watchOS和tvOS支持。在MacOS上，应用程序不需要本地通知徽章的图标，播放声音，或显示警报，同时在后台运行。这些功能已经由了AppKit框架的支持。

### 配置本地通知

如下是用于配置本地通知的步骤：

1. 使用通知的详细信息，创建和配置`UNMutableNotificationContent`对象。
2. 创建`UNCalendarNotificationTrigger`、`UNTimeIntervalNotificationTrigger`或`UNLocationNotificationTrigger`对象，来描述发送通知的条件。
3. 用`content`和`trigger`信息，创建`UNNotificationRequest`。
4. 调用`addNotificationRequest:withCompletionHandler:`方法来调度通知。

在创建通知的内容时，请填写反映您希望与用户进行互动类型的UNMutableNotificationContent对象属性。例如，当您要显示alert时，请填写title和body属性。系统使用您提供的信息，来确定如何与用户交互。您还可以在处理已传递到您的应用程序的本地通知时使用此对象中的data。

创建通知的内容后，创建一个触发器对象，以定义何时发送通知。User Notifications framework提供基于时间和基于位置的触发器。使用所需条件来配置触发器，并使用该对象和您的内容来创建UNNotificationRequest对象。

清单3-1 显示了如何创建和配置与alery相关的本地通知。使用UNCalendarNotificationTrigger使通知在特定日期或时间递送，在该示例中，下一次时钟到达早上7:00。

**清单3-1**创建和配置本地通知OBJECTIVE-C的

```objective-c
UNMutableNotificationContent* content = [[UNMutableNotificationContent alloc] init];
content.title = [NSString localizedUserNotificationStringForKey:@"Wake up!" arguments:nil];
content.body = [NSString localizedUserNotificationStringForKey:@"Rise and shine! It's morning time!"
        arguments:nil];
 
// Configure the trigger for a 7am wakeup.
NSDateComponents* date = [[NSDateComponents alloc] init];
date.hour = 7;
date.minute = 0;
UNCalendarNotificationTrigger* trigger = [UNCalendarNotificationTrigger
       triggerWithDateMatchingComponents:date repeats:NO];
 
// Create the request object.
UNNotificationRequest* request = [UNNotificationRequest
       requestWithIdentifier:@"MorningAlarm" content:content trigger:trigger];
```



为UNNotificationRequest对象提供标识符，可以让您在调度本地通知后识别它们。您可以使用标识符，方便以后查找待处理的请求，或在发送之前取消它们。有关计划和取消请求的详细信息，请参阅[Scheduling Local Notifications for Delivery](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SchedulingandHandlingLocalNotifications.html#//apple_ref/doc/uid/TP40008194-CH5-SW5)。

将自定义action分配到本地通知

要在界面中显示本地通知的自定义action，请在配置期间，将一个已注册的类别标识符分配给UNMutableNotificationContent对象的categoryIdentifier属性。系统使用类别信息来确定要包括在通知界面中的哪些动作按钮（如果有的话）。您必须在调度通知请求之前，赋值此属性。

清单3-2显示了如何为本地通知指定类别标识符。在此示例中，“TIMER_EXPIRED”字符串表示在启动时定义的类别，包括两个自定义操作。注册此类别的代码如清单2-3所示。

**清单3-2**定义的动作的类别为本地通知

```objective-c
UNNotificationContent *content = [[UNNotificationContent alloc] init];
// Configure the content. . .
 
// Assign the category (and the associated actions).
content.categoryIdentifier = @"TIMER_EXPIRED";
 
// Create the request and schedule the notification.
```



有关如何与类别注册自定义操作的信息，请参阅 [Configuring Categories and Actionable Notifications](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/SupportingNotificationsinYourApp.html#//apple_ref/doc/uid/TP40008194-CH4-SW26)

### 添加声音到通知内容

如果希望本地通知在发送时播放声音，请为UNMutableNotificationContent对象的sound属性指定一个值。 您可以使用UNNotificationSound对象指定声音，这样您可以播放自定义声音或默认通知声音。 自定义声音必须位于用户设备的本地，才能播放。 将通知的声音文件存储在应用程序的主包中或下载它们，并将它们存储在应用程序容器目录的Library / Sounds子目录中。

要播放默认声音，创建声音文件并将其分配给您的通知内容。例如：

```objective-c
content.sound = [UNNotificationSound defaultSound];
```

当指定自定义声音，仅指定你想要播放的声音文件的文件名。如果系统找到与您提供的名称合适的声音文件

，发送通知时则会播放声音。如果系统没有找到合适的声音文件，则会播放默认的系统声音。

```objective-c
content.sound = [UNNotificationSound soundNamed:@"MySound.aiff"];
```



有关支持的声音文件格式的信息，请参阅*UNNotificationSound Class Reference*。

### 调度发送本地通知

要调度发送本地通知，请创建`UNNotificationRequest`对象，并调用`UNUserNotificationCente`r的addNotificationRequest：withCompletionHandler：方法。 系统异步调度本地通知，并在调度完成或发生错误时调用`completion handler block`。 清单3-3 显示了如何调度发送本地通知。 此示例中的代码完成了清单3-1中创建的通知的调度。

**清单3-3**   调度发送本地通知

```objective-c
// Create the request object.
UNNotificationRequest* request = [UNNotificationRequest
       requestWithIdentifier:@"MorningAlarm" content:content trigger:trigger];
 
UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
[center addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
   if (error != nil) {
       NSLog(@"%@", error.localizedDescription);
   }
}];
```



调度的本地通知保持活跃状态，直到系统取消调度或直到您明确取消这些通知为止。系统在发送后会自动取消调度通知，除非通知的触发器被配置为重复。要在发送单个通知之前取消它或取消重复通知，请调用UNUserNotificationCenter的removePendingNotificationRequestsWithIdentifiers：方法。要取消的通知必须具有UNNotificationRequest对象的标识符。要取消所有挂起的本地通知，无论它们是否具有请求标识符，请调用removeAllPendingNotificationRequests方法。

响应发送的通知

当您的应用程序未运行或处于后台时，系统会使用您指定的交互自动推送本地和远程通知。如果用户选择某个操作，或选择一个标准互动，系统会通知应用用户的选择。然后，您的代码可以使用该选择，来执行其他任务。如果您的应用程式正在前台运作，通知会直接传送到您的应用。然后，您可以决定是默默处理通知或是警告用户。

要响应通知的推送，您必须为共享的UNUserNotificationCenter对象实现一个委托。您的委托对象必须遵守UNUserNotificationCenterDelegate协议，通知中心将通知信息传递到您的应用程序。如果您的通知包含自定义操作，则必须要委托。

> 重要
>
> 在应用完成启动之前，您必须将delegate分配给共享`UNUserNotificationCenter`。如果不这样做，可能会导致你的应用程序无法正确处理通知。

关于如何实现你的委托对象的更多信息，请参阅*UNUserNotificationCenterDelegate Protocol Reference*。

### 当您的应用程序在前台，处理通知

如果在应用处于前台时收到通知，您可以将该通知设为静音，也可以让系统继续显示通知界面。默认情况下，系统会将系统在前台时收到通知时静音，直接将通知的数据传送给应用。您可以使用通知数据直接更新应用界面。例如，如果收到了新的体育比分，您只需在界面中更新该信息。

如果希望系统继续显示通知界面，请为`UNUserNotificationCenter`提供一个委托对象，并实现`userNotificationCenter：willPresentNotification：withCompletionHandler：`方法。此方法的实现仍应处理通知数据。完成后，使用系统需要的传递选项（如果有），来执行completion handler block。如果不指定任何选项，系统将静默通知。清单3-4显示了该方法的示例实现，它告诉系统播放声音。通知的有效载荷，会标识要播放的声音。

**清单3-4**   播放声音，而你的应用程序是在前台

```objective-c
- (void)userNotificationCenter:(UNUserNotificationCenter *)center
        willPresentNotification:(UNNotification *)notification
        withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler {
   // Update the app interface directly.
 
    // Play a sound.
   completionHandler(UNNotificationPresentationOptionSound);
}
```



当应用程序处于后台或未运行时，系统不会调用`userNotificationCenter：willPresentNotification：withCompletionHandler：`方法。在这些情况下，系统根据通知本身中的信息警告用户。您仍然可以使用`UNUserNotificationCenter`对象的`getDeliveredNotificationsWithCompletionHandler：`方法确定是否传递了通知。

### 响应自定义操作的选择

当用户从通知界面中选择自定义操作时，系统会通知应用程序用户的选择。自定义操作的响应打包在`UNNotificationResponse`对象中，并传递给应用程序的`UNUserNotificationCenter`共享对象的代理。要接收响应，您的委托对象必须实现`userNotificationCenter：didReceiveNotificationResponse：withCompletionHandler：`方法。您实现的此方法必须能够处理您的应用或extension的所有自定义操作。

如果您的应用程式或应用程式额外资讯在收到回应时并未执行，系统会在背景启动您的应用程式或应用程式额外资讯，以处理回应。使用提供的背景时间更新数据结构和应用界面，以反映用户的选择。不要使用时间来执行与自定义操作的处理无关的任务。

清单3-5显示了具有多个类别和自定义操作的计时器应用程序的响应处理程序方法的实现。实现使用action和categoryIdentifier属性来确定合适的操作过程。

**清单3-5**处理自定义通知操作

```objective-c
- (void)userNotificationCenter:(UNUserNotificationCenter *)center
           didReceiveNotificationResponse:(UNNotificationResponse *)response
           withCompletionHandler:(void (^)(void))completionHandler {
    if ([response.notification.request.content.categoryIdentifier isEqualToString:@"TIMER_EXPIRED"]) {
        // Handle the actions for the expired timer.
        if ([response.actionIdentifier isEqualToString:@"SNOOZE_ACTION"])
        {
            // Invalidate the old timer and create a new one. . .
        }
        else if ([response.actionIdentifier isEqualToString:@"STOP_ACTION"])
        {
            // Invalidate the timer. . .
        }
 
    }
 
    // Else handle actions for other notification types. . .
}
```



### 处理标准系统操作

在系统的通知界面中，用户可以显式地关闭通知界面或启动你的应用，而不是选择你的某个自定义操作。关闭界面包括点击适用的按钮或直接关闭该界面; 忽略通知或忽略通知栏并不代表一个明确的关闭。当系统操作被触发，用户通知中心将report传递给`userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:`方法的委托。传递给该方法的响应对象包含以下操作标识符之一：

- `UNNotificationDismissActionIdentifier` 让你知道用户在不选择自定义操作的情况下，关闭了通知界面。
- `UNNotificationDefaultActionIdentifier` 让你知道用户在不选择自定义操作的情况下，启动了应用程序。

您处理标准的系统操作和处理其他操作一样。清单3-6显示了一个模板`userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:`来检查这些特殊的动作方法。

**清单3-6**处理的标准体系行动

```objective-c
- (void)userNotificationCenter:(UNUserNotificationCenter *)center
          didReceiveNotificationResponse:(UNNotificationResponse *)response
          withCompletionHandler:(void (^)(void))completionHandler {
   if ([response.actionIdentifier isEqualToString:UNNotificationDismissActionIdentifier]) {
       // The user dismissed the notification without taking action.
   }
   else if ([response.actionIdentifier isEqualToString:UNNotificationDefaultActionIdentifier]) {
       // The user launched the app.
   }
 
   // Else handle any custom actions. . .
}
```

