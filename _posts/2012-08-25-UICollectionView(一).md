## iOS6 进阶UICollectionView（一）　　
　　在前面的章节中，你学到了很多关于UICollectionView，但这里你将会遇到更多。UICollectionView支持不同类型的布局，但到目前为止，你只能看到了一种类型的布局 - Flow Layout。 
　　Flow Layout流布局是iOS 6SDK自带的collection view布局。但是，您可以编写自己的 - 并通过这样做，你可以轻松地创建令人惊叹的界面和令人羡慕的影响。 
　　
　　本章将引导您完成创建三个这样的自定义布局。首先，我会告诉你如何模仿照片应用程序的行为，让用户增删照片。
　　
![](http://upload-images.jianshu.io/upload_images/1670295-964bdaf80ce790ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

　　然后您将学习如何创建一个堆叠栏布局像Pi​​nterest的使用。这种布局在普及获得了与开发商，因为这样的作品真的很好，用于显示可变大小图像的天然单一视图：

![](http://upload-images.jianshu.io/upload_images/1670295-19ca997b74ab8101.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
         
　　最后，你将实现苹果公司的音乐应用程序，你所知道和喜爱的简单而优雅，“Cover Flow”的布局： 

![](http://upload-images.jianshu.io/upload_images/1670295-fdc188826cef7cb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
　　最重要的是，让你的手指编码涂满这些布局，您将非常熟悉UICollectionView API。这将使你装上火药，准备创建自己的自定义布局，以任何方式你可能想象到显示数据 ， 用有史以来最少最简洁的代码！
### 深入collection  view
在您开始代码，关于UICollectionView怎么样建立和维护布局的认识是很重要的。因此，让我们来看看所有关于UICollectionView的类，它们如何协同工作和布局的生命周期。

#### 了解层级

![](http://upload-images.jianshu.io/upload_images/1670295-03a553978b652678.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如你所想，中心是UICollectionView。当你创建一个UICollectionView，你需要给它三类信息，a datasource，a delegate，a layout。

* The data source:实现UICollectionViewDataSource协议的类。它的主要任务是提供显示的数据，并为每个元素（cells, supplementaryviews, and decoration views）返回UICollectionResuableViews。在下一章，将设置此项。
* The delegate：实现UICollectionViewDelegate协议的类。当事件发生，它被调用，如用户选择cell的事件。
* The layout：UICollectionViewLayout的子类。这是本章的意义所在！它的任务是告诉collection view在何处放置所有不同的元素。
　　UIKit提供的唯一布局称为UICollectionViewFlowLayout。有一个小问题小主意，当您使用此布局 - 当你使用它，collection view的代理必须是一个UICollectionViewDelegateFlowLayout。这是UICollectionViewDelegate的子协议，只是增加了一些更多的方法来对一个flow layout所需的基础协议。

### 布局属性（Layout attributes）
　　UICollectionView处理所有复杂cell管理，并与UICollectionViewDataSource接口，找出哪些元素来显示。您可以实现数据源告诉collectionView section的个数和cell的内容，并配置最终会被显示的cell。
UICollectionView结合UICollectionViewLayout子接口，弄清楚如何在屏幕上布局单元格。 UICollectionViewLayout是一个抽象基类，所以需重写来创建自定义布局。
>注意：抽象类是，你必须继承并不能直接实例化一个类。在Objective-C，这并不完全正确，你不能实例化一个抽象类的实例，因为没有机制来执行本。但是，如果文档指定它作为一个抽象类，那么你应该继承它。
　　当collection view想知道在哪里放置ells, supplementary views,and decoration views，它会访问layout的布局属性。封装了这些数据的类被称为UICollectionViewLayoutAttributes，它描述了以下布局的详细信息：

•bounds：collection view的坐标系内,视图的位置。
•center：坐标系统内的视图的中心。这可以用尺寸参数代替的帧参数来设置沿。
•sizw：视图的大小。这可以与中心参数代替的帧参数来设置沿。
•Transform3D：如果要旋转，缩放或改造它被应用到视图的CATransform3D对象。
•alpha：视图的透明度。
•zIndex：相对于其他视图的视图的顺序。这可以用来定义视图具体地说是否应在或低于其他视图顶部。具有较高的z索引视图将在具有较低z索引视图的顶部显示。
•hidden：是否该视图隐藏。

### layout的生命周期
　　collection view调用它的布局了很多不同的方法，来收集所需绘制所有元素在正确的屏幕地方的信息。了解laoyout的周期有点复杂，所以我画了一个流程图来帮助你。
　　在下图中，蓝框表示由collection view调用的布局方法，橙框代表collection view的状态和绿框是让用户调用的方法。快速浏览一下图，然后阅读详细的解释。

![生命周期](http://upload-images.jianshu.io/upload_images/1670295-508c0e2c650a1490.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
　　周期开始时，collection view不知道的布局任何信息。它首先调用prepareLayout - 这个方法是布局的机会，设置任何它需要的内部数据结构。
　　至此，collection view确定它需要的数据的一切，但仍然对布局layout一无所知。这意味着，你可以愉快地调用numberOfSections或numberOfItemsInSection这两个方法，这两个实在prepareLayout内部调用的。
　　然后collection view通过调用collectionViewContentSize获取其内容的大小。这是在布局layout必须告诉collection view可滚动区域究竟需要多大，使用户能够浏览到所有项目。
　　在此之后，collection view调用layoutAttributesForElementsInRect：，用滚动区域的初始可见部分的矩形作为参数，调用上述方法。布局必须返回矩形框内所有的元素的布局属性。
　　现在collection view了解足够的信息来滚动，并要求其访问每个元素的代理，并适当地布置它们在屏幕上。当用户滚动屏幕和新的cell映入眼帘，collection view继续呼吁layoutAttributesForElementsInRect：确定布局的新的可见元素。
　　而在该状态下，没有其他方法会被调用，直到三种情况之一发生：
1.invalidateLayout。在这一点上，布局将被再次从头开始访问所有的布局信息。布局失效，异味着如果某些改变，布局属性layout attributes也会随之改变。例如，如果布局属性是依赖于某个变量，该变量变化，那么你需要在此变量的setter方法内，失效布局，调用invalidateLayout。
2.insertItemsAtIndexPaths或者deleteItemsAtIndexPaths等被调用。如果用户添加或删除cell，这将导致collection view调用prepareForCollectionViewUpdates布局方法来通知，更新即将发生。如果是插入item，它会获取新的cell布局属性​​。最后，调用finalizeCollectionViewUpdates。该collection view然后返回到正常状态(橙框 Normal State）。
3.collection view的bounds改变，如切换横竖屏，重排屏幕布局。这将导致collection view调用prepareForAnimatedBoundsChange。然后，将更改后的bounds被获取，finalizeAnimatedBoundsChange被调用。
　　通常情况下，你不必太担心这个生命周期的复杂性。知道它存在，当你想知道为什么方法刚刚被调用，或者想对某些事件的顺序，就参考一下上述图表。
　　这一点你会在本章中得到了大量的实践，并通过本章的最后，更为直观！
### 自定义布局要重写的方法（Methods to override in a custom layout）
　　你可以在自定义布局子类中实现许多方法。然而，大部分时间，仅少数​​的这些方法需要被实现。这里有三个最重要的：
•collectionViewContentSize - 此方法返回collection view的内容大小，这意味着collection view所有内容的大小，不仅仅是可见区域。例如，一个简单的表，宽度等于视图的宽度，高度等于所以cell的高度和。
•layoutAttributesForItemAtIndexPath：（NSIndexPath *） - 此方法接受一个索引路径，并返回该cell的布局属性。
•layoutAttributesForElementsInRect：（CGRect） - 这个方法需要一个矩形参数，并返回布局的属性数组对应于所有那些在该矩形可见的元素（cells, supplementary views, and decoration views）。
　　如果从最后一章记得，苹果已经提供了一个名为UICollectionViewFlowLayout默认布局。它只是简单规定了从左到右，从top到bottom。如果你愿意，你也可以继承的flow layout。如果您的自定义布局和系统flow layout非常相似，只是对每个单元的属性需要一些调整。
　　在本章中，你将继承这两个抽象基类和flow layout创建一些自定义布局。所以，让我们开始flow layout派对！

![](http://upload-images.jianshu.io/upload_images/1670295-b537bb00c5e879a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)