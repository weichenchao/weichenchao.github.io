---
layout: post
title: View Controller Catalog for iOS(一) - UINavigationController
tags: 
- UINavigationController

---

## 解剖导航视图

导航控制器的主要工作是管理内容视图控制器的呈现，也负责呈现一些自定义视图。具体地讲，它呈现了一个navigation bar，其中包含一个返回按钮和一些可自定义的按钮。导航控制器也可以选择呈现toolbar以及其上填充自定义按钮。

图1-1显示了一个导航界面。在该图中，Navigation view是存储在导航控制器的`view`属性。所有的在界面上的其他view，是由导航控制器管理的不透明视图层次结构的一部分。

![图1-1 导航界面](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/NavigationViews_2x.png)

虽然navigation bar(导航栏)和toolbar(工具栏)是自定义视图，但是禁止直接修改它们在导航层次的结构。自定义这些view的唯一办法是通过`UINavigationController`和`UIViewController`类的方法。有关如何自定义navigation bar的内容，请参阅[自定义导航栏外观](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html#//apple_ref/doc/uid/TP40011313-CH2-SW3)。有关如何显示和导航界面来配置自定义toolbar项目的信息，请参阅[显示导航工具栏](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html#//apple_ref/doc/uid/TP40011313-CH2-SW4)。

## 导航接口的对象

navigation controller使用几种对象来实现导航界面。你是负责提供一些对象，其余的由导航控制器本身被创建。具体而言，您有责任提供视图控制器(你要呈现的内容)。如果你想响应navigation controller(导航控制器)的通知，您也可以提供一个delegate对象。导航控制器创建视图，如导航栏和工具栏，这是用于导航界面，它是负责管理这些视图。图1-2显示了导航控制器和这些关键对象之间的关系。

![图1-2 对象由导航控制器管理](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/nav_controllers_objects.jpg)

对于navigation bar和toolbar，你只能自定义其外观和行为的某些方面。导航控制器独立配置和显示它们。此外，导航控制器对象自动将其自身指定为`UINavigationBar`的代理，并禁止其他对象改变代理。

您可以修改delegate和导航堆栈上的其他view controller。该*导航堆栈*是由导航控制器管理自定义视图控制器对象的后进先出集合。添加到堆栈中的第一项将成为*根视图控制器*，并且永远不会弹出堆栈。通过使用`UINavigationController`类的方法，其他项目可以被添加到堆栈。

图1-3示出了导航控制器和导航堆栈上的对象之间的相关关系。（注意，在顶视图控制器`topViewController`和可见视图控制器`visibleViewController`不一定相同。例如，如果你是模态出目前视图控制器，`visibleViewController`属性的值会改变,来反映现在正在展示的模态视图控制器，但`topViewController`属性的值不变。）

![图1-3 导航堆栈](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/nav_controllers_stack.jpg)

导航控制器的主要职责是通过将新的内容视图控制器压入堆栈或`push`出堆栈的内容视图控制器，响应用户的操作。从航堆栈上`push`出的每个视图控制器负责展示您的应用程序数据的某些部分。通常情况下，当用户在当前可见的视图中选择一个项目，创建了具有该项目的详细信息的视图控制器，并将其`push`出导航堆栈。例如，当用户选择的照片应用程序相册，应用程序`push`出显示的照片在相册中的视图控制器。

这一过程遵循简单的设计模式，每一个在堆栈中的内容视图控制器提供配置和`push`出在堆栈顶部的内容视图控制器。你应该避免依赖于特定的类的实例被压入堆栈做一个视图控制器。相反，当`pop`出视图控制器，将数据沿着堆栈往下传，其下的视图控制器则作为delegate。

在大多数情况下，你不用用代码从堆栈弹出视图控制器。相反，导航控制器提供当用户点击它，可以自动弹出最上面的视图控制器导航栏上的后退按钮。

有关如何自定义导航栏的更多信息，请参见[自定义导航栏外观](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html#//apple_ref/doc/uid/TP40011313-CH2-SW3)。有关推视图控制器到导航堆栈的信息，请参阅[修改导航堆栈](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html#//apple_ref/doc/uid/TP40011313-CH2-SW12)。有关如何自定义工具栏的内容，请参阅[显示导航工具栏](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html#//apple_ref/doc/uid/TP40011313-CH2-SW4)。

## 创建导航界面

当你创建一个导航界面，你需要决定你打算如何使用导航界面。

- 直接作为窗口的根视图控制器安装。
- 安装它作为一个选项卡的标签栏界面视图控制器。
- 它安装在一个拆分视图`split view`界面中的两根视图控制器之一。（仅适用于iPad的）
- Modal ，从另一个视图控制器模态出。
- popover。（仅适用于iPad的）

前三情况下，导航控制器提供您的基本界面的重要组成部分，存留直到应用程序退出。最后两个场景反映导航控制器的临时使用场景，在这种情况下使用该导航控制器的过程是相同的。唯一的区别是，导航控制器继续提供不提供与单个内容视图控制器额外的导航功能。虽然下面将集中讨论如何建立更持久的类型的导航界面，大部分的定制步骤和一般信息适用所有的导航控制器。

**注：**  有关如何呈现导航控制器模态一个具体的例子，请参阅[显示导航控制器模态](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/CombiningViewControllers.html#//apple_ref/doc/uid/TP40011313-CH6-SW3)。

### 定义内容视图控制器的导航界面

每个导航界面表示同一的级别。例如，照片应用程序显示可用的相册的列表。选择一个相册，然后显示这些照片在相册，选择照片显示照片的放大版。

**图1-4**   定义视图控制器为数据的每个水平![](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/photos_app.jpg)

要实现一个导航界面，你必须决定要出席你的数据每个层级的数据。对于每个级别，你必须提供一个内容视图控制器来管理，并在这一水平呈现数据。如果多层次的呈现是一样的，你可以创建相同的视图控制器类的多个实例，并配置每一个管理自己的数据集。例如，照片应用程序有三个不同类型的演示所示，图1-4，所以需要使用三个不同的视图控制器类。

### 使用Storyboard创建导航界面

如果要创建一个新的Xcode项目，` Master-Detail Application`模板已经为你在`storyboard`创建好导航控制器，并将其设置为第一个场景。

要在`storyboard`创建一个导航控制器，请执行以下操作：

1. 从库中拖出一个导航控制器。
2. Interface Builder中创建了一个导航控制器和视图控制器，并创建它们之间的关系。这种关系确定新创建的`view controller`是`navigation controller`的`root view controller`。
3. 在属性检查器` Attributes inspector`中选择` Is Initial View Controller`属性，来展示（或以另一种方式用户界面中提供的视图控制器）。

### 代码创建导航界面

如果您希望以创建方式创建一个导航控制器，你可以在你的代码中任何合适的时机创建。例如，如果导航控制器在应用窗口提供了根视图，你可以在`applicationDidFinishLaunching:`方法中创建导航控制器](undefined)。

当[创建](undefined)一个导航控制器，你必须做到以下几点：

1. 创建导航界面的根视图控制器。

   这个目的是在导航堆栈的顶层视图控制器。导航栏显示时，显示其视图没有后退按钮和视图控制器不能从导航堆栈弹出。

2. 创建导航控制器，[初始化](undefined)使用它`initWithRootViewController:`的方法。

3. 设置导航控制器作为你的窗口（或以其他方式）的根视图控制器。

清单1-1显示了一个简单实现的`applicationDidFinishLaunching:`创建导航控制器，并将其设置为应用程序的主窗口的根视图控制器方法。`navigationController`和`window`变量是应用程序委托的成员变量，`MyRootViewController`类是一个自定义视图控制器类。当显示了这个例子的窗口，导航界面呈现在导航界面根视图控制器视图。

**清单1-1**   

| `- (void)applicationDidFinishLaunching:(UIApplication *)application` |
| ---------------------------------------- |
| `{`                                      |
| `    UIViewController *myViewController = [[MyViewController alloc] init];` |
| `    navigationController = [[UINavigationController alloc]` |
| `                                initWithRootViewController:myViewController];` |
| ` `                                      |
| `    window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];` |
| `    window.rootViewController = navigationController;` |
| `    [window makeKeyAndVisible];`        |
| `}`                                      |



## 全屏化导航视图

通常情况下，导航界面会在`navigation bar`的底部，`tool bar`或标签栏`tab bar`的顶部之间的距离，显示您的自定义内容（参见[图1-1](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html#//apple_ref/doc/uid/TP40011313-CH2-SW13)）。然而，一个视图控制器可以指定全屏幕的布局。在一个全屏幕的布局，内容视图配置为重叠导航栏，状态栏和工具栏为宜。这种安排可以让你为用户展示最大限度的内容，当你需要更多的空间时。

当确定一个视图是否应该填充全屏或大部分屏，导航控制器考虑几个因素，包括以下内容：

- 底层窗口（或父视图）大小以填满整个屏幕范围？
- 配置了导航栏半透明？
- 导航工具栏（如果使用的话），被配置为半透明？
- 底层视图控制器的`wantsFullScreenLayout`属性设置为`YES`？

这些因素的每个用于确定自定义视图的最终尺寸。上述列表中的项目的顺序也被认为是每个因素的优先级。窗口大小是上述第一限制因素; 如果您的应用程序的主窗口（或在模态呈现视图控制器的情况下，包含父视图）不会跨越屏幕，它所包含的视图也不能。同样地，如果在导航栏或工具栏是可见的，但不是半透明，如果视图控制器希望其使用全屏显示视图无关紧要。导航控制器永远不会显示一个不透明的导航栏层级之后的内容。

如果要创建一个导航界面，并且希望您的自定义内容，以跨越大部分或全部的画面，在这里，你应该采取的步骤：

1. 配置您的自定义视图的框架，以填补屏幕边界。

   一定要配置您的视图的`autoresizing`属性。该`autoresizing`属性确保如果view需要进行调整，view也会相应地调整其内容。另外，当您的视图大小改变时，可以调用`setNeedsLayout`视图的方法来指示其子视图的位置应调整。

2. 要重叠导航栏，设置`translucent` 属性`YES`。

3. 要重叠可选的工具栏，设置`translucent`属性`YES`。

4. 要重叠状态栏，设置`wantsFullScreenLayout`属性`YES`。

当展示您的导航界面，向窗口或视图添加你的导航视图，也必须大小适当。如果应用程序使用导航控制器作为其主界面，这时你的主窗口的大小应与屏幕尺寸。换句话说，你应该设置其大小为`UIScreen`的`bounds`属性（而不是`applicationFrame`属性）。

如果你模态出一个导航控制器，由导航控制器呈现的内容是被执行呈现视图控制器所限制。如果视图控制器不想重叠状态栏`toolbar`，然后被`present`模态的导航控制器，也不允许重叠状态栏。换句话说，父视图总是对被它模态出的子视图有一定的影响。

有关如何配置接口，支持全屏幕布局的更多信息，请参阅 *View Controller Programming Guide for iOS*。

## 修改导航堆栈

你是负责创建驻留在导航堆栈上的对象。当初始化导航控制器对象，你必须提供一个内容视图控制器来显示数据层次的根内容。您可以添加或删除视图控制器，以响应用户的交互。导航控制器类提供了管理导航堆栈中的内容的几个选项。这些选项包括你可能在你的应用所遇到的各种情况。表1-1列出了这些场景，你对他们如何应对。

| 脚本            | 描述                                       |
| ------------- | ---------------------------------------- |
| 显示分层数据的下一个层级。 | 当用户选择了最上面的视图控制器显示的项目，可以使用segue或`pushViewController:animated:`方法，将一个新的视图控制器`push`到导航堆栈。新视图控制器负责呈现所选项目的内容。 |
| 回到上一个层次。      | 导航控制器通常提供后退按钮，从堆栈中取出最上面的视图控制器，并返回到上一画面。您也可以以代码方式使用删除最上面的视图控制器`popViewControllerAnimated:`方法。 |
| 导航堆栈恢复到以前的状态。 | 当你的应用程序启动后，您可以使用`setViewControllers:animated:`的方法来导航控制器恢复到以前的状态。例如，您可以使用此方法向用户返回到他们在观看时，他们最后退出程序相同的屏幕。为了您的应用程序恢复到以前的状态，你必须先保存足够的状态信息重新创建所需的视图控制器。当用户退出你的应用程序，你需要节省一些标记或指示用户在数据层次中的位置等信息。在接下来的发射时间，你会再读取该状态信息，并使用它调用之前重新创建所需的视图控制器`setViewControllers:animated:`方法。有关保存和恢复状态的详细信息，请参阅  [App States and Multitasking](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4) in *App Programming Guide for iOS*. |
| 返回用户的根视图控制器。  | 要返回到您的导航界面的顶部，使用`popToRootViewControllerAnimated:`方法。此方法删除所有`controller`，只保留导航堆栈的根视图控制器。 |
| 回到任何一个层级。     | 同一时间回退n个基本，使用`popToViewController:animated:`方法。在您使用的导航控制器来管理的自定义内容编辑（而不是呈现内容模态），你可能会使用的情况下这种方法。如果用户决定取消操作已经推多个编辑屏幕后，您可以使用此方法可以一次删除所有的编辑界面，而不是一次一个。 |

当你动画地push或pop视图控制器，导航控制器会自动创建动画。例如，如果您使用弹出多个视图控制器从堆栈`popToViewController:animated:`方法，导航控制器使用动画在最上面的视图控制器。所有其他中间视图控制器被解除没有动画。如果你动画地push或pop弹出一个项目，则必须等到这个动画，才能push或pop另一个视图控制器

## 监视导航堆栈的变化

每当你push或pop出一个视图控制器，导航控制器将消息发送到受影响的视图控制器。每当堆栈变化，导航控制器也将消息发送给它的[delegate](undefined)时。图1-5显示了事件的一个PUSH或POP操作，并且在每个阶段发送到您的自定义对象相应的信息过程中发生的顺序。新的视图控制器体现了这一观点控制器即将成为堆栈上的最顶层视图控制器。

![**图1-5**   中堆栈的变化发送的邮件](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/nav_controller_notifications.png)

您可以使用导航控制器的委托方法，来协调内容视图控制器，例如，更新他们之间共享的状态。如果你一次性push或pop出多个视图控制器，只有即将可见的视图控制器和曾经可见的（即将消失）视图控制器，有调用的方法。中间视图控制器没有调用该方法，除非链接的回调里面发生（例如，如果你的实现`viewWillAppear:`通话`pushViewController:animated:`）。

您可以使用`isMovingToParentViewController`和`isMovingFromParentViewController`方法，以确定在pop或push过程中，一个视图控制器的出现或消失。

## 自定义导航栏外观

导航条是管理在导航界面中的控件的视图，导航控制器对象来管理它，并扮演着一个特殊的角色。为确保一致性，减少建立导航界面所需的工作量，每个导航控制器对象创建自己的导航栏，并负大部分责任管理该栏的内容。根据需要，导航控制器与其他对象（比如你的内容视图控制器）相互作用在这个过程中提供帮助。

**注意：**  您还可以创建导航栏作为独立意见，并使用它们。有关使用方法和属性的详细信息`UINavigationBar`类，请参阅*UINavigationBar的类参考*。

### 配置导航项目对象

导航栏的结构类似于一个导航控制器的结构。像导航控制器，导航栏是由其他对象提供的内容，的容器。在一个导航条的情况下，内容是由一个或多个提供`UINavigationItem`的对象，这是使用被称为导航项目堆栈堆栈数据结构存储。每个导航项目提供一组完整的视图和内容，在导航栏中显示。

图1-6显示了一些涉及到在运行导航栏的主要对象。导航栏的所有者（无论是导航控制器或您的自定义代码）负责项目推入堆栈，并根据需要弹出他们。为了提供正确的导航，导航栏保持指针来选择堆栈的对象。虽然大多数导航栏的内容是从最上面的导航项目获得指针到后面项目保持这样一个后退按钮（与上项目的名称）可以被创建。

![**图1-6**   与导航栏关联的对象](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/navigation_bar_objects.jpg)

**重要提示：**  当您使用导航控制器的导航栏，导航栏[delegate](undefined)总是被设置为所属导航控制器对象。试图改变delegate对象，会引发异常。

在导航界面，在导航堆栈中的每个`view controller`提供了`navigationItem`属性。naviagation堆栈和`navigation item`堆栈总是并行：在导航堆栈上的每个内容视图控制器，其`navigation item`在`navigation item`堆栈中也在相同位置。

导航条放在三个主要位置的项目：左，右和中央表1-2列出了[属性](undefined)的的`UINavigationItem`，用于配置每一个位置的类。当配置与导航控制器相关的`navigation item`，要知道，在某些位置的自定义控件可能被忽略。每个位置的描述，包括您的自定义对象是如何使用。

| 位置     | 属性                                     | 描述                                       |
| ------ | -------------------------------------- | ---------------------------------------- |
| left   | `backBarButtonItem``leftBarButtonItem` | 在导航界面时，导航控制器分配一个后退按钮，默认左侧位置。要获取导航控制器提供的默认后退按钮，通过`backBarButtonItem`属性获取。要指定自定义按钮或查看左侧位置，从而替换默认后退按钮，指定`UIBarButtonItem`对象的`leftBarButtonItem`属性。 |
| center | `titleView`                            | 在导航界面，导航控制器显示在默认情况下你的内容视图控制器的标题自定义视图。与自己的自定义视图您可以根据需要替换这一观点。如果你不提供自定义标题视图，导航栏显示与导航项目的标题字符串自定义视图。或者，如果导航项目不提供标题，导航栏使用视图控制器的标题。 |
| right  | `rightBarButtonItem`                   | 这一立场是默认为nil。它通常用于放置按键进行编辑或修改当前屏幕。您还可以将自定义视图这里由包裹在视图`UIBarButtonItem`对象。 |

图1-7显示了导航栏的内容被组装为一个导航界面。与当前视图控制器相关的导航项目提供了中心和导航栏的右边的位置的内容。前一个视图控制器的导航项目提供了左侧位置的内容。虽然左，右项目需要指定一个`UIBarButtonItem`对象，你可以包装成如图所示的栏按钮项的视图。如果你不提供自定义标题视图，`navigation item`使用当前视图控制器的标题为您创建一个。

![**图1-7**   导航栏结构](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/navbar_custom.jpg)

### 显示和隐藏导航栏

当导航栏在导航控制器一起使用时，您始终使用`setNavigationBarHidden:animated:`的方法`UINavigationController`来显示和隐藏导航栏。你千万不要通过直接修改隐藏导航栏，`UINavigationBar`对象的`hidden` [属性](undefined)。除了显示或隐藏栏，使用导航控制器的方法为您提供了更复杂的行为，是自由的。特别是，如果一个视图控制器显示或隐藏其导航栏`viewWillAppear:`的方法，新视图控制器的导航栏显示状态也跟随当前控制器。

由于用户需求的导航栏上的后退按钮导航回到前一个屏幕，你不应该隐藏导航栏，而不给用户一些方式来回到之前的屏幕。提供导航支持最常用的方法是拦截触摸事件，并用它们来切换导航栏的可见性。例如，当全屏显示一个图像的照片应用程序来实现的。您也可以检测滑动手势，并用它们来弹出当前视图控制器从堆栈，但这种行为是不是简单地切换导航栏的能见度。

### 直接修改导航栏对象

在一个导航界面，导航控制器拥有其`UINavigationBar`目的，并负责管理它。不允许改变导航栏对象或直接修改其边界，框架或alpha值。但是，也有少数[的属性](undefined)，你可以修改，其中包括：

- `barStyle` 属性
- `translucent` 属性
- `tintColor` 属性

图1-8显示了如何`barStyle`和`translucent`属性影响导航栏的外观。半透明样式，这是值得注意的是，如果基础视图控制器的主要观点是滚动视图，导航栏会自动调整内容插入值，以使内容从导航栏下的滚动出来。它不会使这种调整对于其他类型的意见。

**图1-8**   导航条的风格![](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/navbar_styles.jpg)

如果你想显示或隐藏整个导航栏，则应该同样使用`setNavigationBarHidden:animated:`导航控制器的方法，而不是直接修改导航栏。有关显示和隐藏导航栏的更多信息，请参阅[显示和隐藏的导航栏](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html#//apple_ref/doc/uid/TP40011313-CH2-SW15)。

#### 使用自定义按钮和视图作为导航项目

要自定义导航栏的外观，修改其对应controller的属性`UINavigationItem`对象。你可以从它的视图控制器`navigationItem` [属性](undefined)得到。

如果您选择不修改默认导航项目，导航项目提供了一组应该在很多情况下足以适用的默认对象。您所做的任何自定义设置优先于默认对象。

对于最上面的视图控制器，所显示的导航栏的`left item`，由以下规则确定：

- 如果您自定义栏按钮项目分配到`leftBarButtonItem`，该item被赋予优先级最高。
- 如果你不提供自定义栏按钮项，在导航堆栈上的低一个级别的视图控制器的`backBarButtonItem`属性，`navigation item`有效，在导航栏上显示该项目。
- 如果没有任何视图控制器的指定栏按钮项目时，默认的后退按钮被使用，它的标题设置为上一个视图的title属性的值，也就是说，视图控制器向下一个级别上导航堆栈。（如果最顶部视图控制器是根视图控制器，显示没有默认返回按钮）。

对于最上面的视图控制器，即显示在导航条的`center item`，由以下规则确定：

- 如果您分配一个自定义视图到`titleView`，导航栏显示视图。
  - 如果没有自定义标题视图设置，导航栏将显示包含视图控制器的标题自定义视图。从获得该视图的字符串`title`视图控制器的导航项目的property。如果该属性的值是`nil`，视图控制器本身的`title`属性被利用。

对于最上面的视图控制器，所显示的导航栏的`right item`，由以下规则确定：

- 如果新的顶级视图控制器具有自定义权限栏按钮项，将显示该项目。要指定自定义权限栏按钮项目，设置`rightBarButtonItem`的导航项目的属性。
- 如果没有指定自定义权限栏按钮项，导航栏显示在栏右边什么都没有。

要添加导航栏控件上面的自定义提示文字，则指定`prompt`的导航项目的属性。

图1-9显示了不同的导航栏的配置，包括一些使用自定义视图和提示。本图中的导航栏是从样本项目*定制UINavigationBar的*。

**图1-9**   导航栏自定义按钮![](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/navbar_custom_items.jpg)

清单1-2显示了将需要创建第三导航栏的NavBar应用程序代码图1-9，这是包含自定义视图右侧栏按钮项的导航栏。由于是在导航栏上合适的位置，就必须包装成一个自定义视图`UIBarButtonItem`其分配给前对象`rightBarButtonItem`属性。

**清单1-2**   创建自定义栏按钮项目

| `//查看3  - 以便自定义权栏按钮`                     |
| ---------------------------------------- |
| `UISegmentedControl * segmentedControl = [[UISegmentedControl页头] initWithItems：` |
| `                                      [NSArray的arrayWithObjects：` |
| `                                          [UIImage的imageNamed：@“up.png”]，` |
| `                                          [UIImage的imageNamed：@“down.png”]，` |
| `                                          零]];` |
| ` `                                      |
| `[segmentedControl addTarget：自我行动：@选择（segmentAction :) forControlEvents：UIControlEventValueChanged];` |
| `segmentedControl.frame = CGRectMake（0，0，90，kCustomButtonHeight）;` |
| `segmentedControl.segmentedControlStyle = UISegmentedControlStyleBar;` |
| `segmentedControl.momentary = YES;`      |
| ` `                                      |
| `defaultTintColor = segmentedControl.tintColor; //跟踪，如果这如果以后需要它。` |
| `*的UIBarButtonItem = segmentBarItem [的UIBarButtonItem页头] initWithCustomView：segmentedControl];` |
| `self.navigationItem.rightBarButtonItem = segmentBarItem;` |

配置您的视图控制器的导航项目编程是大多数应用程序最常用的方法。虽然你可以创建一个使用界面生成器栏按钮的项目，它往往是更简单以编程方式创建它们。您应该创建在项目`viewDidLoad`视图控制器的方法。

### 使用编辑和Done按钮

支持就地编辑视图可以在其导航栏一种特殊类型的按钮，允许用户显示和编辑模式之间来回切换。该`editButtonItem`方法`UIViewController`返回一个预配置的按钮按下时的编辑和完成按钮之间切换，并调用视图控制器的`setEditing:animated:`用适当的值的方法。要将此按钮添加到您的视图控制器的导航栏，可以使用类似于下面的代码：

如果您在您的导航栏这个按钮，你还必须重写你的视图控制器的`setEditing:animated:`方法，并用它来调整您的视角层次。有关实施此方法的详细信息，请参阅创建自定义内容视图控制器的*View Controller Programming Guide for iOS*.。

## 显示导航工具栏

在iOS 3.0及其以后，导航界面可以显示一个工具栏，并与当前可见的视图控制器提供的`item`填充它并计算。工具栏本身由导航控制器对象管理。在这个水平上支持一个工具栏，以创建屏幕之间的平滑过渡是必要的。当在导航的最顶层视图控制器栈的变化，导航控制器动画套不同的工具栏项目之间的变化。它还创建了要切换工具栏的可见性为特定视图控制器的情况下流畅的动画。

要为您的导航界面配置工具栏，你必须做到以下几点：

- 通过设置显示工具栏`toolbarHidden` [属性](undefined)导航控制器对象的`NO`。

- 指定数组`UIBarButtonItem`对象的`toolbarItems`每个内容视图控制器的属性，详见 [Specifying the Toolbar Items](https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html#//apple_ref/doc/uid/TP40011313-CH2-SW26)

  如果你不希望显示特定内容视图控制器一个工具栏，详见 [Showing and Hiding the Toolbar](https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html#//apple_ref/doc/uid/TP40011313-CH2-SW25)。

图1-10显示了您与您的内容视图控制器关联的对象如何反映在工具栏中的例子。项目显示在他们的阵列中提供相同的顺序在工具栏中。该系列包括：各类栏按钮的物品，包括固定和灵活的空间物品，系统按钮项目，或提供任何自定义按钮项目。在这个例子中，这五个项目是从邮件应用程序的所有按钮项目。

**图1-10**   在导航界面工具栏项目![](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/vc_toolbar_objects.jpg)

### 指定toolbar item

在配置item，记得要关联适当的action和target。在大多数情况下，target应该是视图控制器本身，因为它是负责提供工具栏项。图1-11示出了具有中心分段控制样品的工具栏。

要使用故事板指定工具栏项目：

1. 从库中拖动工具栏。

2. 两种灵活的空间栏按钮项添加到工具栏，从库中拖动。

3. 从库中添加分段控制，灵活的空间栏按钮之间，从库中拖动。

   使用Inspector配置分段控制。

**图1-11**   分段控件工具栏中中心![](https://developer.apple.com/library/prerelease/content/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Art/nav_toolbar.jpg)

清单1-3显示了以编程方式指定的工具栏项目所需的代码。你会实现该方法在您的视图控制器，并在[初始化](undefined)时调用。

**清单1-3**   

| `- (void)configureToolbarItems`          |
| ---------------------------------------- |
| `{`                                      |
| `   UIBarButtonItem *flexibleSpaceButtonItem = [[UIBarButtonItem alloc]` |
| `                        initWithBarButtonSystemItem:UIBarButtonSystemItemFlexibleSpace` |
| `                        target:nil action:nil];` |
| ` `                                      |
| `   // Create and configure the segmented control` |
| `   UISegmentedControl *sortToggle = [[UISegmentedControl alloc]` |
| `                        initWithItems:[NSArray arrayWithObjects:@"Ascending",` |
| `                                        @"Descending", nil]];` |
| `   sortToggle.segmentedControlStyle = UISegmentedControlStyleBar;` |
| `   sortToggle.selectedSegmentIndex = 0;` |
| `   [sortToggle addTarget:self action:@selector(toggleSorting:)` |
| `               forControlEvents:UIControlEventValueChanged];` |
| ` `                                      |
| `   // Create the bar button item for the segmented control` |
| `   UIBarButtonItem *sortToggleButtonItem = [[UIBarButtonItem alloc]` |
| `                                    initWithCustomView:sortToggle];` |
| ` `                                      |
| `   // Set our toolbar items`            |
| `   self.toolbarItems = [NSArray arrayWithObjects:` |
| `                         flexibleSpaceButtonItem,` |
| `                         sortToggleButtonItem,` |
| `                         flexibleSpaceButtonItem,` |
| `                         nil];`         |
| `}`                                      |

除了在初始化过程中设置`toolbar items`，视图控制器还可以改变其现有的一套`toolbar items`,通过动态的使用`setToolbarItems:animated:`的方法。此方法在更新`toolbar`命令，以反映用户操作的情况下非常有用。例如，你可以用它来实现一套分级的`toolbar item`，即点击栏上的按钮，然后显示一组相关的子按钮。

### 显示和隐藏工具栏

要隐藏特定视图控制器的工具栏，设置`hidesBottomBarWhenPushed`该视图控制器的属性`YES`。当导航控制器遇到带有此属性的一个视图控制器`YES`，每当视图控制器推到（或删除）导航堆栈，它产生适当的过渡动画。

如果你有时想隐藏工具栏，你可以随时调用`setToolbarHidden:animated:`导航控制器的方法。使用这种方法的常用方法是将它与调用相结合`setNavigationBarHidden:animated:`的方法来创建一个临时的全屏视图。例如，照片应用程序切换时，它会显示一张照片都酒吧的知名度和用户点击屏幕。

