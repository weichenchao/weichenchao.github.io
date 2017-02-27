---
layout: post
title: ReactiveCocoa常用操作 - ReactiveCocoa（二）
tags: 
- ReactiveCocoa
---

详见[ReactiveCocoa 中 RACSignal 所有变换操作底层实现分析(下)](http://www.jianshu.com/p/d507e534dda0)

### 转换类

1、map：对return返回的东西，升阶

2、flatternMap：等于return返回的东西

3、mapReplace：

```objective-c
- (instancetype)mapReplace:(id)object {
    return [[self map:^(id _) {
        return object;
    }]];
}

```
​

### 组合

1、concat：A和B组合成一个信号，A先逐个发完，再发B

2、then：A和B组合成一个信号，过滤掉A的全部信号，等信号A发完，再发B

	先过滤掉之前的信号发出的值。2.使用concat连接then返回的信号
3、merge：把多个信号合并为一个信号，任何一个信号有新值的时候就会调用

4、zipWith：把两个信号压缩成一个信号，只有当两个信号同时发出信号内容时，并且把两个信号的内容合并成一个元组，才会触发压缩流的next事件

5、combineLatest：将多个信号合并起来，合并成元祖，并且拿到各个信号的最新的值,必须每个合并的signal至少都有过一次sendNext，才会触发合并的信号。

	和zipWith一样，combineLatest可以为多个信号
6、reduce聚合：用于信号发出的内容是元组，把信号发出元组的值聚合成一个值
	常见的用法，（先组合在聚合）
	combineLatest:(id<NSFastEnumeration>)signals reduce:(id (^)())reduceBlock
	reduce中的block简介:
	reduceblcok中的参数，有多少信号组合，reduceblcok就有多少参数，每个参数就是之前信号发出的内容
	reduceblcok的返回值：聚合信号之后的内容



### 过滤

1、filter:过滤信号，使用它可以获取满足条件的信号

2、ignore:忽略完某些值的信号

3、distinctUntilChanged:当上一次的值和当前的值有明显的变化就会发出信号，否则会被忽略掉

4、take:从开始一共取N次的信号takeLast:取最后N次的信号,前提条件，订阅者必须调用完成，因为只有完成，就知道总共有多少信号

5、takeUntil:(RACSignal *):获取信号直到某个信号执行完成

6、skip:(NSUInteger):跳过几个信号,不接受

7、switchToLatest:用于signalOfSignals（信号的信号），有时候信号也会发出信号，会在signalOfSignals中，获取signalOfSignals发送的最新信号。

8、catch：捕捉错误，将错误事件转换成catchBlock指定的事件，错误事件进行特殊处理，其他（next，完成）事件

9、catchTo：捕捉错误，将错误事件转换成特定的signal
​
​

### 顺序

1、doNext: 执行Next之前，会先执行这个Block

2、doCompleted: 执行sendCompleted之前，会先执行这个Block

### 时间

1、timeout：超时，可以让一个信号在一定的时间后，自动报错

2、interval 定时：每隔一段时间发出信号

3、delay 延迟发送next

4、throttle节流:当某个信号发送比较频繁时，可以使用节流，在某一段时间不发送信号内容，过了一段时间获取信号的最新内容发出

### 懒激活

1、defer：第一次订阅无效，第二次才真正开始

2、collect：收集源信号的所有sendNext事件，直到源信号发出sendComplete事件，才将收集好的sendNext事件发出去

### 内存泄漏

让信号释放的两种操作，前提不要循环引用

1、sendError，或sendCompleted（自释放）

2、takeUntil（被动释放）

### 用RAC开发的思路

1、把数据来源变成信号（来源可能是多个）

2、套入各种管道模型：collect +zip模型，多个map模型

3、输出

###参考

[用 ReactiveCocoa 事半功倍的写代码（一）](http://fengjian0106.github.io/2016/04/17/The-Power-Of-Composition-In-FRP-Part-1/)

[常用操作动态效果图](http://rxmarbles.com/#last)


