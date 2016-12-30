---
layout: post
title: View Controller Catalog for iOS(四)-Split View Controllers
tags: 
- View Controller Catalog for iOS
---

# 分屏控制器

`UISplitViewController`类是管理两个窗格信息的*容器视图控制器*。第一个窗格具有320点的固定宽度，以及与`visible window`相等的高度。第二个窗格填充剩余的空间。 图4-1显示了一个分屏控制器界面。

**图4-1**   分屏视图界面![img](https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/splitview_master.png)

`split view`界面的窗格包含视图控制器管理的内容。由于该窗格包含应用程序的具体内容，所以由你来管理这两个视图控制器之间的相互作用。然而，旋转和其它系统相关的行为由`split view controller`本身管理。

`split view controller`必须始终在创建任何界面的`root`。换言之，你必须始终安装`UISplitViewController`对象的视图，作为你的应用程序窗口的根视图。`split view controller`的窗格则可能包含导航控制器，标签栏控制器，或者您需要实现界面的任何其他类型的视图控制器。`split view controller`不能模态呈现。

将`split view controller`集成到应用程序中最简单的方法是从一个新项目开始。在Xcode中基于`split view`的应用程序模板，为构建集成`split view controller`的界面提供了一个很好的起点。实现`split view controller`界面需要的一切都已具备。你所要做的就是，修改视图控制器的数组来呈现您的内容。修改这些视图控制器的过程是iPhone应用程序的过程中使用是相同的。唯一的区别是，现在有更多的屏幕空间，可用于显示您的详细相关内容。但是，您也可以将`split view controller`集成到现有的界面。

## 使用Storyboard创建`split view controller`

如果要创建一个新的Xcode项目，Master-Detail Application template已经在storyboard包含了split view，设置为第一个场景。

将`split view controller`添加到现有的应用程序：

1. 打开应用程序的`main storyboard`。

2. 从library 拖拽出`split view controller`。

   Interface Builder中创建一个分屏控制器，导航控制器和视图控制器，并创建它们之间的关系。这些关系确定新创建的视图控制器，作为分割视图控制器的左边和右边窗格。

3. 通过在`Attributes inspector`中选择`Is Initial View Controller`，将它作为第一个视图控制器，来显示（或以另一种方式在用户界面中呈现视图控制器）。

管理在拆分视图中嵌入的两个视图控制器的内容是你的责任。您配置这些视图控制器，就像你会在你的应用程序配置其他视图控制器。例如，对于嵌入的导航和标签栏控制器，你可能需要指定其他视图控制器的信息。

## 代码方式创建`split view controller`

以编程方式创建`split view controller`，创建`UISplitViewController`类的实例，并将`view controller`赋值给其两个属性。因为它的内容是从您提供的视图控制器即时建立，因此创建一个拆分视图控制器时，你不必指定`nib file`。因此，你可以只使用`init`方法来初始化它。清单4-1显示了在启动时如何创建和配置分屏视图界面的例子。用你的应用程序的内容视图控制器对象替换第一和第二视图控制器。`window`变量被认为是一个输出，指向[窗口](undefined)从应用程序的主nib文件加载。

**清单4-1**   编程方式创建一个拆分视图控制器

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
   MyFirstViewController* firstVC = [[MyFirstViewController alloc] init];
   MySecondViewController* secondVC = [[MySecondViewController alloc] init];
 
   UISplitViewController* splitVC = [[UISplitViewController alloc] init];
   splitVC.viewControllers = [NSArray arrayWithObjects:firstVC, secondVC, nil];
 
    window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    window.rootViewController = splitVC;
   [window makeKeyAndVisible];
 
   return YES;
}

```



## 在拆分视图支持方向变化

`split view controller`依赖于它的两个被包含的视图控制器，来确定方位支持哪些。如果被包含视图控制器同时支持一个方向，程序才支持。即使视图控制器当前并不显示，它也必须支持此方向。当方向发生变化，`split view controller`自动处理大部分的旋转行为。

横屏时，`split view controller`并排呈现两个窗格，并使用分割线将它们分开。竖屏时，`split view controller`要么显示出两个窗格，要么仅示出了第二个较大的窗格，且提供一个工具栏按钮，用于使用popover，具体取决于由`splitViewController:shouldHideViewController:inOrientation:` [委托](undefined)方法返回的值。