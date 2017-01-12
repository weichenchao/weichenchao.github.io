---
layout: post
title: Event Delivery: The Responder Chain
tags: 
- Event Handling Guide for iOS(三)
---

# 事件传递：响应者链

当你设计应用时，你可能希望动态地响应事件。例如，触摸可能发生在屏幕上的很多不同对象上，你必须决定你希望哪一个对象来响应给定的事件，理解对象是如何接收事件的。

当用户产生的事件发生时，UIKit创建一个封装了所有需要处理该事件信息的事件对象。然后将该事件对象放在 `active app’s `(`Application object`)事件队列中。对于触摸事件来说，该对象是封装在` UIEvent` 对象中的一组touch。对于运动事件来说，事件对象不一样，取决于你使用哪种框架，以及哪种类型的运动事件。

一个事件沿着特定的路径传递，直到它被传递给了会处理它的对象。首先，单例` UIApplication` 对象从队列的顶部获取一个事件，并且向下分派。典型的，它将事件发送给应用的主 `window object` ，`key window`对象将事件传递给`initial`(最初的)对象来处理。`initial`对象取决于事件的类型。

- **Touch event.** 对于触摸事件来说，`window`对象先尝试将事件传递给触碰的`view`。那个view被称为`hit-test view`。找到`hit-test view`的过程叫做*hit-testing*，详见 *Hit-Testing Returns the View Where a Touch Occurred*。
- **Motion and remote control events.**对于这些事件，`window`对象将发送摇晃运动或者远程控制事件，给第一响应者来处理。详见*The Responder Chain Is Made Up of Responder Objects*。

这些事件路径的终极目标是找到一个可以处理、响应事件的对象。因此，`UIKit`会先将事件发送给最适合处理该事件的对象。对于`touch`事件，对象是`hit-test view`，而其他事件，则是第一响应者。以下的章节更详细的解释了`hit-test view`和第一响应着是如何被确定的。

## Hit-Testing 返回触碰的view

iOS使用`hit-testing`来找到触碰的view。`Hit-testing`需要检查`touch`是否在相关的`view`对象的边界内。如果是，它会递归检查那个`view`的所有子`view`。包含触摸点的视图层次中最底层的`view`，成为`hit-test view`。iOS确定了`hit-test view`之后，会将`touch`事件传递给这个`view`,让它来处理事件。

为了说明,假设用户触摸了图2-1中的view E。iOS以下面的顺序通过检查子视图来找到`hit-test view`。

1. touch在view A的边界内，所以会检查子视图B和C。
2. touch不在view B的边界内，但是在view C的边界内，所以它会检查子视图D和E。
3. touch不在view D的边界内，但是在view E的边界内。
   View E是包含touch的视图层次中最底层的view，所以它成为了`hit-test view`。

 **图2-1** Hit-testing返回被触摸的子视图

![图2-1 Hit-testing返回被触摸的子视图](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/hit_testing_2x.png)



`hitTest:withEvent:` 方法为指定的 `CGPoint` 和 `UIEvent `返回`hit test view`。`hitTest:withEvent`方法先调用`pointInside:withEvent:`方法。如果传递给`hitTest:withEvent:`的point是在view边界内，`pointInside:withEvent:`方法返回`YES`。然后，该方法会在每一个返回`YES`的子视图上递归调用hitTest:withEvent:方法。

如果传给`hitTest:withEvent:`的point并不在view的边界内，`pointInside:withEvent:`方法返回`NO`，该point被忽略，`hitTest:withEvent:`返回`nil`。如果一个子view返回`NO`,则该view层次的整个分支被忽略，因为如果touch不是发生在那个子view上，它也不会是发生在那个子view的任何子视图上。那意味着，在父视图之外的子视图上的任何点不能接收到touch事件，因为touch point必须在父视图和子视图的边界之内。如果subview的 `clipsToBounds` 设置为`NO`，这是可能发生的。

> **注意:** touch对象在整个生命周期内都被关联到它的`hit-test view`上，即使touch在稍后移动到了view之外。

`hit-test view`有优先处理touch事件的机会。如果hit-test view不能处理事件，事件会沿着视图的响应者链被传递，直到系统找到可以处理touch事件的对象。

## 响应者链由响应者对象组成

许多类型的事件依靠响应者链来进行事件传递。响应者链是一系列连接着的响应者对象。开始于第一个响应者，结束于`application`对象。如果第一个响应者不能处理事件，它会将事件传递给响应者链中的下一个响应者。

*responder object* 是一个可以响应并处理事件的对象。 `UIResponder` 类是所有响应者对象的基类，它不仅为事件处理定义了编程接口，也为常见的响应者行为定义了编程接口。`UIApplication`,`UIViewController`,和`UIView`的实例都是响应者，这意味着所有的视图和大多数的主控制器对象都是响应者。注意`Core Animation layers`不是响应者。

*first responder* 首先要接收事件。通常，第一响应者是一个view对象。一个对象可以通过以下两件事成为第一响应者：

1. 重写`canBecomeFirstResponder` 方法，使其返回`YES`。
2. 收到`becomeFirstResponder `消息。如果有必要，对象可以给自己发送该消息。

> **注意:** 确保在让对象成为第一响应者之前，你的程序已经建立了对象的图。例如，你通常在你重写的 `viewDidApper: `方法中调用 `becomeFirstResponder `方法。如果你尝试在 `viewWillAppear: `赋值第一响应者，此时对象图尚未建立，那么`becomeFirstResponder`方法会返回`NO`。

`Events` 不是唯一依靠响应者链的对象。响应者链被用在以下情况:

- **Touche events.** 如果`hit-test view`不能处理`touch`事件，事件沿着（开始于`hit-test view`的）响应者链传递。

- **Motion events.** 处理摇晃事件，第一响应者必须实现`UIResponder`类的 `motionBegan:withEvent:` 和`motionEnded:withEvent:` 方法，详见Detecting Shake-Motion Events with UIEvent。

- **Remote control events.** 处理远程控制事件，第一响应者必须实现`UIResponder`类的 `remotecontrolReceivedWithEvent: `方法。

- **Action messages.** 当用户操作一个`control`时，比如一个`button`或`switch`,`action`方法的`target`是`nil`,消息会通过以第一响应者(可能是控件本身）开始的响应者链被发送。

- **Editing-munu messages.** 当用户点击编辑菜单命令的时候，iOS使用响应者链找到实现了必要方法(如cut:,copy:,paste:)的对象。更多信息，请看 *Displaying and Managing the Edit Menu*以及示例代码工程:*CopyPasteTile.*

- **Text editing.** 当用户点击一个`text field`或者`text view`时，该view会自动变成第一响应者。默认情况下，虚拟键盘出现，`text field`或`text view`变成编辑状态。你可以显示一个自定义的` input view`来代替键盘的，如果这样对你的应用是合适的。你可以为任何的响应者对象加入一个自定义的` input view`。详见*Custom Views for Data input* 。

  `UIKit`自动将用户点击的`text field`或`text view`变成第一响应者;应用必须显式的使用`becomFirstResponder`方法来设置所有其他的对象。

## 响应者链按照特定的传递路径

如果initial对象 (`hit-test view`或第一响应者 )，不处理事件，`UIKit`传递事件给链中的下一个响应者。每一个响应者对象自己决定是否想要处理事件，或者是通过 `nextResponder` 方法将事件向下一个响应者传递。这个过程一直持续到有响应者处理了事件，或者没有响应者了。

当iOS检测到事件，并将它传递给initial对象(通常时是view)时，响应者链序列开始。initial view可以最优先处理事件。图2-2展示了两个应用配置各自的事件传递路径。一个应用的事件传递路径依靠它的具体结构，但是所有的事件传递路径都是同样的启发法。

**图2-2** iOS的响应者链

![](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/iOS_responder_chain_2x.png)

左边的应用，事件沿着以下路径传递：

1. **initial View** 试图处理事件或消息。如果它不能处理这个事件，它将事件传递给父视图，因为`initial view`不是视图层次中最顶层的view。
2. **superview** 试图处理事件。如果父视图不能处理这个事件，它会将事件传递给父视图，因为它仍然不是视图层中的最顶层view。
3. 在视图控制器的视图层次中的 **topmost view** 试图处理该事件。如果最顶层view不能处理事件，它将事件传递给它的view controller。
4. **view controller** 试图处理事件，如果不能处理的话，会将事件传递给window。
5. 如果 **window object** 仍然不能处理事件，它会将事件传递给 `singleton app object`,即单例的`Application`。
6. 如果 **app object** 不能处理事件，则它会废弃事件。

右边的应用按照稍微不同的路径，但是所有的事件传递路径安装这些启发法:

1. `view`沿着它的视图控制器的视图层次向上传递`event`,直到最顶层视图。
2. `topmost`视图将事件传递给它的`view controller.`
3. `view controller`将事件传递给它的最顶层view的父视图。
   重复1-3步，直到事件到达`root view controller`.
4. `root view controller`将事件传递给`window`对象。
5. `window`将事件传递给`app`对象。

**重要:** 如果你实现了一个自定义的view来处理远程控制事件，`action`消息，`shake-motion`事件，或者`editing-menu`消息，不要直接使用` nextResponder` 来向上发送`event`或`message`。相反，需要调用当前`event handing method`的父类实现，让`UIKit`来为你处理响应者链的遍历。