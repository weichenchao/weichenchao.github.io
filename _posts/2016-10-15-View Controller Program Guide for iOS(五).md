---
layout: post
title: View Controller Program Guide for iOS(五)
tags: 
- UINavigationController


---

## 适配模型-The Adaptive Model

适配界面是一个对可用空间的最佳利用。作为自适应意味着能够调整内容适配任何iOS设备。在iOS中的自适应模型`Adaptive Model`用简单且动态的方法来重新排列，并调整内容来应对变化。当你利用这个模式，一个单一的应用程序用很少的额外代码，就能适应不同的屏幕尺寸（如图12-1）。

**图12-1**适配不同的设备和方向![图片：../Art/VCPG_AdaptiveModel_13-1_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_AdaptiveModel_13-1_2x.png)

一个适配屏幕的重要工具是自动布局。使用自动布局，您可以定义规则（被称为*约束Constraints*）控制控制器的视图布局。你可以在Interface Builder或以代码方式创建约束。当父视图的大小改变时，iOS根据你指定的约束，自动调整大小和重新定位。

特性`trait`是上述自适应模型的另一个重要组成部分。性状描述你的控制器和视图必须运行的环境。性状帮助您在你的界面做最高决策。

### 特性的作用

为了表征 Size Classes，Apple 在 iOS 8 中引入了一个新的类--UITraitCollection。这个类封装了像水平和竖直方向的 Size Class 等信息。iOS 8 的 UIKit 中大多数 UI 的基础类 (包括 UIScreen，UIWindow，UIViewController 和 UIView) 都实现了 UITraitEnvironment 这个接口，通过其中的 traitCollection 这个属性，我们可以拿到对应的 UITraitCollection 对象，从而得知当前的 Size Class，并进一步确定界面的布局。

和 UIKit 中的响应者链正好相反，traitCollection 将会在 view hierarchy 中自上而下地进行传递。对于没有指定 traitCollection 的 UI 部件，将使用其父节点的 traitCollection。这在布局包含 childViewController 的界面的时候会相当有用。在 UITraitEnvironment 这个接口中另一个非常有用的是 -traitCollectionDidChange:。在 traitCollection 发生变化时，这个方法将被调用。在实际操作时，我们往往会在 ViewController 中重写 -traitCollectionDidChange: 或者 -willTransitionToTraitCollection:withTransitionCoordinator: 方法 (对于 ViewController 来说的话，后者也许是更好的选择，因为提供了转场上下文方便进行动画；但是对于普通的 View 来说就只有前面一个方法了)，然后在其中对当前的 traitCollection 进行判断，并进行重新布局以及动画。

当仅有约束是不够管理布局，您的视图控制器还有几个机会做出改变。视图控制器，视图和其他几个对象管理*特性集合traits collection*（指定与该对象关联的当前的环境）。表12-1描述的特点，以及如何使用它们来影响你的用户界面。

| 特征                    | 例子                                | 描述                                       |
| --------------------- | --------------------------------- | ---------------------------------------- |
| `horizontalSizeClass` | `UIUserInterfaceSizeClassCompact` | 这种特性传达你界面的常规宽度。用它来作大致的布局决策，如视图是否垂直堆叠，并排显示，完全隐藏，或通过其他方法显示出来。 |
| `verticalSizeClass`   | `UIUserInterfaceSizeClassRegular` | 这种特性传达你界面的一般高度。如果您的设计需要所有的内容，以适应屏幕上而无需滚动，利用这一特点进行布局的决定。 |
| `displayScale`        | `2.0`                             | 这种特性传达的内容，是否显示Retina显示屏或标准分辨率的显示屏上。使用它（如果需要），以像素级的布局决定或选择要显示的图像的版本。 |
| `userInterfaceIdiom`  | `UIUserInterfaceIdiomPhone`       | 这个特性提供了向后兼容性和传达上，你的应用程序正在运行的设备类型。避免使用这种特质尽可能。布局决定，使用水平和垂直尺寸的类来代替。 |

使用特性做出关于如何呈现用户界面的决定。当建立在Interface Builder的界面中，使用特性改变的view和将于显示的图形。如图许多UIKit类一样，如`UIImageAsset`，量身定制（他们提供来指定的特性）的信息。

这里有一些提示，帮助您了解什么时候使用不同类型的特性：

- **使用`size classes`粗略更改您的界面。`size classes`**的变化是适当的时候来添加或删除view，添加或删除子视图控制器，或更改布局约束。你也可以什么也不做，让你的界面其现有的布局约束，自动适应使用。
- **永远不要假设某个尺寸级别对应某个视图的特定宽度或高度。**您的视图控制器“size class可能因为各种原因而改变。例如，iPhone上的容器视图控制器可能会使它的子控制器水平regular，迫使它以不同的方式显示其内容。
- **使用Interface Builder为每个size class指定不同的布局约束。**请详见 [Configuring Your Storyboard to Handle Different Size Classes](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/BuildinganAdaptiveInterface.html#//apple_ref/doc/uid/TP40007457-CH32-SW2)。
- **避免使用惯用信息来决定你的界面的布局或内容。**在iPad和iPhone上运行的应用程序应显示相同的信息，并应使用size class进行布局。

### 当你的特性和大小的变化发生？

很少发生变化特征，但他们确实会发生。UIKit的更新的基础上改变底层环境视图控制器的特点。尺寸类的性状更可能比显示刻度性状发生变化。成语特质应该很少，如果有的话，变化。由于以下原因发生大小变化类：

- 垂直或水平尺寸类视图控制器的窗口的改变通常是因为设备的旋转。
- 水平或垂直尺寸类的容器视图控制器的改变。
- 水平或垂直大小类当前视图控制器是由它的容器明确更改。

在视图控制器的层次结构尺寸课改向下传播任何子视图控制器。窗口对象可以充当层次结构的根，为它的根视图控制器基准大小类特征。在纵向和横向之间的设备方向的变化，窗口更新自己的尺寸级别的信息和传播信息下视图控制器层次。容器视图控制器可以通过更改子视图控制器未修改或者他们可以覆盖每个孩子的特质。

在iOS系统中8和更高版本，窗口原点总是在左上角，当装置横向和纵向之间旋转窗口的边界改变。窗口大小变化与任何相应的性状变化而向下传播视图控制器层次。在层次中的每个视图控制器，UIKit会调用下面的方法报告这些变化：

1. 在`willTransitionToTraitCollection:withTransitionCoordinator:`讲述每个相关视图控制器，其特征是即将改变。
2. 在`viewWillTransitionToSize:withTransitionCoordinator:`讲述每个相关视图控制器，它的大小是即将改变。
3. 在`traitCollectionDidChange:`告诉它的特点，现在已经改变了每一个相关的视图控制器。

当走在视图控制器层次结构，UIKit中时，才会有一个变化报告报告更改视图控制器。如果容器视图控制器覆盖了大小班的孩子，那些孩子不通知容器的大小变化类时。同样，如果一个视图控制器的看法有一个固定的宽度和高度，没有收到大小变化的通知。

图12-2显示了如何当一个旋转在iPhone上6.纵向到横向的旋转改变垂直尺寸级屏幕与普通紧凑发生视图控制器的特点和视图大小被更新。尺寸规格的变化和相应的视图大小的变化，然后向下传播视图控制器层次。动画视图以新的大小后，UIKit会调用视图控制器的应用之前规模类和视图大小的变化`traitCollectionDidChange:`的方法。

**图12-2**更新视图控制器的特点和视图大小![图片：../Art/VCPG_RotationTraits_fig_13-2_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_RotationTraits_fig_13-2_2x.png)

### 默认大小类为不同的设备

每个iOS设备都有，你可以设计自己的界面时作为参考使用。大小类别的默认设置表12-2列出了大小班在纵向和横向的设备。表中未列出的设备具有相同的尺寸类作为具有相同屏幕尺寸的装置。

| 设备                         | 肖像                | 景观                  |
| -------------------------- | ----------------- | ------------------- |
| iPad的（全部）小型平板电脑            | 垂直尺寸类：常规卧式径级：正常   | 垂直尺寸类：常规卧式径级：正常     |
| iPhone 6加                  | 垂直尺寸类：常规卧式径级：结构紧凑 | 垂直尺寸类：结构紧凑卧式径级：正常   |
| iPhone 6                   | 垂直尺寸类：常规卧式径级：结构紧凑 | 垂直尺寸类：结构紧凑卧式径级：结构紧凑 |
| iPhone 5SiPhone 5Ciphone 5 | 垂直尺寸类：常规卧式径级：结构紧凑 | 垂直尺寸类：结构紧凑卧式径级：结构紧凑 |
| iPhone 4S                  | 垂直尺寸类：常规卧式径级：结构紧凑 | 垂直尺寸类：结构紧凑卧式径级：结构紧凑 |