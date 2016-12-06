---
layout: post
title: About iOS App Architectu
tags: 
- APP
---

# 关于iOS应用架构

应用程序需要与iOS一起工作，以确保它们提供良好的用户体验。 除了将应用的界面和设计设计良好外，良好的用户体验还包含许多其他因素。 用户希望iOS应用程序快速响应，同时期望应用程序尽可能少使用功率。 应用程序需要支持所有最新的iOS设备，同时仍然应用程序是为当前设备量身定制的。 实现所有这些行为可能看起来令人望而生畏，但iOS提供了您实现它所需要的帮助。

![../Art/ios_pg_intro_2x.png](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/ios_pg_intro_2x.png)

本文档强调了如何让应用在iOS上正常运行的核心行为。 您不一定实现本文档中描述的每个功能，但是您应该在每个创建的项目考虑这些功能。

> **注：**
>
>  iOS应用程序的开发需要安装了iOS SDK的基于Intel的Macintosh计算机。 有关如何获取iOS SDK的信息，请转到 [iOS Dev Center](https://developer.apple.com/devcenter/ios/)。

## 大概

当你准备把你的创意变成一个应用程序，则需要了解系统和您的应用程序之间的关联。

### 应用需要支持主要功能

系统期望每个应用程序有一些特定的资源和配置数据，如应用程序图标和有关应用程序的功能的信息。Xcode为每个新项目提供一些信息，但你必须提供资源文件，你必须确保在应用提交之前项目中的信息正确。

**相关章节：**[Expected App Behaviors](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ExpectedAppBehaviors/ExpectedAppBehaviors.html#//apple_ref/doc/uid/TP40007072-CH3-SW2)

### 应用程序请遵循定义明确（Well-Defined）的执行路径

从用户启动应用程序到退出的期间，应用程序遵循明确定义的执行路径。在应用程序的生命周期中，它可以在前台和后台执行之间转换，它可以终止和重新启动，也可以暂时进入睡眠。每次它转换到一个新的状态，应用程序的期望改变。前台应用程序可以做任何事情，但后台应用程序必须尽可能少。您可以使用状态转换来相应地调整应用的行为。

**相关章节：** [The App Life Cycle](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/TheAppLifeCycle/TheAppLifeCycle.html#//apple_ref/doc/uid/TP40007072-CH2-SW1), [Strategies for Handling App State Transitions](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforHandlingAppStateTransitions/StrategiesforHandlingAppStateTransitions.html#//apple_ref/doc/uid/TP40007072-CH8-SW1)

### 应用必须在多任务环境中高效运行

电池寿命对用户很重要，性能，响应能力和良好的用户体验也是如此。最小化应用程序对电池的使用可确保用户可以一天运行应用程序，而无需为设备充电，但启动并准备好快速运行也很重要。 iOS多任务实现提供了良好的电池寿命，而不牺牲用户期望的响应和用户体验，但实施需要应用程序采用系统提供的行为。

**相关章节：**   [Background Execution](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW1), [Strategies for Handling App State Transitions](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforHandlingAppStateTransitions/StrategiesforHandlingAppStateTransitions.html#//apple_ref/doc/uid/TP40007072-CH8-SW1)

### 应用程序之间的通信遵循特定的途径

为了安全起见，iOS应用在沙盒中运行，与其他应用的互动有限制。当您想与系统上的其他应用程序通信时，有特定的方法可以执行。

**相关章节：**[Inter-App Communication](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Inter-AppCommunication/Inter-AppCommunication.html#//apple_ref/doc/uid/TP40007072-CH6-SW2)

### 性能调优的重要性

应用程序执行的每个任务都有相关的电源成本。耗完用户电池的应用程序会产生负面的用户体验，并且比电池几天内用完的应用更容易被删除。因此，请注意不同操作的成本，并利用系统提供的节能措施。

**相关章节：**[Performance Tips](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/PerformanceTips/PerformanceTips.html#//apple_ref/doc/uid/TP40007072-CH7-SW1)

## 如何使用本文档

本文档不是创建iOS应用程序的初学者指南。它是为开发人员准备优化他们的应用程序，然后将其放入App Store。使用本文档作为指导，了解您的应用如何与系统进行交互，以及必须做什么以使这些交互顺利进行。

## 先决条件

本文档提供有关iOS应用程序架构的详细信息，并向您展示如何实施许多应用程序级功能。本书假设您已经安装了iOS SDK，配置了您的开发环境，并且了解在Xcode中创建和实现应用程序的基础知识。

如果您是iOS开发新手，请参阅*Start Developing iOS Apps (Swift)*。该文档提供了对开发过程的分步介绍，以帮助您快速上手。它还包括一个实践教程，引导您完成从开始到完成的应用程序创建过程，向您展示如何创建一个简单的应用程序，并使其快速运行。

## 也可以看看

如果您正在学习iOS，请阅读 *iOS Technology Overview* ，了解您可以合并到您的iOS应用程序中的技术和功能。