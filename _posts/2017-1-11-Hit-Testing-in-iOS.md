---
layout: post
title: Hit-testing in iOS
tags: 
- Hit-testing
---

`Hit-testing`是一个确定point（如触摸点）是否在绘制在屏幕上的既定图形对象(如`UIView`)的过程。IOS使用`Hit-testing`，以确定哪些`UIView`是用户的手指下最顶层的视图，即应该接收触摸事件。通过使用前序深度优先遍历算法搜索视图层次，来实现`Hit-testing`。

在解释`Hit-testing`的工作原理之前，重要的是要了解何时执行`Hit-testing`。下图说明了单次触摸的流程，从手指触摸屏幕直到手指离开屏幕为止：

[![触摸事件流](http://smnh.me/images/hit-test-touch-event-flow.png)](http://smnh.me/images/hit-test-touch-event-flow.png)

如上图所示，每当手指触摸屏幕时都会执行`Hit-testing`。而且，是在任何视图或手势识别器接收到了代表该触摸所属事件的`UIEvent`对象之前。

> 注意：由于未知的原因，`Hit-testing`连续执行多次。然而，确定的`Hit-testing`视图保持相同。

`Hit-testing`完成和触摸点在最顶层的视图被确定后，在触摸时间序列的所有阶段（即：开始，移动，结束，或者取消）中，`hit-test view`都与`UITouch`对象关联。除了`hit-test view`，该视图或其根视图关联的手势识别也与`UITouch`对象关联。然后，`hit-test view`开始接收触摸事件的序列。

重要的是，即使手指移动到`hit-test view`界限之外，到了另一视图，`hit-test view`仍然继续接收所有触摸，直到触摸事件序列的结束：

> “触摸对象是一直与`hit-test view`相关联，即使之后触摸移动视图外。” 
> [Event Handling Guide for iOS, iOS Developer Library](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW4)

如前所述，`Hit-testing`使用前序深度优先遍历（首先访问根节点，然后从较高到较低的索引遍历其子树）。这种遍历允许减少遍历迭代的次数，并且一旦找到包含触摸点的第一最深后代视图，停止搜索处理。这是可能的，因为子视图总是呈现在其超级视图之前，并且同级视图总是呈现在其同级视图的前面，并且在子视图数组中具有较低的索引。这样，当多个重叠视图包含特定点时，最右侧子树中的最深视图将是最前面的视图。

> “在视觉上，子视图的内容掩盖了其父视图的全部或部分内容。每个超级视图将其子视图存储在有序数组中，该数组中的顺序也会影响每个子视图的可见性。如果两个兄弟子视图相互重叠，那么最后添加的子视图出现在另一个的顶部
> [视图编程指南适用于iOS，iOS开发库](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW24)

下图显示了在屏幕上绘制的视图层次结构树及其匹配UI的示例。从左到右的树枝排列反映了subviews数组的顺序。

[![查看层次结构树](http://smnh.me/images/hit-test-view-hierarchy.png)](http://smnh.me/images/hit-test-view-hierarchy.png)

可以看出，“View A”和“View B”以及它们的子视图“View A.2”和“View B.1”是重叠的。但由于“View B”的子视图索引高于“View A”，“View B”及其子视图呈现在“View A”及其子视图上方。因此，当用户的手指在与“View A.2”重叠的区域中触摸“View B.1”时，应当通过`Hit-testing`返回“View B.1”。

通过采用前序深度优先遍历，允许在找到包含触摸点的最深视图时，停止遍历：

[![查看层次结构深度优先遍历](http://smnh.me/images/hit-test-depth-first-traversal.png)](http://smnh.me/images/hit-test-depth-first-traversal.png)

遍历算法开始通过发送`hitTest:withEvent:`消息到`UIWindow`，这是视图层级的根视图。此方法返回的值。是包含触摸点的最前面的视图。

下面的流程图说明了`Hit-testing`逻辑。

![](http://smnh.me/images/hit-test-flowchart.png)



下面的代码展示原生`hitTest:withEvent:`方法的可能实现方式：

```objective-c
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    if (!self.isUserInteractionEnabled || self.isHidden || self.alpha <= 0.01) {
        return nil;
    }
    if ([self pointInside:point withEvent:event]) {
        for (UIView *subview in [self.subviews reverseObjectEnumerator]) {
            CGPoint convertedPoint = [subview convertPoint:point fromView:self];
            UIView *hitTestView = [subview hitTest:convertedPoint withEvent:event];
            if (hitTestView) {
                return hitTestView;
            }
        }
        return self;
    }
    return nil;
}
```



`hitTest:withEvent:`方法首先检查视图允许接收的触摸。以下是允许视图接收触摸的情况：

- 视图未隐藏：
  `self.hidden == NO`
- 该视图已启用用户交互：
  `self.userInteractionEnabled == YES`
- 视图的alpha级别大于0.01：
  `self.alpha > 0.01`
- 视图包含point：
  `pointInside:withEvent: == YES`

然后，如果该视图允许接收触摸，这种方法向每个子视图从最后一个到第一个发送`hitTest:withEvent:`消息，来遍历接收者的子数，直至它们中一个返回非`nil`值。由其中一个子视图返回的第一个非零值是接触点下的最前面的视图，并由接收器返回。如果所有接收器子视图返回`nil`或接收器没有子视图，接收器返回自身。

另外，如果视图不允许接收触摸，此方法返回`nil`,而不会遍历子树。因此，`Hit-testing`过程可能不会访问视图层次结构中的所有视图。

### 重写 `hitTest:withEvent:`的常见应用场景

`hitTest:withEvent:`可以被重写，当想要触摸处理被重定向另一个视图，在触摸时间序列的所有阶段时。

> 因为执行`Hit-testing`，只有在触摸事件序列的第一触摸事件被发送到它的接收器（与触摸`UITouchPhaseBegan`阶段）之前，并重写`hitTest:withEvent:`，可以将重定向事件将重定向序列的所有触摸事件。

#### 增加视图的触摸区域

这可以证明重写一个用例`hitTest:withEvent:`的方法是，一个视图的触摸面积应大于它的边界大。例如，下图显示了一个`UIView`20×20的有大小。此大小可能太小，无法处理附近的触摸。因此，它的触摸区域可在每个方向通过重写`hitTest:withEvent:`的方法增加10点：

![增加触摸面积](http://smnh.me/images/hit-test-increase-touch-area.png)

```objective-c
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    if (!self.isUserInteractionEnabled || self.isHidden || self.alpha <= 0.01) {
        return nil;
    }
    CGRect touchRect = CGRectInset(self.bounds, -10, -10);
    if (CGRectContainsPoint(touchRect, point)) {
        for (UIView *subview in [self.subviews reverseObjectEnumerator]) {
            CGPoint convertedPoint = [subview convertPoint:point fromView:self];
            UIView *hitTestView = [subview hitTest:convertedPoint withEvent:event];
            if (hitTestView) {
                return hitTestView;
            }
        }
        return self;
    }
    return nil;
}
```



> 注：为了正确地对`view`进行`hit-testing`，父视图的边界应包含其子视图所需的触摸区域，或`hitTest:withEvent:`方法应该是被重写，包括所需的触摸区域。

#### 将触摸事件传递到下面的视图

有时，视图需要忽略触摸事件，并将其传递到下面的视图。例如，假设在应用程序视图上方，放置透明覆盖视图。叠加层有一些控件和按钮，作为子视图，它们应该正常响应触摸。但是触摸叠加层放入其他地方，应该将触摸事件传递到叠加层下方的视图。为了实现这一行为，重写`hitTest:withEvent:`方法，以返回其包含的接触点一个子视图，并在所有其他情况下返回`nil`，包括当覆盖层包含接触点情况下：

```ob
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    UIView *hitTestView = [super hitTest:point withEvent:event];
    if (hitTestView == self) {
        hitTestView = nil;
    }
    return hitTestView;
}

```



#### 将触摸事件传递到子视图

不同的场景，可能需要父视图将所有触摸事件重定向到其唯一的子视图。当子视图只占据其父视图一部分，但是应当响应于其父视图中发生的所有触摸时，可能需要这种行为。例如，假设由父视图和`UIScrollView`组成的图像轮播，其中`pagingEnabled`设置为`YES`和`clipsToBounds`设置为`NO`创建创建效果：

[![将触摸事件传递到子视图](http://smnh.me/images/hit-test-pass-touches-to-subviews.png)](http://smnh.me/images/hit-test-pass-touches-to-subviews.png)

为了使`UIScrollView`响应发生内部自身的边界和它的父视图的边界内的触摸，`hitTest:withEvent:`方法需要被重写：

```objective-c
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    UIView *hitTestView = [super hitTest:point withEvent:event];
    if (hitTestView) {
        hitTestView = self.scrollView;
    }
    return hitTestView;
}
```

