---
layout: post
title: The Adaptive Model
tags: View Controller Programming Guide for iOS
- app

---

## 自适应模型

自适应界面是对可用空间的最佳利用。自适应意味着能够调整您的内容，使其恰当显示在任何iOS设备。 iOS中的自适应模型支持简单且动态的方式，来重新排列和调整内容以响应更改。 当您利用此模型时，只需很少的额外代码，应用程序就可以适应显着不同的屏幕大小（如图12-1所示）。

**图12-1** 适应不同的设备和方向![图片：../Art/VCPG_AdaptiveModel_13-1_2x.png](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_AdaptiveModel_13-1_2x.png)

用于构建自适应界面的重要工具是Auto Layout。 使用“Auto Layout”，可以定义控制视图布局的规则（称为约束）。 您可以在Interface Builder中以可视方式创建这些约束，或在代码中以编程方式创建这些约束。 当父视图的大小更改时，iOS会根据您指定的约束自动调整其大小并重新定位其余视图。

Traits是自适应模型的另一个重要组成部分。 特征描述了视图控制器和视图必须操作的环境。 Traits帮助您做出关于界面的高级决策。

### 性状的作用

当只有约束不足以管理布局时，您的视图控制器有几个机会进行更改。 视图控制器，视图和一些其他对象管理Traits，这些特征明确与该对象关联的当前环境。 表12-1介绍了Traits以及如何使用Traits来影响用户界面。

| 特征                    | 例子                                | 描述                                       |
| --------------------- | --------------------------------- | ---------------------------------------- |
| `horizontalSizeClass` | `UIUserInterfaceSizeClassCompact` | 这种特征代表界面的常规宽度。用它来作粗略的布局决策，如视图是否垂直堆叠，并排显示，完全隐藏，或通过其他方法显示出来。 |
| `verticalSizeClass`   | `UIUserInterfaceSizeClassRegular` | 这种特征传达你的接口的常规高度。如果您的设计需要所有的内容适配在屏幕上，而不滚动，利用这一特征进行布局决策。 |
| `displayScale`        | `2.0`                             | 这种特征传达内容是否显示在视网膜显示器或标准分辨率显示器上。 使用它（根据需要）进行像素级布局决定或选择要显示的图像的版本。 |
| `userInterfaceIdiom`  | `UIUserInterfaceIdiomPhone`       | 提供此特征是为了向后兼容，并传达您的应用程序运行的设备类型。 尽可能避免使用这个特征。 对于布局决定，请改用水平和垂直大小类。 |

使用Traits来决定如何呈现您的用户界面。在Interface Builder中构建界面时，使Ttraits更改您显示的视图和图像，或使用它们应用不同的约束集。许多UIKit类，如`UIImageAsset`，也做了Traits的支持。（在 Image Asset 的编辑面板中选择某张图片，Inspector 里现在多了一个 Width 和 Height 的组合，添加我们需要对应的 Size Class， 然后把合适的图拖上去，这样在运行时 SDK 就将从中挑选对应的 Size 的图进行替换了。）

这里有一些提示，帮助您了解什么时候使用不同类型的Traits：

- **使用Size Classes对界面粗略适配。**Size Classes改变是添加或删除视图，添加或删除子视图控制器，或更改布局约束的合适时机。你也可以什么也不做，使用其现有的布局约束让你的界面自动适应。
- **永远不要假设Size Classes对应于视图的特定宽度或高度。**您的视图控制Size Classes因多种情况而改变。例如，iPhone上的容器视图控制器可能会使它的子控制器水平，迫使它以不同的方式显示其内容。
- **使用Interface Builder，为每个Size Class指定不同的布局约束。**使用Interface Builder来指定的约束比手写代码添加和删除约束要简单得多。视图控制器通过从故事板加载适当的约束，自动处理Size Class的变化。
- **避免使用习语，来决定界面的布局或内容。**iPad和iPhone上运行的应用程序，一般应显示相同的信息，并应使用大小类进行布局的决定。

### 当Traits和大小的变化发生？

Trait很少改变，但他们确实也会发生改变。UIKit基于底层环境的更改，来更新控制器的trait。尺寸类的trait比显示scale的trait更可能变化。习语特征应该很少，如果有的话，变化。由于以下原因发生size class改变：

- 垂直或水平尺寸类视图控制器的窗口的改变，通常是因为设备的旋转。
- 水平或垂直尺寸类的容器视图控制器的改变。
- 水平或垂直大小类当前视图控制器是由它的容器明确更改。

视图控制器层次中的Size class更改，会继承到任何子视图控制器。 窗口对象是层次结构的根，为其根视图控制器提供基线 size class traits。 当设备方向在竖屏和横屏之间变化时，窗口更新其自己的size class信息，并将该信息沿着视图控制器层次结构传播。 容器视图控制器可以将修改传递给未修改的子视图控制器，或者重写每个子对象的trait。

在iOS 8及更高版本中，当设备在横屏和纵屏方向之间旋转时，窗口始终位于左上角，窗口的bounds将更改。 窗口大小更改以及任何对应的特性更改，沿着视图控制器层次结构一起传播。 对于层次结构中的每个视图控制器，UIKit调用以下方法来报告这些更改：

1. `willTransitionToTraitCollection:withTransitionCoordinator:`讲述每个相关视图控制器，其特征即将改变。
2. `viewWillTransitionToSize:withTransitionCoordinator:`讲述每个相关视图控制器，其大小是即将改变。
3. `traitCollectionDidChange:`告诉每一个相关的视图控制器，特征集合已改变。

当沿着视图控制器层次结构移动时，UIKit仅在报告更改时才将更改报告给视图控制器。 如果容器视图控制器覆盖其子控制器的size class ，那么当容器的大小类更改时不会通知这些子控制器。 类似地，如果视图控制器的视图具有固定的宽度和高度，则它不会接收大小更改通知。

图12-2显示了当iPhone 6上发生旋转时，视图控制器的trait和视图大小如何更新。从纵屏到横屏的旋转，将屏幕的垂直size class从常规更改为紧凑。 大小类别改变和对应的视图大小改变然后沿着视图控制器层级传播。 在将视图动画化为新size后，UIKit在调用视图控制器的`traitCollectionDidChange：`方法之前,应用大小类和视图大小更改。

**图12-2**更新视图控制器的特点和视图大小![图片：../Art/VCPG_RotationTraits_fig_13-2_2x.png](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_RotationTraits_fig_13-2_2x.png)

### 不同的设备的默认size class

每个iOS设备都有一组默认的大小类，您可以在设计界面时将其用作指南。 表12-2列出了纵向和横向设备的大小类别。 表中未列出的设备与具有相同屏幕尺寸的设备具有相同的大小类别。

| Device                     | Portrait                                 | Landscape                                |
| -------------------------- | ---------------------------------------- | ---------------------------------------- |
| iPad (all)iPad Mini        | Vertical size class: Regular       Horizontal size class: Regular | Vertical size class: Regular     Horizontal size class: Regular |
| iPhone 6 Plus              | Vertical size class: Regular       Horizontal size class: Compact | Vertical size class: Compact  Horizontal size class: Regular |
| iPhone 6                   | Vertical size class: Regular       Horizontal size class: Compact | Vertical size class: Compact  Horizontal size class: Compact |
| iPhone 5siPhone 5ciPhone 5 | Vertical size class: Regular       Horizontal size class: Compact | Vertical size class: Compact  Horizontal size class: Compact |
| iPhone 4s                  | Vertical size class: Regular       Horizontal size class: Compact | Vertical size class: Compact  Horizontal size class: Compact |

