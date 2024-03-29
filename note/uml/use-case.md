---
title: UML用例图
description: 
author: Laeni
tags: UML, 软件工程
date: '2021-05-24'
updated: '2021-05-26'
---
## 用例图

用例图描述了人们希望如何使用一个系统，将相关用户、用户需要系统提供的服务以及系统需要用户提供的服务更清晰的显示出来，以便使系统用户更容易理解这些元素的用途，也便于开发人员最终实现这些元素。

用例图的结构主要分为三个部分：参与者、用例、关系（参与者与用例之间的关系）。

### 1 参与者

顾名思义，参与者代表系统外部与系统发生交互的人或事物，在UML中，通常用名字写在下面的人形图标表示。

需要注意，人指的是参与者与系统发生交互时的角色，不代指具体的人，也可以是任何的事；事物指的是某一个应用程序或者特殊进程，例如微信登录，通过跳转微信确认登录信息，微信对系统产生输入时，可以把微信作为参与者；而设定时间，强制退出账号时，时间这一特殊进程对系统产生输入，因此时间也可以作为参与者。

### 2 用例

#### 2.1 用例的说明

用例是对系统的用户需求（主要是功能需求）的描述，用例表达了系统的功能和所提供的服务。

用例是系统外部可见的一个功能单元，是某一个参与者在系统中做某件事从开始到结束的一系列活动的集合，以及结束时应该返回的可观测、有意义的结果，其中还包含可能的各种分支情况；具体用例在用例属性中说明。

#### 2.2 用例的特征

用例都是动宾结构；例如：登录账号用例是相互独立的用例由参与者启动有可观测的执行结果

### 3 关系

参与者与用例之间的关系主要包括关联、归纳（泛化）、包含、拓展和依赖。

#### 3.1 关联关系

关系说明：表示参与者与用例之间的关系

展示形式：![image-20210524195119265](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/1.png) 直线

举例说明：用户登录系统

​        ![image-20210525094606148](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/2.png)

#### 3.2 归纳（泛化/继承）关系

关系说明：表示参与者与参与者之间、用例与用例之间的关系

展示形式：![image-20210525111039393](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/3.png)用箭头表示，箭头从子参与者（子用例）指向父参与者（基础用例），一般父参与者（基础用例）相对子参与者（子用例)更为抽象

举例说明：VIP会员和普通用户，归纳为用户；账号登录与微信登录，也可归纳为登录系统。

![img](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/4.jpeg)

#### 3.3 包含关系

关系说明：表示用例与用例之间的关系

展示形式：![image-20210525113418782](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/5.png)用带有“include/包含”的箭头表示，箭头从基础用例指向包含用例

举例说明：用户在账号登录过程中，包括输入账号、输入密码、确认登录等操作

![img](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/6.jpeg)

#### 3.4 拓展关系

关系说明：表示用例与用例之间的关系；用于拓展用例对基础用例的增强；拓展用例是在特定条件出现时，才会被执行的用例

展示形式：![image-20210525122602296](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/11.png)用带有“extend/拓展”的箭头表示，由拓展用例指向基础用例

举例说明：用户在登录过程中忘记了密码

![img](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/12.jpeg)

图4 用例与用例之间的拓展关系

#### 3.5 依赖关系

关系说明：表示用例与用例之间的关系；一个用例在活动执行过程中，要依赖另一个用例的执行

展现形式：![image-20210525140623994](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/7.png)以一条直线相连

举例说明：用户要登录系统后，才能查看首页信息

补充说明：A用例依赖B用例，A用例或使用B用例执行后的返回结果，或使用B用例执行部分功能。依赖关系类似于包含关系，都是在用例执行过程中，调用其它用例来完成部分任务。

#### 3.6 注释

对于部分有特殊条件支撑的用例，也可以添加注释加以说明，例如VIP用户与普通用户登录系统后，可查看的菜单、数据甚至对系统的操作都是不一样的，此时可以在对应用例上加以注释，以强调此用例的特殊需求。

![img](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/8.jpeg)

#### 3.7 子系统

关系说明：用于强调某部分用例的强关联性，例如门户包含系统登录、首页信息展示等。

![img](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/9.jpeg)

#### 3.8 各关系的对比

为了对包含、拓展和归纳（泛化）关系更好的区分，以图7为例说明各种关系之间的差别：

##### 1）用例的使用条件

包含用例与归纳（泛化）的子用例，都没有限定的使用条件；例如用户登录系统时，直接选择输入账号密码登录系统，或者通过微信登录系统；而忘记密码是在用户账号登录时遗忘密码才会发生的用例，是有特定条件下才会发生的用例。

##### 2）直接、间接提供服务

归纳（泛化）的子用例与拓展用例为参与者直接提供服务，例如用户登录系统时，会直接选择账号登录或微信登录，而账号登录或微信登录直接为参与者提供登录服务；而包含关系的用例，为参与者提供间接服务，例如账号登录时，需要输入账号、输入密码等，这些用例直接鼓舞于账号登录这个用例，间接为参与者提供登录服务。

##### 3）其余说明

延伸用例与基础用例相互独立，两者之间不包含对方用例的内容。归纳（泛化）的子用例包含基础用例所有内容、基础用例与其他用例的关系以及基础用例与参与者之间的关系；例如账号登录是登录系统的子用例，但账号登录包含了登录系统的内容、登录系统与展示首页的关系以及登录系统与参与者的关系。

## 用例描述

完成了用例图，实际上工作只完成了一半，更重要的是对每个用例进行具体的说明；包括说明用例之间的关系、参与者身份角色以及用例从开始至结束过程中的条件及分支情况等；具体用例说明形式可参考下表：

![img](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/uml/use-case/10.jpeg)

用例的描述针对不同业务系统，描述的重点可能会存在差异，因此用例描述的重点在于清晰表达用例需求，不必拘泥于表达形式。

最后

不管用例图与表格画得多么酷炫，最终目的也是为了团队同事可以用最短的时间及精力完成对需求的理解。因此扎实的文档能力是产品的基础要求，希望这份总结能给到对用例说明无从下手的童鞋一点帮助。

如有错误，希望各位指正；共勉！

本文由 @庞庞 原创发布于人人都是产品经理。未经许可，禁止转载

题图来自Unsplash，基于CC0协议



## 参考/摘录文献

- [【知乎】用例图怎么画？](https://www.zhihu.com/question/418851220)
- [【百家号】详解 UML 用例图画法 & 用例说明方式](https://baijiahao.baidu.com/s?id=1661400666935924580&wfr=spider&for=pc)

