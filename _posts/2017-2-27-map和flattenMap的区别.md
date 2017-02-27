---
layout: post
title: flattenMap和map
tags: 
- ReactiveCocoa

---

### RWTableViewCell的flatternMap方法

 1、执行flattenMap，就会执行bind方法，并由bind方法返回新signal
2、此时传入bind的闭包是对flattenMap闭包的转换（只对参数做转换，处理逻辑一致），此时，bind闭包和flattenMap闭包被保存，但不触发执行
3、步骤1返回的新信号被订阅，block6被执行
4、bind的闭包执行，得到block3，block3是RACStreamBindBlock类型。block3 = bindingBlock
6、旧信号保存在signals数组

7、销毁者容器创建

8、RACSignal内的self代表是旧信号，从[signal bind:]，signal为旧信号

9、订阅旧信号，[self subscribeNext:block10]，block10 = 旧信号的nextBlock。

10、订阅旧信号就会触发旧信号的didSubscribe闭包，旧信号的订阅者就会发送事件，执行旧信号的nextBlock。

11、block10内，会执行会bindingBlock，相当于执行block3，block3会转换旧信号的事件值，并返回新的信号

12、如果signal存在，然后发送新信号的事件；如果不存在就发送结束事件![QQ20170122-1](/Users/m4399/Pictures/com.tencent.ScreenCaptureEIM/QQ20170122-1.png)

### 准备工作不多说：

创建旧信号，保存旧信号didSubscirbe闭包

执行flattenMap，保存flattenMap方法的闭包，得到新信号（新信号是通过bind方法得到的）

订阅新信号

### 1、订阅新信号

就会调用bind方法的内部定义

通过一系列转化，bindingBlock = flattenMap方法的闭包

订阅新信号，就会订阅旧信号，触发旧信号的sendNext发送事件

### 2、此时就会更新block的value值



### 3、调用bindingBlock（flattenMap方法的闭包）

也就是调用block3，调用了block3，也就是调用了block（value），也是执行了flattenMap方法的闭包，返回了信号

### 4、发送新信号的事件

bind方法会让新信号发送事件，新信号的nextBlock被调用。

总结：

1、block(value)，block指的是flattenMap方法的闭包，block(value)指的是闭包返回的信号

2、接收的事件就是flattenMap方法的闭包返回的信号，发出的事件

3、新信号就是block(value)返回的东西

 ![456571B2-92C3-4C6D-88BC-0B4B2CD8235C](/Users/m4399/Library/Containers/com.tencent.eimmac/Data/Library/Application Support/QQ/Users/2853316748/QQ/Temp.db/456571B2-92C3-4C6D-88BC-0B4B2CD8235C.png)

### map

map是升阶。相当于是指定flattenMap闭包，特殊的flattenMap闭包

1、flattenMap闭包是

	^(id value) {	
		return [class return:block(value)];//在这里对信号升阶了
	}]
2、新信号是：block(value)的东西，然后再封装一层信号