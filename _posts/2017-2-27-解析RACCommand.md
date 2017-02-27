---
layout: post
title: RACCommand实战 - ReactiveCocoa（四）
tags: 
- ReactiveCocoa
---

### 简介

为什么要设计出来RACCommand？个人理解，是做了数据双向的处理，匹配数据加载和按钮点击场景。

### 创建 RACCommand

`RACCommand`的创建有两种形式：

```objective-c
- (id)initWithSignalBlock:(RACSignal * (^)(id input))signalBlock;  ①
```

```objective-c
- (id)initWithEnabled:(RACSignal *)enabledSignal signalBlock:(RACSignal * (^)(id input))signalBlock;  ②
```

```objective-c
 _button.rac_command = [[RACCommand alloc]initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
        return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
                   [subscriber sendNext:@1];
                   [subscriber sendCompleted]; 
                }];
            }];③

```



第一种就是直接通过传进一个用于构建`RACSignal`的`block`参数来初始化`RACCommand`，而`block`中的参数`input`为执行command时传入的数据，另外，创建出的signal可在里面完成一些数据操作，如网络请求，本地数据库读写等等，而第二种则另外还需要传进一个能传递`BOOL`事件的`RACSignal`，这个signal的作用相当于过滤，当传递的布尔事件为真值时，command能够执行，反之则不行。

注意: 伴随着command一起构建的signal，记得要在操作完成后发送完成消息以表示其执行完了：

```
[subscriber sendCompleted];

```

否则不能再执行此command。

第三种方法，UIButton中有属性`rac_command`用于绑定一个已经创建好的command（其使用在后面讲到），当你使用第二种方式创建command时，button的`enable`属性会随command的可执行性而改变，意思是当传递布尔事件的信号传递了真值事件，按钮才可使用。另外，当你按下按钮，command开始执行时，按钮的`enable`被自动设置成了`NO`，除非command执行完了，怎么判断command执行完成了呢？就是当其伴随的signal发送完成事件的时候（上面提及到）。

注意: 当button的`rac_command`已经绑定了某个command，而这个command又是以第二种方式初始化，那么你就不能动态改变button的`enable`，如:

```
RAC(self.button, enable) = someSignal;

```

这样子运行起来会报错。



### 订阅 RACCommand

订阅`RACCommand`我们可以使用其内部的属性`executionSignals`返回一个信号，然后对这个信号进行订阅。

```
[[aCommand executionSignals]
    subscribeNext:^(id x) {
    	NSLog(@"%@",x);
    }];

```

在订阅的block中，我们打印了传递事件`x`的描述，最后会发现`x`原来是一个`RACSignal`，原因是`RACCommand`中的`executionSignals`属性是一个包裹着信号的信号，其包裹着的信号就是我们当初在创建`RACCommand`时进行构建的信号，也就是信号的信号。多阶信号，可以使用函数`switchToLatest`进行降阶转换：

```
[[[aCommand executionSignals]switchToLatest]
    subscribeNext:^(id x) {
        //  Do something...
    }];

```



在对command进行错误处理的时候，我们不应该使用`subscribeError:`对command的`executionSignals`进行错误的订阅，因为`executionSignals`这个信号是不会发送error事件的。我们需要用到command的一个属性：`errors`，对错误进行订阅：

```
[aCommand.errors
	subscribeNext:^(NSError *x) {
		NSLog(@"ERROR! --> %@",x);
}];

```

### 执行 RACCommand

`RACCommand`的执行使用下面的这个函数：

```
- (RACSignal *)execute:(id)input;
```

input会作为创建command时其内部signal的构建block中的参数，用于传递数据。就是触发信号的didSubscribe闭包。

### 与 RACSubject的区别

用计算机网络中的术语，`RACSubject`更像“单工”，而`RACCommand`就类似于“半双工”。

- `RACSubject`只能单向发送事件，发送者将事件发送出去让接收者接收事件后进行处理，所以，`RACSubject`可代替代理，被监听者可利用subject发送事件，监听者接收事件然后进行相应的监听处理，不过，事件的传递方向是单向的。

- 对于`RACCommand`，用HTTP请求能够更形象地说明其原理，HTTP请求是由请求者向服务器发送一条网络请求，而服务器接收到请求然后经过相应处理后再向请求者返回处理过后的结果，数据流是双向的，`RACCommand`正是如此，让我想让某个部件进行某种会产生结果的操作时，利用`RACCommand`向此部件发送执行事件，部件接收到执行事件后进行相应操作处理并也通过`RACCommand`将操作结果回调到上层，使得事件得以双向流通。

  以上的解释是建立在`RACCommand`的事件产生与接收者为同一个对象的前提下的，而`RACCommand`也能将事件产生者和订阅者分离，让某个对象专门发送事件，通过`RACCommand`将事件传递到对数据进行操作处理的对象，最后，当数据处理完后再搭载着`RACCommand`把结果事件传出来，并被订阅者对象订阅。![418B8399-BE93-476E-9FD3-BC36EE81D53C](/Users/m4399/Library/Containers/com.tencent.eimmac/Data/Library/Application Support/QQ/Users/2853316748/QQ/Temp.db/418B8399-BE93-476E-9FD3-BC36EE81D53C.png)

参考：

[ReactiveCocoa之RACCommand使用(五)](http://blog.csdn.net/majiakun1/article/details/52937770)

[优雅的RACCommand](http://draveness.me/raccommand/)
