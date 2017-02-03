---
layout: post
title: UISplitViewController
tags: 
- View Controller Catalog for iOS
---

`UISplitViewController`类是提供了一个主 - 从界面的容器视图控制器。在主 - 从界面中，主视图控制器的更改驱动从视图控制器（详细信息）进行更改。两个视图控制器可以被布置为三种情况：并排，每次只有一个是可见的，或者一个部分被覆盖，一个显示。在iOS中8及更高版本，可以在所有iOS设备使用`UISplitViewController`; 在iOS的以前版本中，该类仅在iPad上可用。

## 概述

在构建应用程序的用户界面时，分屏视图控制器通常是应用程序窗口的根视图控制器。分屏视图控制器没有自己的显着外观。它的大部分外观由您安装的子视图控制器定义。您可以配置子视图控制器，通过使用Interface Builder或代码方式，将视图控制器赋值给[`viewControllers`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623181-viewcontrollers)属性。子视图控制器可以是自定义视图控制器，也可以是其他容器视图控制器，如导航控制器。

> 注意
>
> 您不能将分屏视图控制器推送到导航堆栈。虽然可以在其他容器视图控制器中将分屏视图控制器安装为子控制器，但在大多数情况下不建议这样做。分割视图控制器通常安装在应用程序窗口的根目录下。有关如何实现界面的提示和指导下，详见[ iOS Human Interface Guidelines](https://developer.apple.com/ios/human-interface-guidelines/)。

分屏视图控制器基于可用空间确定其子视图控制器的布置。在水平regular的环境中，分屏视图控制器尽可能并列地呈现它的视图控制器。在水平compact的环境中，分屏视图控制器更像导航控制器，最开始显示主视图控制器，并根据需要pop或push从视图控制器。您也可以指定分屏视图控制器模式，通过分赋值[`preferredDisplayMode`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623170-preferreddisplaymode)属性。

当屏幕上显示，分屏视图控制器及其`delegate`对象一起来管理其子视图控制器的展示。`delegate`是一个对象，遵守[`UISplitViewControllerDelegate`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate)协议。使用该协议的方法，来在更改时自定义拆分视图界面的行为。有关协议的方法的详细信息，请参阅[`UISplitViewControllerDelegate`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate)。

### 配置拆分视图界面的外观

分割视图控制器的可视化配置由其当前显示模式控制。您不能直接设置显示模式; 相反，你使用[`preferredDisplayMode`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623170-preferreddisplaymode)属性间接设置。分割视图控制器尽力遵守您指定的显示模式，但由于空间限制，可能无法直观地适应该模式。如，分割视图控制器不能在水平compact的环境中并排显示其子视图控制器。

`Table 1`列出了可用的显示模式，并描述了如何在屏幕上布置视图控制器。该表还列出了用于请求指定显示模式的常量。

**表格1** 分割视图控制器的显示模式

| 模式             | 描述                                       |
| -------------- | ---------------------------------------- |
| side by side并排 | 两个视图控制器同时并排显示在屏幕上。主视图控制器显示在左侧，并且通常比次视图控制器窄。您可以使用[`preferredPrimaryColumnWidthFraction`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623183-preferredprimarycolumnwidthfract)属性，调节主视图控制器的宽度。当[`isCollapsed`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623185-iscollapsed)属性为[`true`](https://developer.apple.com/reference/swift/true)时，不适用此显示模式。在collapse折叠的界面中，一次只能显示一个视图控制器。此模式是由[`allVisible`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdisplaymode/1623190-allvisible)常量表示。 |
| hidden         | 辅助视图控制器显示在屏幕上，主视图控制器隐藏。要显示主视图控制器，必须需以模态方式显示或更改显示模式。此模式是由[`primaryHidden`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdisplaymode/1623205-primaryhidden)常量表示。 |
| overlay        | 辅助视图控制器在屏幕上，主视图控制器层叠在从视图控制器下面。在此模式下，辅助视图控制器部分遮盖主视图控制器。此模式是由[`primaryOverlay`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdisplaymode/1623195-primaryoverlay)常量表示。 |



设置`preferre display mode`后，分屏视图控制器更新本身，并在[`displayMode`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623194-displaymode)属性体现真实的显示模式。您可以随时更改`preferre display mode`，这使分屏视图控制器相应地进行调整。分屏视图控制器还设置`built-in gesture recognizer`，允许用户使用滑动来改变显示模式。您可以通过设置[`presentsWithGesture`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623171-presentswithgesture)属性为[`false`](https://developer.apple.com/reference/swift/false)，抑制这种手势识别。

[`displayModeButtonItem`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623196-displaymodebuttonitem)方法返回一个特殊的bar item，用于改变用户界面的显示模式。分割视图控制器管理此项目的行为和外观。所有你需要做的是将其添加到合适的导航栏或工具栏中。当点击它时，按钮发送action到分屏控制器，告诉它改变其当前的显示模式为`targetDisplayModeForAction(in:)`指定的模式。指定自动（或根本不实现委托方法）的显示模式，会导致分屏视图控制现适配当前Size Class。如，在纵向方向的iPad上，分割视图控制器在隐藏和重叠模式之间切换。

### 在拆分视图界面中更改子视图控制器

在设计分屏视图界面时，最好安装不会更改的主视图控制器和辅助视图控制器。常见的技术是在这两处安装导航控制器，然后根据需要push和pop新内容。拥有这些类型的锚点视图控制器可以更轻松地关注您的内容，并允许拆分视图控制器将其默认行为应用于整个界面。

当你确实需要更改主要或次要视图控制器的情况下，建议您一定要使用[`show(_:sender:)`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623199-show)和[`showDetailViewController(_:sender:)`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623182-showdetailviewcontroller)方法。使用这些方法（而不是直接修改`viewControllers`属性），可以让分屏视图控制器present指定的视图控制器的方式是当前显示最适合模式和size class。分屏视图控制器知道如何以更直观的方式调整界面。它甚至可以与其他容器视图控制器（如导航控制器）一起使用来呈现视图控制器。例如，在一个紧凑的环境中，主视图控制器是一个导航控制器，调用`showDetailViewController(_:sender:)`不会取代次视图控制器。相反，主导航控制器将视图控制器推送到其导航堆栈上。

### 折叠和展开分屏视图界面

当拆分视图控制器的size class在水平regular和水平紧凑之间切换时，执行折叠和展开转换。在这些转换期间，分割视图控制器改变它如何显示其子视图控制器。当从水平regular改变为水平compact时，分割视图控制器将一个视图控制器折叠到另一个视图控制器。当从水平compact回到水平regular时，它会再次展开界面，并根据显示模式显示其中一个或两个子视图控制器。

当转换到折叠的界面时，拆分视图控制器使用`delegate`来管理转换。在折叠转换结束时，拆分视图控制器通常仅显示主视图控制器的内容。您可以更改此行为，通过实现[`primaryViewController(forCollapsing:)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623197-primaryviewcontroller)的委托方法。您可以使用该方法指定辅助视图控制器或完全不同的视图控制器 - 也许更适合在水平compact环境中显示。如果要执行视图控制器和视图层次结构的其他调整，还可以实现[`splitViewController(_:collapseSecondary:onto:)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623184-splitviewcontroller)委托方法。

展开过程中，通过询问`delegate`指定哪个视图控制器成为主视图控制器并且给予`delegate`转换本身的机会，来逆向折叠过程。如果您实现了折叠的委托方法，你同时也应该实现[`primaryViewController(forExpanding:)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623188-primaryviewcontroller)和[`splitViewController(_:separateSecondaryFrom:)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623189-splitviewcontroller)方法，来展开界面。如果不实现任何方法，则分屏视图控制器提供默认行为来处理折叠和扩展转换。

有关使用管理折叠和展开方法的详细信息，请参阅[`UISplitViewControllerDelegate`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate)。

### 消息转发到子视图控制器

分屏视图控制器将自身插入在应用程序的窗口和其子视图控制器之间。因此，到子视图控制器的所有消息都必须通过分屏视图控制器。这通常在在预料中工作，消息的流应该是相对直观的。例如，仅当相应的子视图控制器实际出现在屏幕上时，才发送视图外观和消失的消息。

### 状态保存

在iOS 6之后，如果您赋值视图控制器的[`restorationIdentifier`](https://developer.apple.com/reference/uikit/uiviewcontroller/1621499-restorationidentifier)属性，它会保留有自己的有效恢复标识符中的所有子视图控制器。在下一启动周期期间，分割视图控制器将保留的视图控制器恢复到其先前状态。分割视图控制器的子视图控制器可以使用相同的恢复标识符。分割视图控制器自动存储附加信息，以确保每个子节点的恢复路径是唯一的。

有关状态保存和恢复如何工作的详细信息，请参阅[App Programming Guide for iOS](https://developer.apple.com/library/prerelease/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007072)。

## 符号

### 管理子视图控制器

[`var viewControllers: UIViewController`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623181-viewcontrollers)由接收器管理的视图控制器数组。

[`var presentsWithGesture: Bool`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623171-presentswithgesture)指定是否可以使用滑动手势显示和关闭隐藏的视图控制器。

### 管理显示模式

[`var preferredDisplayMode: UISplitViewControllerDisplayMode`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623170-preferreddisplaymode)分割视图控制器接口的优选布置。

[`var displayMode: UISplitViewControllerDisplayMode`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623194-displaymode)分割视图控制器的内容的当前布置。

[`var displayModeButtonItem: UIBarButtonItem`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623196-displaymodebuttonitem)改变分割视图控制器的显示模式的按钮。

### 获取拆分视图配置

[`var isCollapsed: Bool`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623185-iscollapsed)指示是否仅显示一个子视图控制器的布尔值。

[`var preferredPrimaryColumnWidthFraction: CGFloat`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623183-preferredprimarycolumnwidthfract)主视图控制器的内容的相对宽度。

[`var primaryColumnWidth: CGFloat`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623200-primarycolumnwidth)主视图控制器内容的宽度（以磅为单位）。

[`var minimumPrimaryColumnWidth: CGFloat`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623198-minimumprimarycolumnwidth)主视图控制器内容所需的最小宽度（以磅为单位）。

[`var maximumPrimaryColumnWidth: CGFloat`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623180-maximumprimarycolumnwidth)主视图控制器的内容允许的最大宽度（以磅为单位）。

### 访问委托对象

[`var delegate: UISplitViewControllerDelegate?`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623167-delegate)您想要接收分屏视图控制器消息的代理。

### 显示视图控制器的操作方法

[`func showDetailViewController(UIViewController, sender: Any?)`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623182-showdetailviewcontroller) 将指定的视图控制器呈现为辅助（从）视图控制器。

[`func show(UIViewController, sender: Any?)`](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1623199-show) 将指定的视图控制器呈现为主视图控制器。

### 常量

[`UISplitViewControllerDisplayMode`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdisplaymode)描述分割视图控制器的可能显示模式的常量。

[拆分视图控制器尺寸](https://developer.apple.com/reference/uikit/uisplitviewcontroller/1653393-split_view_controller_dimensions)常量，指示主视图控制器的默认宽度。

# UISplitViewControllerDelegate

### 响应显示模式的更改

[`splitViewController:willChangeToDisplayMode:`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623176-splitviewcontroller?language=objc)：通知`delegate`分屏视图控制器的显示模式即将改变。

[`targetDisplayModeForActionInSplitViewController:`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623192-targetdisplaymodeforactioninspli?language=objc)Asks the delegate to provide the display mode to apply when a split view controller action occurs.

[`func splitViewController(UISplitViewController, willChangeTo: UISplitViewControllerDisplayMode)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623176-splitviewcontroller)

[`func targetDisplayModeForAction(in: UISplitViewController)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623192-targetdisplaymodeforaction)在发生分割视图控制器action时，询问`delegate`提供显示模式以应用。

### 覆盖接口方向

[`func splitViewControllerPreferredInterfaceOrientationForPresentation(UISplitViewController)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623169-splitviewcontrollerpreferredinte) 询问`delegate`，呈现分割视图控制器时使用的方向。

[`func splitViewControllerSupportedInterfaceOrientations(UISplitViewController)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623178-splitviewcontrollersupportedinte) 询问`delegate`，指定拆分视图控制器支持的接口方向。

### 折叠和展开接口

[`func primaryViewController(forCollapsing: UISplitViewController)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623197-primaryviewcontroller) 询问`delegate`，提供单独的视图控制器作为主视图控制器，在拆分视图界面折叠后显示。

[`func splitViewController(UISplitViewController, collapseSecondary: UIViewController, onto: UIViewController)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623184-splitviewcontroller) 询问`delegate`，调整主视图控制器，并将辅助视图控制器合并到折叠的界面中。

[`func primaryViewController(forExpanding: UISplitViewController)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623188-primaryviewcontroller)询问`delegate`，提供视图控制器作为主视图控制器，在拆分视图界面展开时。

[`func splitViewController(UISplitViewController, separateSecondaryFrom: UIViewController)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623189-splitviewcontroller)询问`delegate`，为拆分视图界面，提供新的辅助视图控制器。

### 覆盖演示行为

[`func splitViewController(UISplitViewController, show: UIViewController, sender: Any?)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623168-splitviewcontroller)询问委托，如果它想要在分割视图界面的主要位置显示视图控制器。

[`func splitViewController(UISplitViewController, showDetail: UIViewController, sender: Any?)`](https://developer.apple.com/reference/uikit/uisplitviewcontrollerdelegate/1623204-splitviewcontroller)询问委托，如果它想要在分割视图接口的辅助位置显示视图控制器。
