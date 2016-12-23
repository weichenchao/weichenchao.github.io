---
layout: post
title: Building an Adaptive Interface
tags: View Controller Programming Guide for iOS
- app
---

## 构建自适应界面

自适应界面应响应trait和大小的变化。从视图控制器级别，使用trait，粗略确定显示的内容和内容的布局。例如，size class改变时，您可以选择更改视图属性，显示或隐藏视图，或显示完全不同的视图。做出这些重大决定之后，可以使用大小的变化进行微调您的内容。

### 适应trait的变化

Traits提供了一种为因环境不同而配置应用程序不同的方法，您可以使用它们对界面进行粗调。使用traits进行的大部分更改可以直接在故事板文件中完成，但有些需要额外的代码。

### 配置故事板，来处理不同尺寸的类

Interface Builder使您可以轻松地将界面适应不同size class。故事板编辑器包括支持以不同大小类配置显示您的界面，用于删除特定配置中的视图以及指定不同的布局约束。您还可以创建针对不同大小类别提供不同图像的图片资源。使用这些工具意味着您不必在运行时以编程方式进行相同的更改。相反，UIKit会在当前大小类更改时自动更新您的界面。

图13-1显示了在Interface Builder中用于配置接口的工具。大小类查看控件会更改您的界面的外观。使用该控件来查看您的界面将如何查找给定的大小类。对于单个视图，使用安装控件配置是否存在给定大小类配置的视图。使用复选框左侧的加号（+）按钮添加新配置。

**图13-1**自定义为不同大小的类接口![图片：../Art/size_class_ib_setting_2x.png](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/size_class_ib_setting_2x.png)

> 注意
>
> 卸载的视图会留在你的视图层次中，并且可以正常操作，但它们不会显示在屏幕上。

图片资源是存储应用程序图片资源的首选方式。 每个图片资源包含同一图片的多个版本，每个版本都针对特定配置进行设计。 除了为标准和Retina显示指定不同的图像，还可以为不同的水平和垂直尺寸类别指定不同的图像。 配置图像资源时，UIImageView对象会自动选择与当前size class和分辨率相关联的图像。

图13-2显示了图像资产属性。 更改width和height属性，会在目录中添加更多图像的插槽。 用图像填充这些插槽，以用于每个尺寸类组合。

**图13-2**为不同规模类配置图像资源![图片：../Art/size_class_image_asset_2x.png](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/size_class_image_asset_2x.png)

### 更改子控制器的Trait

默认情况下，子控制器继承其父视图控制器的trait。 对于size class，子控制器与父控制器具有相同的trait无意义。 例如，常规环境中的视图控制器可能想要向其一个或多个子项指定compact size class，以反映该子项的减少的空间量。 实现容器控制器时，通过调用容器视图控制器的`setOverrideTraitCollection：forChildViewController：`方法来修改子控制器的traits。

清单13-1显示了如何创建一组新trait并将它们与子视图控制器相关联。 您从父视图控制器执行此代码，只需要这样做一次。 覆盖的特性会保留在子代中，直到您再次更改它们或直到您从视图控制器层次结构中删除子代。

**清单13-1**更改子视图控制器的特点

```objective-c
UITraitCollection* horizTrait = [UITraitCollection
                 traitCollectionWithHorizontalSizeClass:UIUserInterfaceSizeClassRegular];
UITraitCollection* vertTrait = [UITraitCollection
                 traitCollectionWithVerticalSizeClass:UIUserInterfaceSizeClassCompact];
UITraitCollection* childTraits = [UITraitCollection
                 traitCollectionWithTraitsFromCollections:@[horizTrait, vertTrait]];
 
[self setOverrideTraitCollection:childTraits forChildViewController:self.childViewControllers[0]];
```

当父控制器的trait改变时，子控制器会继承（父级没有显式覆盖的）trait。例如，当父级的水平size class从regular变为compact时，上例中的子控制器继续保留其regular horizontal size class。但是，如果`displayScale trait`更改，则子控制器会重新继承新值。

### 让呈现的视图控制器适应新样式

呈现的视图控制器可以在regular和compact环境之间自动适配。当从水平regula环境转换到水平compact环境时，UIKit默认将内置呈现风格更改为`UIModalPresentationFullScreen`。对于自定义展示样式，呈现的控制器可以确定适配行为，并相应地调整展示。

对于某些应用，适配全屏样式可能会出现问题。例如，弹出窗口通常通过在其边界外部敲击来移除，但是在compact环境中弹出窗口覆盖整个屏幕，这样做是不可能的，如图13-3所示。当默认的适应风格不适当时，你可以告诉UIKit使用不同的风格或呈现不同的视图控制器，更好地适合全屏风格。

**图13-3** regular和compact的环境中弹窗![图片：../Art/VCPG_popover-in-regular-and-compact-views_13_3_2x.png](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_popover-in-regular-and-compact-views_13_3_2x.png)

要更改呈现样式默认的适配行为，为相关的`presentation controller`指定`delagate`。您可以使用`presented controller`的`presentationController`属性,访问`presentation controller`。在进行适配之前，`presentation controller`会咨询你的`delegate`。代理返回与默认不同的呈现样式，并且它向`presentation controller`提供要替代的控制器，来显示。

使用委托的`adaptivePresentationStyleForPresentationController：`方法，来指定与默认不同的展示样式。当转换到compact的境，唯一支持的样式是全屏样式或`UIModalPresentationNone`。返回`UIModalPresentationNone`告诉呈现控制器忽略compact环境，并继续使用先前的呈现样式。在弹出窗口的情况下，忽略更改，会给所有设备上类似iPad的popover行为。图13-4并排显示了默认的全屏自适应和没有适配，你可以对比两者。

**图13-4**更改适应行为的呈现视图控制器![图片：../Art/VCPG_changing-adaptive-behavior-for-presented-view-controller_13-4_2x.png](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_changing-adaptive-behavior-for-presented-view-controller_13-4_2x.png)

要完全取代视图控制器，实现委托的`presentationController:viewControllerForAdaptivePresentationStyle:`方法。当适配compact环境，你可使用这个方法来插入导航控制器到您的视图层次，或加载是专门为更小的空间设计的视图控制器。

### 实现自适应的提示`

当从水平regular变为水平compact时，需要额外的修改。水平compact的默认行为会将其更改为全屏演示。因为`Popovers`通常通过点击弹出框的边界以外的方式来关闭，全屏显示会无法关闭`Popovers`。通过执行以下操作来补偿：

- **将`Popovers`所属的视图控制器插入到某个现有的导航堆栈。**当有一个父导航控制器，关闭`Popovers`，并push`Popovers`所属的视图控制器到导航堆栈。
- **全屏时，添加控件来关闭`Popovers`。**您可以将控件添加到`Popovers`的视图控制器，或者，有更好的选择是使用导航控制器`presentationController:viewControllerForAdaptivePresentationStyle:`的方法，替代`Popovers`。使用导航控制器为您提供了一个模态界面模式接口和空间，来添加一个完成按钮或其他控件关闭内容。
  - **使用presentation controller的委托，消除适配的改变。**获取`Popoverspresentation controller`，指定这个控制器的`delegate`，并实现委托方法`adaptivePresentationStyleForPresentationController:`。上述方法返回`UIModalPresentationNone`，会导致`Popovers`继续显示为`Popovers`。欲了解更多信息，请参阅[Adapting Presented View Controllers to a New Style](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/BuildinganAdaptiveInterface.html#//apple_ref/doc/uid/TP40007457-CH32-SW6)。

### 应对大小的变化

尺寸的变化，可能会发生的原因有很多，其中包括：

- 底层窗口的变化，通常是因为改变方向。
- 父视图控制器重新调整布局子控制器。
- `presentation controller`改变其`presented controller`的大小。

当大小改变时，UIKit通过正常布局过程，自动更新可见视图控制器层次结构的大小和位置。 如果您使用“自动布局”限制指定视图的大小和位置，则您的应用会自动适应任何大小更改，并应在具有不同屏幕尺寸的设备上运行。

如果Auto Layout约束不足以达到你想要的效果，你可以使用`viewWillTransitionToSize:withTransitionCoordinator:`的方法，来更改您的布局。您也可以使用该方法来创建更多的附加动画，与大小变化的动画同时运行。例如，界面旋转过程中，可以使用转换协调器的`targetTransform`属性，来创造界面某些部分的反旋转矩阵。