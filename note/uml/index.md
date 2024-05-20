---
title: UML概述
description: 
author: Laeni
tags: UML, 软件工程
date: '2021-05-24'
updated: '2021-05-24'
hide: true
---

UML(Unified Modeling Language，统一建模语言)。UML支持面向对象的技术，能够准确、方便地表达面向对像的概念，体现面向对象的分析和设计风格。在进行项目的时候通过使用 UML 的面向对象图的方式能更明确、清晰地表达项目中的架设思想、项目结构、执行顺序等一些逻辑思维。

## UML 特点

  面向对象
  可视化，表达能力强
  独立于过程
  独立于程序设计
  容易掌握使用

## UML模型(共三部分)

  事物(Things): UML模型中最基本的构成元素，是具有代表性的成分的抽象
  关系(Relationships): 关系把事物紧密联系在一起
  图(Diagrams ): 图是事物和关系的可视化表示

## UML种类

### 用例图（Use Case Diagrame）

描述参与者之间的关系。

用例图主要用于**需求分析阶段**，是面向系统分析人员，需求人员甚至是用户的。

用例图描述了人们希望如何使用一个系统，将相关用户、用户需要系统提供的服务以及系统需要用户提供的服务更清晰的显示出来，以便使系统用户更容易理解这些元素的用途，也便于开发人员最终实现这些元素。
用例图是外部用户（被称为参与者）所能观察到的系统功能的模型图,是系统的蓝图。

用例图呈现了参与者、用例以及它们之间的关系，主要用于对系统、子系统或类的功能行为进行建模。
- Use Case(用例)

  解释1: 是对系统的用户需求（主要是功能需求）的描述，用例表达了系统的功能和所提供的服务，描述了活动者与系统交互中的对话。

- Actor(参与者) - 参与者是系统外部的一个实体，它以某种方式参与了用例的执行过程，在UML中，通常用名字写在下面的人形图标表示。
  值得注意的是：参与者不一定是人，也可以是任何的事。
  
- 二者之间的关系：包括1关联关系，2泛化关系，3包含关系，4扩展关系

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\1.jpg)

### 流程图

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\2.jpg)

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\3.jpg)



![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\4.jpg)

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\5.jpg)

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\1.jpg)

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\6.jpg)

### 对象图

描述一组对象之间的关系。

是类图的实例，几乎使用与类图完全相同的标识。一个对象图是类图的一个实例。由于对象存在生命周期，因此对象图只能在系统某一时间段存在。

### 时序图/顺序图

一个交互，强调消息的时间顺序。

显示对象之间的动态合作关系，它强调对象之间消息发送的顺序，同时显示对象之间的交互。

对象图和时序图选择一个，本质上是一回事。这两个图实际上提体现了业务的设计。在架构设计角度上非常重要。

![image-20210602153259062](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\7.png)

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\8.jpg)

### 协作图

一个交互，强调消息发送和接受对象的结构组织。

描述对象间的协作关系，协作图跟时序图相似，显示对象间的动态合作关系。除显示信息交换外，协作图还显示对象以及它们之间的关系。如果强调时间和顺序，则使用时序图；如果强调上下级关系，则选择协作图。

![image-20210602153309989](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\9.png)

![image-20210602153334142](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\10.png)

![image-20210602153351023](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\11.png)

---

### 类图

描述接口以及协作之间的关系。

描述系统中类的静态结构，不仅定义系统中的类，表示类之间的联系，如关联、依赖、聚合等，也包括类的属性和操作，类图描述的是一种静态关系，在系统的整个生命周期都是有效的。

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\12.jpg)

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\13.jpg)

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\14.jpg)

### 包图

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\15.jpg)

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\16.jpg)

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\17.jpg)

### 组织结构图

### 泳道图
### E-R图

### 构件图

描述一组构件之间的关系。

![image-20210602153435616](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\18.png)

![image-20210602153504190](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\19.png)

### 部署图

描述一组接点之间的关系。

定义系统中软硬件的物理体系结构。

![image-20210602153547311](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\20.png)

### 活动图

一个状态机，强调从活动到活动的流动。

描述满足用例要求所要进行的活动以及活动间的约束关系，有利于识别并进行活动。

![image-20210602153004270](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\21.png)

![image-20210602153048539](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\22.png)

### 复合状态图
### 组件图

描述代码部件的物理结构及各部件之间的依赖关系，组件图有助于分析和理解部件之间的相互影响程度。

### 状态图

一个状态机，强调对象按事件排序的行为。

描述类的对象所有可能的状态以及事件发生时状态的转移条件，状态图是对类图的补充。

![image-20210602153127386](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\23.png)

![image-20210602153149540](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\24.png)

## UML分类

### 一、UML从整体上分类

UML九种图，具体的可以分为五类。

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\25.png)

### 二、UML从静态和动态的角度分类

从静态和动态的角度，主要可以分为静态模型和动态模型。

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\26.png)

### 三、从建模的角度分类。

从建模的角度主要可以分为三类：静态建模，动态建模，物理架构建模。

![img](F:\Objects\cn.laeni\blog-content\note\uml\index.assets\27.png)

## UML图常用使用时机

| 阶段           | UML图                  |
| -------------- | ---------------------- |
| 需求分析       | 用例图、类图、包图     |
| 系统分析与设计 | 类图、包图             |
| 网络架构设计   | 组件图、部署图         |
| -              | 时序图、活动图、状态图 |

---

## 参考/摘录文献

- [【百度百科】统一建模语言](https://baike.baidu.com/item/%E7%BB%9F%E4%B8%80%E5%BB%BA%E6%A8%A1%E8%AF%AD%E8%A8%80/3160571)
- [【CSDN】UML九种图的分类](https://blog.csdn.net/nangeali/article/details/48953587)

