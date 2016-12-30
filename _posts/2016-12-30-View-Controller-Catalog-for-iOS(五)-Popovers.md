---
layout: post
title: View Controller Catalog for iOS(五)-Popovers
tags: 
- View Controller Catalog for iOS

---

# Popovers

虽然本身不是一个视图控制器，`UIPopoverController`类管理视图控制器的呈现。您可以使用`popover controller`对象展示当前内容，`popover`是一个浮动在你的应用程序的窗口上方的可视化图层。Popovers提供呈现或收集来自用户的信息的轻量的方式，并常用在以下情况：

- 要在屏幕上显示对象的信息
- 管理经常访问的工具或配置选项
- 呈现一个动作列表，来执行视图内某个对象
- 竖屏时，要从`split view controller`呈现一个窗格

对于上述操作，使用`popover`比模态视图会减少干扰和麻烦。在iPad应用程序，保留模态视图，是用于用户明确接受或取消某些动作或信息的情况。例如，你可以使用一个模态视图来询问用户，授予您的应用程序的其它部分访问的密码权限。至于其他大多数情况，你可以使用`popover`来代替。`popover`的好处是，他们不覆盖整个屏幕，可以通过简单地将`popover`视图外部范围被移除。因此，他们是在用户不需要与您的内容进行交互，但为用户提供信息的情况下的最佳选择。

**注：**  模态显示另一个视图控制器之前，您需要先移除可见的`popover`。有关何时以及如何在您的应用程序使用popovers具体准则，参考*iOS Human Interface Guidelines*。

图5-1显示了用于显示从`split view controller`窗格的`popover`的一个例子。选择`popover`的一个播放，导致应用程序的主视图显示有关该播放信息。

**图5-1**   使用`popover`来显示主面板

![img](https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/splitview_portrait_popover.jpg)

## 创建和呈现`popover`

`popover`的内容是源于一个您提供的`view controller`对象。Popovers能够呈现大多数类型的视图控制器。当你准备在`popover`中呈现视图控制器，做到以下几点：

1. 创建`UIPopoverController`类的实例和使用`view controller`初始化它。
2. 指定`popover`的大小，您可以通过以下两种方式进行操作：
   - 赋值你想在弹出窗展现的`view controller`的`contentSizeForViewInPopover`属性。
   - 赋值`popover controller`的`popoverContentSize`属性 。
3. （可选）指定一个[委托](undefined)到`popover`。有关委托的更多信息，请参阅 [Implementing a Popover Delegate](https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/Popovers.html#//apple_ref/doc/uid/TP40011313-CH5-SW10)。
4. 呈现`popover`。

当你呈现`popover`时，你将其与用户界面的特定部分相关联。Popovers通常与`tool bar`按钮相关联，因此`presentPopoverFromBarButtonItem:permittedArrowDirections:animated:`方法是便捷方式，用来呈现从您的应用程序的工具栏的popovers。还可以使用`presentPopoverFromRect:inView:permittedArrowDirections:animated:`的方法，将`popover`与视图中的一个特定部分相关联 。

`popover`通常从正在呈现的视图控制器的属性`contentSizeForViewInPopover`，获得初始大小。该属性中存储的默认只为320像素宽×1100像素高。您可以通过指定`contentSizeForViewInPopover`属性一个新值，代替默认值。或者，你可以赋值给`popover controller`的`popoverContentSize`属性。如果更改由`popover`显示的视图控制器，`popoverContentSize`属性包含的原来任何自定义`size`信息是由新的视图控制器的`size`所取代。 当弹出窗口可见时，改变内容视图控制器或其大小会自动进行动画效果。你也可以使用`setPopoverContentSize:animated:`方法改变尺寸。

**注意：**  在屏幕上的`popover`的实际位置由`popover controller`本身确定，并且基于几个因素，包括您视图控制器的内容的大小，作为`popover`的源的按钮或视图的位置，和允许的箭头方向。

清单5-1显示了一个简单的action方法，是呈现`popover`，以响应用户点击工具栏按钮。`popover`被存储一个属性（由所属的类定义），保留强引用。`popover`的尺寸被设置为视图控制器的视图的大小，但两者不必相同。当然，如果两者是不一样的，你必须使用一个滚动视图，以确保用户可以看到所有的`popover`的内容。

**清单5-1**   代码方式呈现`popover`

```objective-c
- (IBAction)toolbarItemTapped:(id)sender
{
   MyViewController* content = [[MyViewController alloc] init];
   UIPopoverController* aPopover = [[UIPopoverController alloc]
        initWithContentViewController:content];
   aPopover.delegate = self;
 
   // Store the popover in a custom property for later use.
   self.popoverController = aPopover;
 
   [self.popoverController presentPopoverFromBarButtonItem:sender
        permittedArrowDirections:UIPopoverArrowDirectionAny animated:YES];
}
```



当用户点击`popover`视图外部范围，`popover controller`被自动移除。在`popover`视图内点击，不会自动移除。但您可以通过代码方式使用`dismissPopoverAnimated:`方法，来移除`popover`。你可能会需要这个方法，当用户在您的视图控制器的内容选择项目或执行某些动作，来移除屏幕上`popover`。如果你使用代码方式移除`popover`，而不是系统自动移除，你需要将`popover controller`对象的引用，存储在您的视图控制器可以访问它的地方。系统不提供对当前活动的`popover controller`的引用。

## 实现`popover`代理

当通过用户通过点击`popover view`外部范围来移除`popover`时，`popover`会自动通知其代理。如果由您提供代理，就可以使用这个对象来防止`popover`移除，或响应移除且执行其他操作。`popoverControllerShouldDismissPopover:`委托方法可以让你控制`popover`是否真的要被移除。如果您的代理未实现这个方法，或者如果您的实现返回`YES`，控制器移除了`popover`并向代理发送`popoverControllerDidDismissPopover:`消息。

在大多数情况下，你不需要重写`popoverControllerShouldDismissPopover:`这些方法。该方法适用于移除`popover`可能会导致应用程序出现问题的情况。而不是从该方法中返回`NO`，不过，最好是避免需要保持`popover`活性的设计。例如，如果是这种情况，使用模态呈现内容更好，来强制用户输入所需信息，或接受或取消更改。

当你的委托方法`popoverControllerDidDismissPopover:`被调用时，`popover`本身已经从屏幕上消失。在这一点上，在这个委托方法内删除`popover controller`的引用是安全的，如果你不打算再次使用它。你也可以在此方法内刷新你的用户界面或更新应用的状态。

## 在应用中管理Popovers的提示

你的应用程序编写`popover`相关的代码时，请考虑以下情况：

- 代码方式移除`popover`，需要一个`popover controller`指针。获得这样一个指针的唯一方法是存储自身，通常在内容视图控制器。这确保了内容视图控制器能够响应于相应的用户的行动，以移除`popover`。
- 您可以`popover controller`和重用他们，而不是从头开始创建新的。`popover controller`的可塑性很大，所以你可以在每次使用它们时指定一个不同的视图控制器和配置选项。
- 当呈现`popover`，尽可能为允许的箭头方向指定`UIPopoverArrowDirectionAny`常量， 指定这个常数，UIKit在`popover`定位和调整大小具有最大的灵活性。如果指定了一组允许有限的箭头方向，将`popover controller`可能早显示`popover`之前，会收缩`popover`的大小。