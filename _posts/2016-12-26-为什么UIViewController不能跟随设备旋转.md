---
layout: post
title: Why won't my UIViewController rotate with the device?
tags:
- Rotation
---

## 问：为什么我的`UIViewController`不能跟随设备旋转？

答：`UIViewController`类为iOS应用程序提供了基本的视图管理模型。它自动支持旋转控制器的视图，以响应设备方向的改变。如果视图和子视图的`autoresizing`调整属性被正确配置，那么在大多数情况下，此行为是自动的。下面是一个相当详尽的列表，列出了视图控制器不能旋转的原因。

> **重要提示：** iOS 6引入了对自动旋转的巨大更改。这些更改仅适用于针对iOS 6 SDK进行链接的应用。未在iOS 6 SDK中构建的应用程序将在iOS 6设备上运行时收到旧的自动旋转行为。与最新SDK链接的应用程序应包括对旧自动旋转行为的支持（如果其部署目标小于iOS 6）。有关详细信息，请参阅[View Controller Programming Guide](https://developer.apple.com/library/ios/#featuredarticles/ViewControllerPGforiPhoneOS/RespondingtoDeviceOrientationChanges/RespondingtoDeviceOrientationChanges.html)

- 您已经将视图控制器的`UIView`属性作为子视图，添加到`UIWindow`。

  不建议将任何视图控制器的view属性添加为`UIWindow`的子视图。在`didFinishLaunchingWithOptions :`返回之前，应该在Interface Builder中或通过运行时，将应用程序的根视图控制器赋值给`app window's`的`rootViewController`属性。如果需要同时显示多个视图控制器的内容，您应该自定义容器视图控制器并将其用作根视图控制器。请参阅 [Creating Custom Container View Controllers](https://developer.apple.com/library/ios/redirect/DTS/CustomContainerVC)。

  > **注：**  请记住，`UINavigationController`和`UITabBarController`能够管理自己的堆栈或视图控制器列表。

  **从iOS 6开始**，如果根视图控制器尚未赋值给应用程序的窗口，支持的方向仅由确定的`UIApplication`对象。此外，视图控制器不会被通知的方向变化，这可能会导致异常。

- 视图控制器不支持新的方向。

  **从iOS 6开始**，视图控制器可以通过重写此方法限制其支持的方向：

  `- (NSUInteger)supportedInterfaceOrientations;`

  `supportedInterfaceOrientations`的默认实现返回一个方向，这个方向代表当前设备的惯用推荐方向。也就是说，`UIInterfaceOrientationMaskAll`在iPad设备上返回，`UIInterfaceOrientationMaskAllButUpsideDown`在iPhone设备被返回。除非视图控制器管理的内容只能在这些方向的一个子集来显示，才需要重写此方法。如果覆盖此方法，您的实现必须返回`UIInterfaceOrientationMask`值的按位组合。

  **在iOS 5及以下**，视图控制器可以通过重写这个方法支持多种方向：

  `- (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation;`

  即使实现`shouldAutorotateToInterfaceOrientation`，你应该确保它为所有你想支持的方位返回`YES`。如果`interfaceOrientation`等于`UIInterfaceOrientationPortrait`，默认实现返回`YES`，锁定视图控制器为竖屏。要支持所有的方向，总是返回`YES`。

- 应用程序不支持和视图控制器同一方向。

  **从iOS 6开始**，UIKit会同时使用`UIApplication`对象和最上面的视图控制器来确定支持的方向。在应用程序`Info.plist`的`UISupportedInterfaceOrientations`键必须包含您的应用支持每个方向的值。系统使用这些值来获得应用程序的支持方向掩码（按位组合）。应用程序委托可以实现`application:supportedInterfaceOrientationsForWindow:`，返回`UIInterfaceOrientationMask`，代替原来应用程序的`Info.plist中`的值。

  > **重要提示：**  通常，最上面的视图控制器`topmost controller`就是指窗口的根视图控制器，除非当前呈现了另一个控制器，在这种情况下，所呈现的视图控制器就成为最上面的视图控制器。最上面的视图控制器不应该与导航控制器`topViewController`混淆。

  **在iOS 5及以下**，在应用程序的`Info.plist`中`UISupportedInterfaceOrientations`键的值仅作为一个提示，它告诉Springboard应该应用程序启动之前，重新定位的状态栏。它不会在应用启动后限制其支持的方向。

- 视图控制器拒绝旋转。

  **从iOS 6开始**，当UIKit中确定可能需要进行旋转，它调用最顶层的视图控制器的方法`shouldAutorotate`，来确定是否继续选择。此方法默认返回`YES`; 然而，视图控制器可以通过重写`shouldAutorotate`以返回`NO`，在运行时动态禁用自动旋转。

- 视图控制器是另一个视图控制器的子级。

  **从iOS 6开始**，只有最上面topmost的视图控制器（沿着`UIApplication`对象）参与决定是否响应设备方向的改变而旋转。通常，显示您的内容的视图控制器是常用容器控制器的子级，如`UINavigationController`，`UITabBarController`或自定义容器视图控制器的子级。你可能会发现自己需要子类化容器视图控制器，以便修改其`supportedInterfaceOrientations`和`shouldAutorotate`方法的返回值。

  > **注意：**  在修改UIKit类（例如`UINavigationController`）的旋转行为时，应始终优先考虑子类化（创建`UINavigationController + category`）。因为其他类可能依赖于UIKit容器视图控制器的现有行为，category引入的更改可能会导致意外的行为。

- 视图控制器重写`init`或`initWithNibName:bundle:`但不调用父类的实现。为了正确初始化的对象，在如何初始化方法中则必须调用`super init`或`super initWithNibName`。

- `UITabBarController`还是`UINavigationController`的所有子视图控制器不要在一个共同的方向设定达成一致。

  **在iOS 5及以下的**，以确保所有的子视图控制器正常旋转，则必须实现代表每个选项卡或导航级的每个视图控制器的`shouldAutorotateToInterfaceOrientation`。每个都必须同意相同的方向，才会发生的旋转。也就是说，对于相同的方向，它们都应该返回`YES`。

- 该视图控制器以编程方式改变其子视图的布局。

  某些布局是不可能仅使用`autoresizing masks`就可以完成实现。视图控制器可以通过改变自己的`frame`，`bounds`或`center`属性，来手动布局其子视图。开发人员不应该把布局代码放到旋转方法，如由视图控制器重写的旋转方法`shouldAutorotateToInterfaceOrientation`。有些情况下，旋转发生了，但是并没有调用这些接口方法，从而使视图控制器的子视图处于不一致的状态。

  **如果应用程序只支持的iOS 5或更高版本**，你应该把视图控制器的布局代码放置在`viewWillLayoutSubviews:`或`viewDidLayoutSubviews:`方法，控制器方法。

  **如果应用程序必须支持的iOS 4**，应该将布局代码放置在`layoutSubviews`方法，此方法是视图控制器的view的方法。