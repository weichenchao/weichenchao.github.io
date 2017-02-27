---
layout: post
title: ReactiveCocoa源码分析 - ReactiveCocoa（一）
tags: 
- ReactiveCocoa

---

## 1、信号

#### [RACSignal createSignal];

1. [RACDynamicSignal createSignal];

   这个方法作用：仅做了初始化功能，并保存闭包

   * 创建RACDynamicSignal
   * 保存didSubscribe闭包，但不触发。
   * 格式化命名


#### [RACSignal subscribeNext];

1. [RACSubscriber subscribeNext];

   这个方法作用：保存三个闭包，创建和初始化RACSubscriber

   * 保存但不触发nextBlock，errorBlock，completedBlock三个闭包
   * 创建和初始化RACSubscriber

2. RACDynamicSignal调用subscribe方法，并传入刚创建的订阅者。

   **subscribe这个方法，是订阅的核心**：管理信号，订阅者，销毁者之间的关系

   * a、创建RACCompoundDisposable对象，创建销毁者容器

   * b、**核心的核心**创建RACPassthroughSubscriber对象。RACPassthroughSubscriber分别保存对RACSignal，RACSubscriber，RACCompoundDisposable的引用，链接信号，订阅者，销毁者之间的关系

   * c、管理销毁者容器
     * （一）: 获取调度器单例，并调度
     * （二）: currentScheduler如果存在，执行（三）: ；currentScheduler如果不存在，执行（四）: 
     * （三）: didSubscribe闭包被触发执行，innerDisposable被闭包创建返回，schedulingDisposable=nil
     * （四）:执行[self.backgroundScheduler schedule:block]，schedulingDisposable有上述方法返回; didSubscribe闭包不会触发且等待，innerDisposable=nil。

   * d、销毁者容器加入innerDisposable或schedulingDisposable，根据currentScheduler；

     ​


#### [subscriber sendNext:@1];

1、看上述c（三）执行，didSubscribe闭包，被传入订阅者，并被触发。[subscriber sendNext:@1]方法在闭包里，自然开始执行。同理：[subscriber sendCompleted];和[subscriber sendError];

2、didSubscribe闭包被传入订阅者是RACPassthroughSubscriber类型，负责转发给真正的订阅者RACSubscriber去执行sendNext闭包。

3、同理：[subscriber sendCompleted];和[subscriber sendError];

不同的是：真正的对应的sendCompleted，sendError闭包被执行，还会对销毁者进行销毁

 ![D7F7EFD4-A4FA-40D9-BE11-6FBE2111159A](/Users/m4399/Library/Containers/com.tencent.eimmac/Data/Library/Application Support/QQ/Users/2853316748/QQ/Temp.db/D7F7EFD4-A4FA-40D9-BE11-6FBE2111159A.png)

## 2、bind

顺序是：

创建信号，保存didSubscribe；

订阅信号，保存nextBlock；

订阅过程，触发didSubscribe；

didSubscribe执行过程中，闭包包含了sendNext，触发nextBlock。



1、创建原始信号，保存原始信号（旧信号）的didSubscribe，block 1 = 旧信号的didSubscribe

2、原始信号调用bind方法，return [RACSignal createSignal:innerBlock6]，innerBlock6 = 新信号的didSubscribe，返回绑定信号（新信号）。

3、[bindSignal subscribeNext:block 4]，订阅绑定信号（新信号），block 4 = 新信号的nextBlock

4、订阅新信号，就会执行新信号的didSubscribe =innerBlock6；

5、bind方法第一句：bindingBlock，执行block2，得到block3，bindingBlock = block3

6、旧信号保存在signals数组

7、销毁者容器创建

8、RACSignal内的self代表是旧信号，从[signal bind:]，signal为旧信号

9、订阅旧信号，[self subscribeNext:innerBlock10]，innerBlock10 = 旧信号的nextBlock。

10、订阅旧信号就会触发旧信号的didSubscribe闭包，旧信号的订阅者就会发送事件，执行旧信号的nextBlock。

11、innerBlock10内，会执行会bindingBlock，相当于执行block3，block3会转换旧信号的事件值，

12、如果signal存在，然后发送新信号的事件；如果不存在就发送结束事件

13、执行新信号的nextBlock = block 4



## 3、concat：按顺序合并

1. 调用新的合并信号的didSubscribe。
2. 由于是第一个信号调用的concat方法，所以block中的self是前一个信号signal。合并信号的didSubscribe会先订阅signal。
3. 由于订阅了第一个signal，也就是signalA，于是开始执行signal的didSubscribe，sendNext，sendError。
4. 当前一个信号signal发送sendCompleted之后，就会开始订阅后一个信号signal，也就是signalB。
5. 如何订阅？[signal subscribe:subscriber]；[signalB subscribe:合并信号的订阅者]，调用signalB的didSubscribe。
6. 由于订阅了后一个信号signalB，于是后一个信号signalB开始发送sendNext，sendError，sendCompleted。



##4、zipwith：一对一合并

zipWith里面有两个数组，分别会存储两个信号的值。

1. 一旦订阅了zipWith之后的信号，就开始执行didSubscribe闭包。
2. 在闭包中会先订阅第一个信号。这里假设第一个信号比第二个信号先发出一个值。第一个信号发出来的每一个值都会被加入到第一个数组中保存起来，然后调用sendNext( )闭包。在sendNext( )闭包中，会先判断两个数组里面是否都为空，如果有一个数组里面是空的，就return。由于第二个信号还没有发送值，即第二个信号的数组里面是空的，所以这里第一个值发送不出来。于是第一个信号被订阅之后，发送的值存储到了第一个数组里面了，没有发出去。
3. 第二个信号的值紧接着发出来了，第二个信号每发送一次值，也会存储到第二个数组中，但是这个时候再调用sendNext( )闭包的时候，不会再return了，因为两个数组里面都有值了，两个数组的第0号位置都有一个值了。有值以后就打包成元组RACTuple发送出去。并清空两个数组0号位置存储的值。
4. 以后两个信号每次发送一个，就先存储在数组中，只要有“配对”的另一个信号，就一起打包成元组RACTuple发送出去。从图中也可以看出，zipWith之后的新信号，每个信号的发送时刻是等于两个信号最晚发出信号的时刻。
5. 新信号的完成时间，是当两者任意一个信号完成并且数组里面为空，就算完成了。所以最后第一个信号发送的5的那个值就被丢弃了。

第一个信号依次发送的1，2，3，4的值和第二个信号依次发送的A，B，C，D的值，一一的合在了一起，就像拉链把他们拉在一起。由于5没法配对，所以拉链也拉不上了。

### 5、最简单的信号是单元信号，有4种：

```
// return信号：被订阅后，立马产生一个值事件，然后产生一个完成事件

RACSignal *signal1 = [RACSignal return:someObject];

// error信号：被订阅后，立马产生一个错误事件

RACSignal *signal2 = [RACSignal error:someError];

// empty信号：被订阅后，立马产生一个完成事件

RACSignal *signal3 = [RACSignal empty];

// never信号：永远不产生事件

RACSignal *signal4 = [RACSignal never];

```

### 6、冷热信号的区别

[细说ReactiveCocoa的冷信号与热信号（一)](http://tech.meituan.com/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-1.html)

[细说ReactiveCocoa的冷信号与热信号（二）：为什么要区分冷热信号](http://tech.meituan.com/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-2.html)

[细说ReactiveCocoa的冷信号与热信号（三）：怎么处理冷信号与热信号](http://tech.meituan.com/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-3.html)



| 热信号           | RACSubject                             | RACReplaySubject              | 冷信号                         |
| :------------ | -------------------------------------- | ----------------------------- | --------------------------- |
| connect       | 同subscribe[原始信号被subscribe]             |                               |                             |
| subscribe     | 将订阅者加入集合                               | 将订阅者加入集合；遍历缓存事件，当前单个订阅者多次发送事件 | 执行didSubscribe闭包，触发sendNext |
| subscribeNext | 同subscribe[目标信号被subscribe]             |                               |                             |
| sendNext      | 遍历每个订阅者，每个订阅者对当前的事件值发送事件，触发nextBlock闭包 | 相同（多了一步：处理缓存事件值，把事件指加入缓存）     | 无                           |
|               |                                        |                               |                             |
