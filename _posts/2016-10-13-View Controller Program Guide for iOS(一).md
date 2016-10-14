---
layout: post
title: View Controller Program Guide for iOS(一)
tags: 
- UINavigationController


---

# Overview

## The Role of View Controllers - 视图控制器的角色

视图控制器是你的应用程序内部结构的基础。每个应用程序都至少有一个视图控制器，并且大多数应用程序会有多个。每个视图控制器管理你的应用程序的一部分用户界面，以及该接口和基础数据之间的交互。视图控制器也便于不同用户界面的之间的转换。

因为`view controller`在你的应用程序中扮演着重要的角色，所以它是你所做的一切的中心。`UIViewController`类定义了方法method和属性，来管理视图，处理事件，控制器切换transition，并协调应用程序的其他部分的方法和属性。你可以继承`UIViewController`，并添加你需要实现你的应用程序行为的自定义代码。

有两种类型的视图控制器的：

- `Content view controlller`*内容视图控制器*管理应用内容离散片段，并且你将会经常创建它。
- *`Container view controlller`容器视图控制器*收集来自其他视图控制器的信息（比如*子视图控制器*），并便于导航或呈现不同的视图控制器的内容的方式来呈现它。

多数应用是这两种类型的视图控制器的混合。

### View Management - 视图管理

视图控制器的最重要功能是管理视图的层级。每个视图控制器有一个包含所有视图控制器的内容的唯一的根视图。您可以添加你需要显示内容的子视图到根视图。图1-1显示了视图控制器和视图之间的内在关系。视图控制器总是对它的根视图有引用，每个视图都对其子视图强引用。

**图1-1**视图控制器和视图之间的关系![图片：../Art/VCPG_ControllerHierarchy_fig_1-1_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_ControllerHierarchy_fig_1-1_2x.png)

注意：这是常见的做法，使用[Outlets](https://developer.apple.com/library/prerelease/content/documentation/General/Conceptual/Devpedia-CocoaApp/Outlet.html#//apple_ref/doc/uid/TP40009071-CH4)来访问其他，在视图控制器的视图层级。因为一个视图控制器管理的所有`view`的内容，[Outlets](https://developer.apple.com/library/prerelease/content/documentation/General/Conceptual/Devpedia-CocoaApp/Outlet.html#//apple_ref/doc/uid/TP40009071-CH4)让你存储你所需要的视图引用。[Outlets](https://developer.apple.com/library/prerelease/content/documentation/General/Conceptual/Devpedia-CocoaApp/Outlet.html#//apple_ref/doc/uid/TP40009071-CH4)自身是连接到真实视图对象，而这些真实view是由stroryboard加载而来。

`Content view controller`本身管理所有的视图。容器视图控制器`Container view controller`管理自身的views，以及一个或多个根视图（其子视图控制器的`root controller`）。容器不管理其子项的内容。它只能管理根视图的大小size和位置（x,y)。图1-2显示了一个拆分视图控制器及其子控制器之间的关系。拆分视图控制器管理其所有子视图的大小和位置，但子视图控制器管理的这些view的实际内容。

**图1-2**视图控制器可以管理其他视图控制器的内容![图片：../Art/VCPG_ContainerViewController_fig_1-2_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_ContainerViewController_fig_1-2_2x.png)

有关管理控制器的视图的信息，请参阅 [Managing View Layout](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/DefiningYourSubclass.html#//apple_ref/doc/uid/TP40007457-CH7-SW6)。

### 数据编排

视图控制器是管理的view和您的应用程序的数据之间的中介。 `UIViewController`类的方法和属性，让你管理你的应用程序的视觉呈现。当你继承`UIViewController`，你添加你需要在你的子类来管理您的数据的任何变量。添加自定义变量，如图1-3，其中视图控制器对数据和view引用。两者之间数据传递是控制器的任务。

**图1-3**视图控制器的数据对象和视图之间起中介作用![图片：../Art/VCPG_CustomSubclasses_fig_1-3_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_CustomSubclasses_fig_1-3_2x.png)

你应该将视图控制器和数据对象职责完全分离。大多数用于确保数据结构的完整性的逻辑所属的数据对象本身。视图控制器可以验证从输入的意见来，然后打包你的数据对象要求的格式该输入，但你应该尽量减少视图控制器在管理中的实际数据的作用。

一个`UIDocument`目的是从你的视图控制器分开管理自己的数据的一种方法。文档对象是一个知道如何读取和写入数据，永久存储控制器对象。当你继承，您可以添加任何逻辑和方法，你需要提取数据，并将其传递给视图控制器或您的应用程序的其他部分。视图控制器可以存储接收，使其更容易地更新视图的任何数据的副本，但文件仍拥有真正的数据。

### 用户交互

视图控制器是[响应者对象](https://developer.apple.com/library/prerelease/content/documentation/General/Conceptual/Devpedia-CocoaApp/Responder.html#//apple_ref/doc/uid/TP40009071-CH1)，并能够处理响应链事件。虽然他们有能力这样做，视图控制器很少直接处理触摸事件。相反，通常的view处理自己的触摸事件，并通过[delegate](https://developer.apple.com/library/prerelease/content/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html#//apple_ref/doc/uid/TP40008195-CH14)或target，把结果回传给view controller。因此，在视图控制器大多数事件是处理 delegate和action方法。

有关在您的视图控制器实现action方法的详细信息，请参阅[Handling User Interactions](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/DefiningYourSubclass.html#//apple_ref/doc/uid/TP40007457-CH7-SW11)。有关处理其他类型的事件的信息，请参阅*Event Handling Guide For iOS*。

### 资源管理

视图控制器承担view，它创建的任何对象的所有责任。`UIViewController`类自动处理视图管理的大多数方面。例如，UIKit会自动释放那些不再需要的任何视图相关的资源。在你`UIViewController`的子类，你需要负责管理你明确创建的任何对象。

当可用内存不足时，UIKit的应用要求释放无效的资源。它这样做的一种方法是通过调用`didReceiveMemoryWarning`视图控制器的方法。使用该方法来消除的引用(你不再需要的对象，或是以后就能简单重新创建的对象)。例如，您可以使用该方法来删除缓存数据。这个方法，当内存不足情况发生时释放尽可能多的内存是非常重要的。消耗太多内存的应用可能会直接被系统终止，来恢复内存。

### 适配

视图控制器负责view的展示，并适配不同环境。每一个iOS应用程序应该能够在iPad上，并在几个不同尺寸的iPhone上运行。而不是为每个设备提供不同的视图控制器和视图层级，应该是能够适应空间需求而view不断变化的单一视图控制器。

在iOS系统中，视图控制器需要处理粗粒度`coarse-grained`的变化和细粒度`fine-grained`的变化。当一个视图控制器的性状改变，fine-grained也随之改变。性状是描述整体环境，诸如显示刻度的属性。两个最重要的特征是视图控制器的水平和垂直尺寸，这表明在给定的尺寸，视图控制器拥有多大的空间。您可以使用`size class`而改变视图的布局方式，如图1-4。当`horizontal siza class`是*regular*，视图控制器会额外水平空间的优势，安排其内容。当`horizontal siza class`是*compact*，视图控制器垂直排列其内容。

**图1-4**适应视图大小变化级![图片：../Art/VCPG_SizeClassChanges_fig_1-4_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_SizeClassChanges_fig_1-4_2x.png)

在给定的size class，有可能在任何时候发生`fine-grained`的变化。当用户旋转iPhone从竖屏到横屏，size class可能不会改变，但屏幕尺寸通常会改变。当您使用`Auto Layout`，UIKit会自动调整大小和子视图，以匹配新的尺寸位置。根据需要，控制器可以做出更多的调整。

有关自适应的详细信息，请参阅[The Adaptive Model](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/TheAdaptiveModel.html#//apple_ref/doc/uid/TP40007457-CH19-SW1)。

## 视图控制器层级

您的应用程序的视图控制器之间的关系，定义每个视图控制器所需的行为。UIKit的希望您能在规定的方式使用视图控制器。保持适当的视图控制器的关系，确保自动行为传递到正确的视图控制器在需要的时候。如果你违反了规定的包含和演示的关系，你的应用程序的某些部分会停止。

### 根视图控制器

根视图控制器是视图控制器层次结构的锚。每个窗口都一个根视图控制器(其内容填充该窗口)。根视图控制器定义了用户看到的初始内容。图2-1显示了根视图控制器和窗口之间的关系。因为窗口有没有自己的可视内容，视图控制器的视图提供所有内容。

**图2-1**根视图控制器![图片：../Art/VCPG-root-view-controller_2-1_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG-root-view-controller_2-1_2x.png)

根视图控制器是访问`UIWindow`对象的属性`rootViewController`。当您使用故事板来配置你的视图控制器，UIKit会在启动时会自动设置该属性的值。对于代码方式创建的窗口，你必须自己手动赋值该属性。

### 容器视图控制器

容器视图控制器让你从组装更易于管理和可重用代码的复杂的界面。容器视图控制器具有可选的自定义视图混合在一起的一个或多个子视图控制器的内容，以创建最终的接口。例如，一个`UINavigationController`对象显示子视图控制器的内容，连同导航栏和可选的工具栏(这些由导航控制器管理)。UIKit中包括几个容器视图控制器，包括`UINavigationController`，`UISplitViewController`，和`UIPageViewController`。

容器视图控制器的view总是充满给它的空间。容器视图控制器通常安装在一个窗口的根视图控制器（如图2-2），但它们也可以模态呈现或安装为其他容器的子控制器。容器负责适当定位其子视图。在该图中，该容器放置在两个紧邻的子视图。

**图2-2**的容器充当根视图控制器![图片：../Art/VCPG-container-acting-as-root-view-controller_2-2_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG-container-acting-as-root-view-controller_2-2_2x.png)

由于容器视图控制器管理其子控制器，UIKit的定义如何设置自定义容器那些孩子的规则。有关如何创建自定义的容器视图控制器的详细信息，请参阅[Implementing a Container View Controller](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/ImplementingaContainerViewController.html#//apple_ref/doc/uid/TP40007457-CH11-SW1)。

### present视图控制器

呈现一个新的视图控制器，替换当前视图控制器的内容，通常是隐藏以前的视图控制器的内容。presentation最常用的方法是模态显示新的内容。例如，您可能会出现一个视图控制器从用户那里收集输入。你也可以用它们作为通用构建模块为您的应用程序的界面。

当你present一个视图控制器，UIKit中创建的关系*presenting view controller*和*presented view controller*，如图2-3。这些关系构成了视图控制器的层次结构的一部分，并且在运行时定位其他视图控制器的方法。

**图2-3**主办视图控制器![图片：../Art/VCPG-presented-view-controllers_2-3_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG-presented-view-controllers_2-3_2x.png)

当容器视图控制器都参与其中，UIKit中可以修改present链，来简化代码。不同呈现方式对他们如何出现在屏幕上，例如，一个全屏幕演示总是覆盖整个屏幕不同的规则。当你提出一个视图控制器，UIKit中查找一个视图控制器，提供了演示一个合适的环境。在许多情况下，UIKit的选择最近的容器视图控制器，但它也可能选择窗口的根视图控制器。在某些情况下，你也可以告诉它的UIKit视图控制器定义演示环境，并应处理表现。

图2-4显示了为什么容器通常提供了一个展示的环境。当执行一个全屏幕演示，新的视图控制器需要覆盖整个屏幕。而不是要求孩子以了解它的容器的边界，容器决定是否处理表现。因为本例中的导航控制器覆盖整个屏幕，它充当呈现视图控制器和启动演示。

**图2-4**present 一个视图控制器![图片：../Art/VCPG-container-and-presented-view-controller_2-4_2x.png](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG-container-and-presented-view-controller_2-4_2x.png)

有关演讲的信息，请参阅[The Presentation and Transition Process](https://developer.apple.com/library/prerelease/content/featuredarticles/ViewControllerPGforiPhoneOS/PresentingaViewController.html#//apple_ref/doc/uid/TP40007457-CH14-SW7)。