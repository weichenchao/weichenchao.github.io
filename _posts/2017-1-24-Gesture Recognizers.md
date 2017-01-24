---
layout: post
title: Gesture Recognizers
tags: 
- Event Handling Guide for iOS(二)
---
# Gesture Recognizers
手势识别把低层次的事件处理代码，转换为高层次的action。 它们是你连接到视图的各种对象，它们允许视图像控件一样响应各种操作。 手势识别转化各种触摸，来决定它们是否响应一个特定的手势，比如一个点击(swipe)， 捏合(pinch)，或旋转。 如果它们识别到指定的手势，就给目标对象发送一个action信息。 目标对象通常是视图的视图控制器，它响应**图1-1**中所示的手势。该设计模式既有力又简单；你可以动态地决定视图响应什么操作，你也可以给视图添加各种手势识别而不必要子类化该视图。

**图1-1** 连接到视图的手势识别

![img](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/gestureRecognizer_2x.png)

## 使用手势识别器来简化事件处理

`UIKit` 框架提供了一些系统已经预先定义好的手势识别，来侦测各种常用手势。如果可能的话，最好是使用预定义的手势识别，因为它们可以缩减了代码量。另外，使用一个标准的手势识别而不是自定义，以确保你的应用程序如用户期望的那样工作。

如果你想你的应用程序识别一个独特的手势，比如一个勾或一个漩涡状运动，你可以创建你自己的自定义手势识别。 如何设计和实现你自己的手势识别，详见 [Creating a Custom Gesture Recognizer.](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/GestureRecognizer_basics/GestureRecognizer_basics.html#//apple_ref/doc/uid/TP40009541-CH2-SW44)

### 内建手势识别，来识别各种常用手势

当你设计应用程序时，请思考你要启用的手势。  然后，对于**表1-1**中的每个手势，确定是否符合程序需要。

| Gesture                                  | UIKit class                    |
| ---------------------------------------- | ------------------------------ |
| Tapping (any number of taps)轻拍（任何次）      | `UITapGestureRecognizer`       |
| Pinching in and out (for zooming a view)捏（缩放） | `UIPinchGestureRecognizer`     |
| Panning or dragging                      | `UIPanGestureRecognizer`       |
| Swiping (in any direction)拖动（在任何方向）      | `UISwipeGestureRecognizer`     |
| Rotating (fingers moving in opposite directions)旋转 | `UIRotationGestureRecognizer`  |
| Long press (also known as “touch and hold”)长按 | `UILongPressGestureRecognizer` |

你的应用程序应该只以用户期待的方式来响应。 比如，一个捏合动作应该缩放，而轻击动作应该是选中。 关于正确使用手势的指南，详见*Apps Respond to Gestures, Not Clicks*。

### 手势识别连接到视图

每个手势识别都跟一个视图相关联。 通常，视图可以有多个手势识别，因为单个视图可能响应多个不同的手势。 要想一个手势识别能够识别在一个特殊视图上发生的各种触摸，你必须把手势识别连接到那个视图。 当用户触摸视图，手势识别会在视图对象接收之前，接收一个触摸发生的信息。 因此，手势识别可以通过更改视图的行为来响应各种触摸。

### 手势触发操作消息

当一个手势识别识别出其指定的手势后，它会给它的`target`发送一个`action`消息。 要想创建一个手势识别，你需要初始化一个`target`和一个`action`。

#### 离散和连续的手势

手势有离散手势，也有连续手势。 一个离散手势，比如一个轻击(tap)，只发生一次。一个连续手势，比如捏合(pinching)，会持续一段时间。 对于离散手势，手势识别就给目标发送一个`action`消息。对于连续手势，手势识别则保持发送`action`消息给目标，直到多点触摸序列结束，如**图 1-2**。

 **图1-2** 离散和连续手势

![img](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/discrete_vs_continuous_2x.png)



## 用手势识别响应事件

 你需要做三件事来添加一个内建手势识别到应用程序：

1. 创建并配置一个手势识别实例。

   该步骤包括指定一个`target`，`action`，以及其他的各种属性(比如 numberOfTapsRequired)。

2. 把手势识别连接到一个视图。

3. 实现处理手势的`action`方法。

### 使用Interface Builder来添加一个手势识别

在Xcode里的Interface Builder中，添加手势识别跟你添加其他对象到用户界面是一样的---就是从对象库`object library`中拖拉手势识别，并将手势识别放在某个视图上。完成该操作后，手势识别自动连接到那个视图。 你可以检查手势识别被连接到了哪个视图，并且如果必要，在nib 文件中更改此连接。

当你创建完手势识别对象后，你需要创建并连接一个操作方法。任何时候手势识别识别手势时都将调用该方法。如果你需要在该操作方法外面引用手势识别，你还应该为手势识别创建并连接一个插座变量。

**列表1-1** 用Interface Builder添加手势识别

```objective-c
@interface APLGestureRecognizerViewController ()
@property (nonatomic, strong) IBOutlet UITapGestureRecognizer *tapRecognizer;
@end
 
@implementation
- (IBAction)displayGestureForTapRecognizer:(UITapGestureRecognizer *)recognizer
     // Will implement method later...
}
@end
```

### 通过程序添加一个手势识别

你可以通过分配和初始化一个具体的`UIGestureRecognizer`子类的实例来创建一个手势识别，比如 [UIPinchGestureRecognizer](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIPinchGestureRecognizer_Class/Reference/Reference.html#//apple_ref/occ/cl/UIPinchGestureRecognizer) 。 当你初始化手势识别时，指定一个`target`对象和一个` action selector`，正如列表1-2中所示，通常目标对象就是视图的视图控制器。

如果你通过代码创建一个手势识别，调用`addGestureRecognizer:` 方法来把它连接到视图。 **列表1-2** 创建了一个单个轻击手势识别，指定识别该手势需要一次轻击，然后把手势识别对象连接到一个视图。 通常，你在视图控制器的 viewDidLoad 方法里创建手势识别，如**列表1-2**所示。

**列表1-2** 通过程序创建一个单一的轻击手势识别

```objective-c
- (void)viewDidLoad {
     [super viewDidLoad];
 
     // Create and initialize a tap gesture
     UITapGestureRecognizer *tapRecognizer = [[UITapGestureRecognizer alloc]
          initWithTarget:self action:@selector(respondToTapGesture:)];
 
     // Specify that the gesture must be a single tap
     tapRecognizer.numberOfTapsRequired = 1;
 
     // Add the tap gesture recognizer to the view
     [self.view addGestureRecognizer:tapRecognizer];
 
     // Do any additional setup after loading the view, typically from a nib
} 
```

### Responding to Discrete Gestures   响应离散手势

当你创建一个手势识别时，你把`recognizer`连接到`action`方法。使用该`action`方法来响应手势识别的手势。**列表1-3** 提供了一个响应离散手势的例子。 当用户轻击了带有手势识别的视图时，视图控制器显示一个图像视图表示"Tap"发生。  `showGestureForTapRecognizer: `方法决定了视图中手势的位置，从`recognizer`的`locationInView:` 属性中获取该位置，然后把图片显示到该位置。

 注意： 接下来的3段代码例子取自 *Simple Gesture Recognizers*样本代码工程，你可以查看它以获得更多上下文。

 **列表1-3** 处理一个双击手势

```objective-c
- (IBAction)showGestureForTapRecognizer:(UITapGestureRecognizer *)recognizer {
       // Get the location of the gesture
      CGPoint location = [recognizer locationInView:self.view];
 
       // Display an image view at that location
      [self drawImageForGestureRecognizer:recognizer atPoint:location];
 
       // Animate the image view so that it fades out
      [UIView animateWithDuration:0.5 animations:^{
           self.imageView.alpha = 0.0;
      }];
}
```

每个手势识别都有它自己的属性。比如，如**列表1-4**中，`showGestureForSwipeRecognizer: `方法使用点击swipe手势识别的`direction` 特性，来决定用户是向左清扫还是向右。 然后，它使用该值来让图像从清扫方向消失。

 **列表1-4** 响应一个向左或向右清扫手势

```objective-c
// Respond to a swipe gesture
- (IBAction)showGestureForSwipeRecognizer:(UISwipeGestureRecognizer *)recognizer {
       // Get the location of the gesture
       CGPoint location = [recognizer locationInView:self.view];
 
       // Display an image view at that location
       [self drawImageForGestureRecognizer:recognizer atPoint:location];
 
       // If gesture is a left swipe, specify an end location
       // to the left of the current location
       if (recognizer.direction == UISwipeGestureRecognizerDirectionLeft) {
            location.x -= 220.0;
       } else {
            location.x += 220.0;
       }
 
       // Animate the image view in the direction of the swipe as it fades out
       [UIView animateWithDuration:0.5 animations:^{
            self.imageView.alpha = 0.0;
            self.imageView.center = location;
       }];
}
```

###  响应连续手势

连续手势允许应用程序在手势发生进行时响应。 比如，用户可以一边做捏合手势，一边就可以看见缩放，或者允许用户沿着屏幕拖拉一个对象。

**列表1-5** 跟手势同样的角度显示一个"Rotate"图片，并且当用户停止旋转时，使用动画效果，让图片旋转回水平方向并且消失。 当用户旋转手指时，持续调用 `showGestureForRotationRecognizer: `方法，直到两个手指都拿起。

**列表1-5** 响应一个旋转手势

```objective-c
// Respond to a rotation gesture
- (IBAction)showGestureForRotationRecognizer:(UIRotationGestureRecognizer *)recognizer {
       // Get the location of the gesture
       CGPoint location = [recognizer locationInView:self.view];
 
       // Set the rotation angle of the image view to
       // match the rotation of the gesture
       CGAffineTransform transform = CGAffineTransformMakeRotation([recognizer rotation]);
       self.imageView.transform = transform;
 
       // Display an image view at that location
       [self drawImageForGestureRecognizer:recognizer atPoint:location];
 
      // If the gesture has ended or is canceled, begin the animation
      // back to horizontal and fade out
      if (([recognizer state] == UIGestureRecognizerStateEnded) || ([recognizer state] == UIGestureRecognizerStateCancelled)) {
           [UIView animateWithDuration:0.5 animations:^{
                self.imageView.alpha = 0.0;
                self.imageView.transform = CGAffineTransformIdentity;
           }];
      }
 
}
```

每次调用该方法时，图片都在`drawImageForGestureRecognizer: `方法中被设置为不透明。 当手势完成时，图片在`animateWithDuration: `方法中被设置为透明。`showGestureForRotationRecognizer:`方法通过检查手势识别的状态来判断手势是否完成。 关于这些状态的更多信息，请看[Gesture Recognizers Operate in a Finite State Machine.](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/GestureRecognizer_basics/GestureRecognizer_basics.html#//apple_ref/doc/uid/TP40009541-CH2-SW22)

## 定义手势识别如何交互

通常情况下，当你把手势识别添加到应用程序时，你需要了解你想要`recognizers`之间如何发生交互，或跟其它触摸事件处理的代码如何发生交互。要想完成这些，你首先需要理解更多点关于手势识别如何工作的知识。

### 手势识别在一个有限状态机里操作

手势识别以一种预定义的方式，从一个状态转换到另一个状态。根据特定条件，每种状态都可以转换到几种未来状态中的一种。确切的状态机根据手势识别是离散或是连续会发生变化，正如**图1-3**中所示。所有的手势识别在以`UIGestureRecognizerStatePossible`状态开始， 它们分析接收到的多点触摸序列，并且在分析过程中成功识别手势或失败。失败识别手势意味着手势识别转换到`UIGestureRecognizerStateFailed`状态.

**图1-3** 手势识别的状态机

![img](https://developer.apple.com/library/prerelease/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/gr_state_transitions_2x.png)



当一个离散手势识别识别成功，手势识别从`UIGestureRecognizerStatePossible`可能状态转换到被识别状态  `UIGestureRecognizerStateRecognized`)， 然后识别过程完成。

对于连续手势，当手势第一次被识别时，手势识别开始于`UIGestureRecognizerStateBegan`状态。然后，`Began`转换到改变状态(`UIGestureRecognizerStateChanged`)， 然后继续状态时又从`Changed`状态变为`Changed`状态。 当用户的最后一个手指从视图上举起时，手势识别转换到`UIGestureRecognizerStateEnded`状态， 识别完成。 注意结束状态是识别完成状态的一个别名。

如果连续手势不再适应预期的模式时，应从`Changed`状态过渡到取消状态`UIGestureRecognizerStateCancelled`。

除了转换到失败或取消状态，每次手势识别的状态*改变*时，手势识别都给`target`发送一个`action`消息。 然而，一个离散手势识别只发送一次`action`消息，仅在它从`Possible`状态过渡到`Recognized`状态时。一个连续手势识别在状态发生改变时，会发送多个操作消息。

当一个手势识别到达被识别(或结束)状态时，它把状态重置为`Possible`状态。把状态重置为可能状态，不会触发一个操作消息。

### 跟其它手势识别发生交互

一个视图可以带有多个手势。使用视图的[gestureRecognizers](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/UIView/UIView.html#//apple_ref/occ/instp/UIView/gestureRecognizers) 特性来确定视图都带有哪些手势识别。 你还可以分别使用[addGestureRecognizer:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/UIView/UIView.html#//apple_ref/occ/instm/UIView/addGestureRecognizer:) 方法和[removeGestureRecognizer:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/UIView/UIView.html#//apple_ref/occ/instm/UIView/removeGestureRecognizer:) 方法添加和删除视图上的手势识别，来动态改变视图如何处理手势。

当视图带有多个手势识别时，你可能想要改变手势识别是如何接收和分析触摸事件的竞争关系。默认情况下，没有设置哪个手势识别第一个接收到触摸的优先级，因此每次触摸都可以以不同的顺序传送给手势识别。 你可以重写该默认行为来：

- 指定一个手势识别比另一个手势识别优先分析某个触摸。
- 允许两个手势识别来同时操作。
- 阻止某个手势识别分析某个触摸。

使用[UIGestureRecognizer](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIGestureRecognizer_Class/Reference/Reference.html#//apple_ref/occ/cl/UIGestureRecognizer) 类方法，委托方法，以及其子类重写的方法来影响这些行为。

#### 为两个手势识别声明一个特定顺序

假设，你想要识别一个快速滑动(swipe)和一个慢速拖动(pan)手势，用这两个手势触发不同的操作。默认情况下，当用户尝试`swipe`时，该手势会被理解为一个`pan`。 这是因为`swipe`手势在满足各种必要条件后被设别为一个`swipe`手势之前，首先会满足`pan`手势的各种必要条件。

要同时识别`swipe `和`pan`，需要在`pan`手势识别之前，让`swipe`手势识别来分析触摸事件。 如果`swipe`手势识别已经确定某个触摸是一个`swipe`， 那么`pan`手势识别就不必再去分析该触摸。 如果`swipe`手势识别确定该触摸不是一个`swipe`， 它过渡到`Failed`状态，然后`pan`手势识别开始分析该触摸事件。

在此之前的iOS 7，如果一个手势识别需要另一个手势识别失败，您可以使用`requireGestureRecognizerToFail:`设置两个对象之间的永久关系。通过调用手势识别的`requireGestureRecognizerToFail`方法来说明两个手势识别之间这种类型的关系，如**列表1-6**所示。在该列表中，两个手势识别都被连接到了相同的视图上。

在iOS中7，`UIGestureRecognizerDelegate`引入了两种方法，允许在运行时通过手势识别委托对象，指定障碍要求：

- `gestureRecognizer:shouldRequireFailureOfGestureRecognizer:`
- `gestureRecognizer:shouldBeRequiredToFailByGestureRecognizer:`

对于这两种方法，每次识别尝试，调用姿势识别器委托一次，这意味着可以懒散地确定障碍要求。这也意味着您可以在不同视图层次结构中的识别器之间设置障碍要求。

示例，动态故障要求是在将屏幕边缘平移姿势识别器附加到视图的应用中。在这种情况下，您可能希望与该视图的子树相关联的所有其他相关手势识别器要求屏幕边缘手势识别器失败，因此您可以防止在开始识别过程后取消其他识别器时可能出现的任何图形故障。参考**列表1-6-2**

**列表1-6-1**  `pan`手势识别要求`swipe`手势识别到达fail状态

```objective-c
- (void)viewDidLoad {
       [super viewDidLoad];
       // Do any additional setup after loading the view, typically from a nib
       [self.panRecognizer requireGestureRecognizerToFail:self.swipeRecognizer];
}
```

**列表1-6-2** 创建障碍条件

```objective-c
UIScreenEdgePanGestureRecognizer *myScreenEdgePanGestureRecognizer;
...
myScreenEdgePanGestureRecognizer = [[UIScreenEdgePanGestureRecognizer alloc] initWithTarget:self action:@selector(handleScreenEdgePan:)];
myScreenEdgePanGestureRecognizer.delegate = self;
// Configure the gesture recognizer and attach it to the view.
...
 - (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldBeRequiredToFailByGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer {
    BOOL result = NO;
    if ((gestureRecognizer == myScreenEdgePanGestureRecognizer) && [[otherGestureRecognizer view] isDescendantOfView:[gestureRecognizer view]]) {
        result = YES;
    }
    return result;
 }
```

> 注意：如果你的应用程序识别同时支持单击和双击，并且你的单击手势识别不要求双击识别失败，那么即使是用户双击时，你也应该期待在双击操作之前接收单击操作。该行为是有意设计的，因为最好的用户体验通常都开启多种类型的操作。

> 如果你想要这两种操作不兼容，你的单击识别必须要求双击识别进入失败状态。 然而，这样你的单击操作会滞后用户的输入，因为单击识别会等到双击识别失败后才开始识别。

 

#### 阻止手势识别分析触摸

你可以通过给手势识别添加一个委托对象，来改变手势识别的行为。`UIGestureRecognizerDelegate `协议提供了一组方法来阻止手势识别分析触摸。 你可以选择使用协议中的 [gestureRecognizer:shouldReceiveTouch:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIGestureRecognizerDelegate_Protocol/Reference/Reference.html#//apple_ref/occ/intfm/UIGestureRecognizerDelegate/gestureRecognizer:shouldReceiveTouch:) 方法 或 [gestureRecognizerShouldBegin:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIGestureRecognizerDelegate_Protocol/Reference/Reference.html#//apple_ref/occ/intfm/UIGestureRecognizerDelegate/gestureRecognizerShouldBegin:) 方法。

当触摸开始时，如果能立即确定手势识别是否应该识别触摸，使用 `gestureRecognizer:shouldReceiveTouch: `方法来实现。每次有新触摸时都会调用该方法。 返回`NO`，手势识别接收触摸。默认值是`YES`。该方法不改变手势识别的状态。

**列表1-7** 使用 `gestureRecognizer:shouldReceiveTouch: `委托方法，来阻止手势识别接收自定义子视图中发生的触摸。

**列表1-7**  手势识别接收触摸

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Add the delegate to the tap gesture recognizer
    self.tapGestureRecognizer.delegate = self;
}
 
// Implement the UIGestureRecognizerDelegate method
-(BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {
    // Determine if the touch is inside the custom subview
    if ([touch view] == self.customSubview){
        // If it is, prevent all of the delegate's gesture recognizers
        // from receiving the touch
        return NO;
    }
    return YES;
}
```

如果你想在确定手势识别是否识别触摸之前等待，使用 `gestureRecognizerShouldBegin: `委托方法。 通常，如果你有一个`UIView` 或 `UIControl`子类并带有跟手势识别冲突的自定义触摸事件处理，你可以使用该方法。让这个方法返回`NO`，让手势识别立即进入失败状态，允许其他触摸处理来处理。 当手势识别想要过渡到`Possible`状态以外的状态时，如果手势识别将阻止一个视图或控件接收一个触摸，该方法被调用。

如果你的视图或视图控制器不能成为手势识别的`delegate`，你也可以使用`UIView`的`gestureRecognizerShouldBegin:UIView`方法 。

#### 开启同时手势识别

默认情况下，两个手势识别不能同时识别不同手势。但是，假设你想让用户可以同时捏合并旋转一个视图，你需要改变默认行为，你可以通过调用`gestureRecognizer:shouldRecognizeSimultaneouslyWithGestureRecognizer:` 方法来实现。该方法是可选的。 当一个手势识别的手势分析可能阻碍另一个手势识别识别它的手势时，可以调用该方法，反之亦然。 该方法默认范围`NO`。当你想让两个手势同时分析它们的手势时，返回`YES`。

注意：你只在一个手势识别需要开启同时识别功能时，才需要实现一个委托并返回`YES`。 然而，它还意味着返回NO，并不一定阻止同时识别功能，因为其他手势识别的委托可能返回了`YES`。

 

#### 给两个手势指定一个单向关系

如果你想要控制两个`recognizers`是如何交互的，但你需要指定一个单向关系，你可以重写[canPreventGestureRecognizer:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIGestureRecognizer_Class/Reference/Reference.html#//apple_ref/occ/instm/UIGestureRecognizer/canPreventGestureRecognizer:) 或 [canBePreventedByGestureRecognizer:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIGestureRecognizer_Class/Reference/Reference.html#//apple_ref/occ/instm/UIGestureRecognizer/canBePreventedByGestureRecognizer:) 子类方法并返回`NO`(默认为`YES`)。 比如，如果你想用一个旋转手势阻止一个捏合手势，但是又不想捏合手势阻止一个旋转手势，你可以用如下语句指定。

```
[rotationGestureRecognizer canPreventGestureRecognizer:pinchGestureRecognizer];
```

同时，重写旋转手势识别的子类方法来返回NO。更多管理如何子类化 。详见` Creating a Custom Gesture Recognizer`

如果两个手势都不能相互阻止，使用[gestureRecognizer:shouldRecognizeSimultaneouslyWithGestureRecognizer:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIGestureRecognizerDelegate_Protocol/Reference/Reference.html#//apple_ref/occ/intfm/UIGestureRecognizerDelegate/gestureRecognizer:shouldRecognizeSimultaneouslyWithGestureRecognizer:) 。 默认情况下，捏合手势阻止旋转手势，或者旋转手势阻止捏合手势，因为这两个手势不能同时被识别。

### 跟别的用户界面控件交互

在iOS 6.0 或以后版本中，默认控件操作方法防止重复手势识别的行为。比如，一个按钮的默认操作是一个单击。如果你有一个单击手势识别绑定到一个按钮的父视图上，然后用户点击该按钮，最后按钮的`action`操作方法接收触摸事件而不是手势识别。 这类情况只用于手势识别跟控件的默认操作重复时，包括：

-  单个手指在`UIButton`， `UISwitch`， `UIStepper`， `UISegmentedControl`， and `UIPageControl` 上的单击。
-  单个手势在 `UISlider `上的快速滑动，轻扫方向跟slider平行
-  单个手指的在[UISwitch](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UISwitch_Class/Reference/Reference.html#//apple_ref/occ/cl/UISwitch) 控件上的pan手势，方向跟`switch`平行。

如果你有一个这些控件的自定义子类，你想要改变其默认操作，把手势识别直接连接到控件而不是连接到其父视图。然后，手势识别首先接收到触摸事件。请确认你已经阅读了 *iOS Human Interface Guidelines*文档，以确保你的应用程序提供了一个直观的用户体验，特别是当你重写一个标准控件的默认行为时。

### 手势识别解读原始触摸事件

目前位置，你已经学习了关于手势以及应用程序如何识别并响应它们。 然而，要想创建一个自定义手势识别或想控制时手势别如何跟视图的触摸事件处理相交互，你需要更具体地思考触摸`touch`和事件`event`。

 

### 一个事件包含了当前多点触控序列的所有触控

在iOS中，一个触摸是一个手指在屏幕上的存在或运动。 一个手势有一个或多个触摸，它由[UITouch](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITouch_Class/Reference/Reference.html#//apple_ref/occ/cl/UITouch) 对象表示。比如，一个pinch-close手势有两个触摸--两个手指在屏幕上从相反方向朝着彼此移动。

 一个事件包含一个多点触摸序列的所有触摸。 一个多点触摸序列以一个手指触摸屏幕开始，以最后一个手指离开屏幕结束。 当一个手指移动时，iOS发送v摸对象给事件。一个多点触摸事件由 [UIEventTypeTouches](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIEvent_Class/Reference/Reference.html#//apple_ref/c/econst/UIEventTypeTouches) 类型`UIEvent `对象表示。

每个触摸对象只跟踪一个手指，并且只在触摸序列期间跟踪。 在序列期间，UIKit跟踪手指并更新触摸对象的各种属性。 这些属性包括触摸阶段(phase)，它在视图中的位置，它的前一个位置，以及它的时间戳。

触摸阶段表明一个触摸何时开始，它是移动的还是静止的，以及它何时结束---当手指不再触摸屏幕的时间。 正如**图1-4**所示，应用程序在任何触摸的每个阶段之间接收事件对象。

**图1-4** 一个多点触摸序列和触摸阶段

![A multi-touch sequence and touch phases](https://developer.apple.com/library/prerelease/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/event_touch_time_2x.png)

> 注意：手指没有鼠标点击精确。 当用户触摸屏幕时，接触的区域实际上是椭圆的，并且会比用户期待的位置稍微篇低。 该接触面会根据手指的尺寸和方向，手指使用时的压力，以及其它因素的不同而发生改变。底层多点触摸系统会替你分析该信息并计算一个单击点，因此你不需要自己写代码来实现它。

 

### 应用程序在触摸处理方法中接收触摸

在一个多点触摸序列期间，应用程序在新触摸发生或者给出的触摸阶段发生改变时发送这些信息；它调用以下方法：

- `touchesBegan:withEvent:` 当一个或多个手指触摸屏幕时调用
- `touchesMoved:withEvent:` 当一个或多个手势移动时调用
- `touchesEnded:withEvent:` 当一个或多个手指离开屏幕时调用
- `touchesCancelled:withEvent:` 当触摸序列被系统事件取消时调用，比如有一个来电。

每个方法都跟一个触摸阶段相关联；比如，`touchesBegan: `方法跟 `UITouchPhaseBegan `方法相关联。 触摸对象的阶段(phase)被存储在其`phase `特性里。

注意： 这些方法跟手势识别状态没有关联，比如`UIGestureRecognizerStateBegan` 和 `UIGestureRecognizerStateEnded`等。 手势识别器状态严格表示手势识别器自身的阶段，不表示正在被识别的触摸对象阶段。

 

## 调节触摸到视图的传递

可能有时候你想要在手势识别器之前接收到一个触摸。但是在你可以改变触摸到视图的传递路径之前，你需要理解其默认行为。在简单情况下，当一个触摸发生时，触摸对象从`UIApplication`对象传递到`UIWindow`对象。 然后，窗口首先把触摸发送给触摸发生的视图上关联的任何手势识别器，而不是先发送给视图对象自身。

 **图1-5** 触摸事件的默认传递路径

![img](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/path_of_touches_2x.png)

### 手势识别器首先识别一个触摸

窗口延迟把触摸对象传递给视图，这样手势识别器就可以首先分析触摸。延迟期间，如果手势识别器识别出一个触摸手势，然后窗口就绝不会再把触摸对象传递给视图，并且还取消任何先前传递给视图的任何触摸对象，这些触摸对象都是被识别序列的一部分。

比如，如果你有一个手势识别器用来识别一个离散手势，该手势要求一个双手指的触摸，该触摸就会被解释成两个单独的触摸对象。 当触摸发生时，触摸对象从应用程序对象传递到触摸发生视图的窗口对象，然后发生以下序列，请看图1-6。

**图 1-6**  触摸消息序列

1. 窗口在Began 阶段发送两个触摸对象---通过 `touchesBegan:withEvent: `方法---给手势识别器。 手势识别器还不能识别该手势，因此它的状态是Possible. 窗口发送这些同样的触摸给手势识别器相关联的视图。
2. 窗口在Moved阶段发送两个触摸对象---通过`touchesMoved:withEvent: `方法---- 给手势识别器。 识别器任然不能侦测该手势，状态还是Possible。 窗口然后发送这些触摸到相关联的视图。
3. 窗口在Ended阶段发送一个触摸对象--- 通过`touchesEnded:withEvent: `方法---给手势识别器。 虽然该触摸对象对于手势来说信息还不够，但是窗口还是把该对象扣住(withhold)不发送给视图。
4. 窗口在Ended阶段发送另一个触摸。 手势识别器这是可以识别出该手势，因此把状态设置为Recognized. 就在第一个操作信息被发送之前，视图调用`touchesCancelled:withEvent: `方法来使先前在Began 和Moved阶段发送的触摸对象无效(invalidate)。触摸在Ended阶段被取消。

现在假设手势识别器在最后一步确定它正在分析的多点触摸序列不是它的手势。它把状态设置为`UIGestureRecognizerStateFailed`.。 然后窗口在Ended阶段通过[touchesEnded:withEvent:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIResponder_Class/Reference/Reference.html#//apple_ref/occ/instm/UIResponder/touchesEnded:withEvent:) ，发送这两个触摸对象给相关联的视图。

一个连续手势的手势识别器遵循一个相似的序列，除了它更有可能在触摸对象到达Ended 阶段之前就识别出它的手势。一旦识别出它的手势，它把状态设置为 `UIGestureRecognizerStateBegan` (而不是Recognized). 窗口把多点触摸序列中的所有子序列触摸对象发送给手势识别器，而不是发送到附属的视图。

###  影响到视图的各个触摸的传递

你可以改变 `UIGestureRecognizer `特性的几个值来改变默认传递路径，让它们以特定的方式传递。如果你改变这些特性的默认值，以下行为将发生变化：

- `unresponsive.delaysTouchesBegan`(默认为NO) —正常情况下，窗口在Began 和 Moved 阶段把触摸对象发送给视图和手势识别器。 把`delaysTouchesBegan`设置为`YES`，使得窗口不会在Began阶段把触摸对象发送给视图。 

  这样做确保一个手势识别器识别它的手势时，没有把部分触摸事件传递给相连的视图。 请谨慎设置该特性时，因为它会使你的界面反应迟钝。

- `delaysTouchesEnded`(默认为YES) — 当该特性被设置为YES时，它确保视图不会完成一个动作，而该动作是手势可能想在稍候取消的。当一个手势识别器正在分析一个触摸事件时，窗口不会在Ended阶段传递触摸对象到相连的视图。如果一个手势识别器识别出它的手势，则触摸对象被取消。 如果手势识别器没有识别出它的手势，窗口通过一个`touchesEnded:withEvent:`消息把这些对象传递给视图。设置该特性为`NO`，允许视图和手势识别器可以同时在Ended阶段分析触摸对象。

- 例如，设想一个视图有一个点击手势识别器，它要求双击，然后用户双击了该视图。 把这个属性设置为YES后，视图获得`touchesBegan:withEvent:`， `touchesBegan:withEvent:`， `touchesCancelled:withEvent:`， 以及 `touchesCancelled:withEvent:` 信息序列。如果这个属性被设置为`NO`，视图获得以下信息序列：`touchesBegan:withEvent:`， `touchesEnded:withEvent:`， `touchesBegan:withEvent:`， 以及`touchesCancelled:withEvent:` ，它表示在`touchesBegan:withEvent: `消息中，视图可以识别一个双击。

如果一个手势识别器侦测到一个不属于该手势的触摸，它可以把该触摸直接传递给它的视图。 要想实现它，手势识别器对自己调用[ignoreTouch:forEvent:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIGestureRecognizer_Class/Reference/Reference.html#//apple_ref/occ/instm/UIGestureRecognizer/ignoreTouch:forEvent:)方法，它在触摸对象中传递。

## 创建一个自定义手势识别器

要想实现一个自定义手势识别器，首先在Xcode里创建一个 `UIGestureRecognizer` 的子类。然后，在子类的头文件中加入以下import指令。

```
#import <UIKit/UIGestureRecognizerSubclass.h>
```

下一步，从`UIGestureRecognizerSubclass.h`中拷贝以下方法声明到你的头文件；这些是你在子类中需要重写的方法。

```
- (void)reset;
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```

 在所有这些需要重写的方法中，你必须调用父类的实现，即使该方法有一个null实现。

注意：`UIGestureRecognizerSubclass.h`中的 `state `属性目前是`readwrite`状态，而不是`readonly`，就跟它在`UIGestureRecognizer.h`中一样。 你的子类可以通过给特性分配` UIGestureRecognizerState `常量来改变其状态。

### 为自定义手势识别器实现触摸事件处理方法

一个自定义手势识别器实现的核心是四个方法： `touchesBegan:withEvent:`，`touchesMoved:withEvent:`，`touchesEnded:withEvent:`，and `touchesCancelled:withEvent:`。在这些方法中，你通过设置一个手势识别器的状态，把低层触摸事件解析为手势识别。列表1-8创建了一个离散单击勾选手势的手势识别器。 它记录了手势的中心点---即勾的上升开始点—这样客户端就可以获取这个值。

该例子只有一个单一视图，但是大多是应用程序都有多个视图。一般来说，你应该把触摸位置转换为屏幕的坐标系，这样你就可以正确的识别出跨越多个视图的手势。

列表1-8 一个勾(checkmark)手势识别器的实现

```objective-c
#import <UIKit/UIGestureRecognizerSubclass.h>
 
// Implemented in your custom subclass
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesBegan:touches withEvent:event];
    if ([touches count] != 1) {
        self.state = UIGestureRecognizerStateFailed;
        return;
    }
}
 
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesMoved:touches withEvent:event];
    if (self.state == UIGestureRecognizerStateFailed) return;
    UIWindow *win = [self.view window];
    CGPoint nowPoint = [touches.anyObject locationInView:win];
    CGPoint nowPoint = [touches.anyObject locationInView:self.view];
    CGPoint prevPoint = [touches.anyObject previousLocationInView:self.view];
 
    // strokeUp is a property
    if (!self.strokeUp) {
        // On downstroke, both x and y increase in positive direction
        if (nowPoint.x >= prevPoint.x && nowPoint.y >= prevPoint.y) {
            self.midPoint = nowPoint;
            // Upstroke has increasing x value but decreasing y value
        } else if (nowPoint.x >= prevPoint.x && nowPoint.y <= prevPoint.y) {
            self.strokeUp = YES;
        } else {
            self.state = UIGestureRecognizerStateFailed;
        }
    }
}
 
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesEnded:touches withEvent:event];
    if ((self.state == UIGestureRecognizerStatePossible) && self.strokeUp) {
        self.state = UIGestureRecognizerStateRecognized;
    }
}
 
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesCancelled:touches withEvent:event];
    self.midPoint = CGPointZero;
    self.strokeUp = NO;
    self.state = UIGestureRecognizerStateFailed;
}
```

离散手势和连续手势的状态过渡是不一样的，正如在 [“Gesture Recognizers Operate in a Finite State Machine.”](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/GestureRecognizer_basics/GestureRecognizer_basics.html#//apple_ref/doc/uid/TP40009541-CH2-SW22) 中所述。 当你创建一个自定义手势识别器时，你通过指定响应的状态来表明它是离散手势或是连续手势。比如，列表1-8 中的复选标记(checkmark)手势识别器，它不会把状态设置为`Began `或者 `Changed`，因为它是离散手势。

当你子类化一个手势识别器时，最重要的事情是正确地设置手势识别器的`state`。 为了手势识别器的的交互如预期，iOS需要了解一个手势识别器的状态。比如，如果你想要实现同时识别或者要求一个手势识别器失败，iOS需要了解识别器的当前状态。

关于创建自定义手势识别器的更多信息，请看 *WWDC 2012: Building Advanced Gesture Recognizers*.

### 重置手势识别器的状态

如果你的手势识别器转换到`Recognized`/`Ended`, `Canceled`， 或者`Failed`状态，[UIGestureRecognizer](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIGestureRecognizer_Class/Reference/Reference.html#//apple_ref/occ/cl/UIGestureRecognizer) 类刚好在手势识别器过渡回`Possible`状态前调用[reset](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIGestureRecognizer_Class/Reference/Reference.html#//apple_ref/occ/instm/UIGestureRecognizer/reset) 方法。

实现`reset `方法来重置任何内部状态，这样你的识别器能准备进行识别手势的一个新的尝试，正如**列表 1-9**所示。当手势识别器从该方法返回后，它将不再接收任何在进行的触摸更新。

 **列表1-9** 重置一个手势识别器

```objective-c
- (void)reset {
    [super reset];
    self.midPoint = CGPointZero;
    self.strokeUp = NO;
}
```

