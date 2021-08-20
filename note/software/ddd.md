---
title: DDD入门
tags: 'DDD, 领域驱动设计, 软件工程'
author: Laeni
date: '2021-08-18'
updated: '2021-08-20'
---

DDD似乎一直都比较神秘，不同的人对它的理解不同，导致实际应用也差别很大。以下文章几乎全部摘录自[博客园](https://www.cnblogs.com/ahau10)，手动搬一遍（略有删改）除了作为笔记查阅之外主要是为了加深印象。原文链接在文末可找到。

# DDD入门之理解面向对象(一）

**面向对象编程的误解**
大多数程序员可能都把面向对象里的“对象”理解错了，理解成了语法层面的对象。所以代码才会出现所谓的贫血模型。

**面向对象编程里的“对象”是什么？**
封装了数据和行为的东西

**我们的代码有体现吗？**
没有。语法层面对象的3大特点（封装、继承、多态），我们的代码都有体现。

但是对象里要么是只有数据，没有行为（POJO）。要么是只有行为，没有数据（Serivce）。即所谓的贫血模型。

定义POJO，封装了属性和getter&setter，看似具有封装性，其实仅仅是一个数据容器而已。因为没有把数据和行为封装起来，行为都放在Service里了。

这就是我说的语法层面的面向对象。正式的叫法是面向数据编程。

真正的面向对象，面向的是生活中的真实对象，用代码的方式模拟真实的对象，即我们说的模型：

比如我们要模拟猫这个现实中的对象。我们了解到猫有品种、颜色、体重等属性；猫有吃鱼、捕鼠的行为。

那么首先对它进行建模。这个模型要能如实的反应猫的特点，并且这个模型是稳定的，一旦定了不能随意修改，比如随便的将“捕鼠”这个行为拿掉。

注意这里说的是模型，该模型里有数据，有行为。真正用代码实现模型的是不一定只有一个类。可能有很多个类：聚合根，实体，值对象，域服务等。这些类合起来称作模型！

注意这个点，这些类合起来称作模型！虽然有些类可能只有行为，没有数据（比如域服务），但这不能称作贫血。因为贫血指的是模型，而不是单个类。

网上有很多关于贫血的讨论，有些说法已经跑偏了，他们认为只要某个类没有同时具备数据和行为就是贫血的。或者实体里没有实现持久化就是贫血的。

这种看法是短视的。站在模型的角度从上往下看，实体，值对象，域服务，（实现持久化的）在一个模块里紧密联系，协同合作，就相当于封装了数据和行为，这就不贫血了。

理解了上述的“面向数据”和“贫血”这2个概念，才算是领悟到了“面向对象”的真谛。

# DDD入门之解决了什么问题（二）

## DDD是个啥？它解决了什么问题？

第一个问题不好回答，先回答第二个。第二个问题讲清楚了，第一个问题的答案也就呼之欲出了。即DDD是解决第二个问题的一种手段/方法。这些都是我个人的理解,网上看过很多文章，他们在讲DDD的时候都会先声明这是他们自己的理解。DDD确实有点“玄学”，不过大致还是有迹可循的，很多地方大家还是能达成共识的。

比如DDD试图解决什么问题？我理解的有两个：

1. 业务人员跟技术人员沟通的问题。
   开发小伙伴在开需求梳理会的时候经常说一些技术名词，比如我以前就经常说xx表xx字段之类的。领域专家们（指精通业务的人，比如测试同学就是领域专家）听不懂也不关心这些，他们经常说领域内的名词，就是他们擅长的领域里的“行话”。这其实挺尴尬的，大家言语不统一，沟通成本太高， 更恐怖的是，技术人员可能会把某个概念理解偏了，结果费了九牛二虎之力写出来的代码，验收的时候才发现，代码实现的效果压根不是人家想要的。 所以DDD要求大家（领域专家和技术人员）都使用同一套术语，别再说xx表xx字段（那些是技术实现），也不要把定好的术语口头上改成自己理解术的语。 统一术语，就是每个人都说这个术语，各方都不会理解错误，而且最终代码实现的时候，术语在代码里都要有体现，整个代码看起来就像是用代码把术语翻译了一遍一样！
2. 代码质量问题
   这里的代码质量不是指代码是否规范，而是说代码是否如实的实现了业务，实现的好不好。好不好不是说你的程序跑的有多快，而是业务逻辑是否清晰。业务术语，业务规则，业务流程在代码里是否有清晰的对应关系。如果有新的小伙伴加入，要改一个需求，他能否直接通过已有的代码就能把业务梳理清楚，并清楚地知道需要改哪些地方。而且改好了之后，确信自己改的地方不会影响其他人的代码。
   读到这里也许你会嗤之以鼻，你心想，这可能吗？怕不是痴人说梦，理想主义？

理论上，严格以DDD方式实现的代码就能做到。DDD就是要解决上述的“代码质量”的问题。
前提是所有参与编码的人，不管是老人还是新人，都熟知DDD的编码规则/习惯。

后续的文章里我会给出demo代码，你看完DDD这种风格的代码后，就能体会到我所说的。不出意外，你还会有一种如梦方醒的赶脚。

好，说完了DDD解决的问题，来看看什么是DDD？
中文名叫领域驱动设计，它是一种架构模式。注意架构模式不是架构风格，架构模式采用DDD，具体的架构风格可以是六边形架构或者CQRS架构或者六边形架构+CQRS。很明显，架构模式是个高度抽象的东西，是以“领域”为中心的指导方针，具有高瞻远瞩的特性。
怎么感觉越说越像官话了。 总之一句话，领导层用它来画蓝图（ppt)，底层实现的人用它来开展业务梳理和编码的工作。

## DDD的战略工具

领导们用的，就是划分领域，子领域，构建限界上下文映射图。

## DDD的战术工具

这是团队开发人员需要关注的，具体的有：
聚合、实体、值对象、领域服务、领域事件。
虽然看起来概念有点多，但是这些概念非常重要，非常重要，非常重要。
这些名词不是什么高大上的东西，它们只是工具。我们要想把活儿干好，首先要了解有哪些可用的工具，哪些场合应该用哪种工具。
只有熟悉了这些工具的用途和使用场景，我们才能码出“高质量”的代码。

# SpringBoot+JPA实现DDD

## 业务需求

假设我们要实现一个商品中心这个核心领域。要求如下：

- 商品包含一个或多个明细。一个明细也可以被包含在多个商品里。明细有三种：在线课程、实体书、线下服务。明细不可单独售卖，但可以单独编辑
- 商品和明细都有类目
- 商品的类目和明细的类目可以保持一致，也可以不保持一致
- 明细在不同的商品中可以有不同的价格
- 商品的价格是各明细的价格的总和。商品的价格不可修改，可通过添加优惠券实现减价。
- 在线课程的有效期分两种：截止日期，下单后xx月。
- 商品有状态，必须是上架状态才可以售卖，上架后不可修改
- 商品要审核后才能上架，因为商品内可能有违规的图片、文字，所以必须要经过法务审核

## 构建模型

1. 商品在其生命周期内是可修改的，且有唯一的标识，很明显是个实体。并且，商品是跟外界交互的入口，它是一个聚合根。
2. 课程有唯一的标识，可被修改，虽然它被包含在商品内，但是它可以单独编辑。一个商品被下架了，但是这个课程在其它商品里仍然可以被售卖。它的生命周期是独立的，所以它也是一个聚合根。
3. 实体书和线下服务同理，也是聚合根。
4.  优惠券比较特殊。它有自己的生命周期。如果优惠活动很多，也很复杂，应该将优惠拆分成一个单独的支撑领域。这样优惠可以做的很复杂，比如弄一个规则引擎来配置各种优惠券。这里我只做一个实现DDD的demo，不搞那么复杂，暂时不考虑优惠。
5. 审核也比较特殊，它应该是一个通用领域。这里不考虑审核。

## 实现模型

需要数据库设计吗？ 个人认为不需要了。还记得hibernate的ORM和自动建表的功能吗？曾经我们对hibernate弃如敝履，现在回过头来看，也许我们用错了。
我们总以为Hibernate自动生成ddl的功能很鸡肋，那是因为我们太习惯于先建表，后写代码了。我们一直以为数据库设计是基石，只有把数据库设计好了，才能可靠地实现业务。人家DDD根本不是这么玩的。 先设计数据库，开发人员跟业务之间就被这个数据库的表结构给隔离开了。其关系是： 业务 <--> 数据库 <--> 开发。
DDD的玩法是： 开发 <--> 业务 <--> 仓储（数据库），看到区别了吗？这种玩法是直面业务。DDD好玩的地方就在于：***真正的程序员敢于直面复杂的业务！***

我知道大家嫌弃hibernate，很重要的一个原因是关联表查询的时候很难受。一提到这个大家都感同身受。现在为什么我改变主意了呢？因为实现DDD的时候可以采用CQRS（读写分离）架构。这样一来，在写数据的时候只需要少量简单的查询即可，复杂的查询放在读模型里，读模型不需要跟写模型保持一致，读的时候可以选择原生的SQL，Mybatis，或者NoSQL，再也不用担心配置复杂的一对多，多对多等问题了。

据我所知，.Net的小伙伴比较熟悉DDD，可能是.Net有一整套DDD的框架吧。有人说Java用了Spring框架，天生的只能用贫血结构。因为实体里没有save,update等方法，所以是贫血的。这种说法是有问题的。原因我在[DDD入门之理解面向对象(一）](https://www.cnblogs.com/ahau10/p/13461236.html)这篇文章里已经说过了，这里不再赘述了。 如果有不同看法，欢迎留言讨论。DDD最有争议的就是这个“贫血”和“充血”模型了吧。

我打算用Spring Boot + JPA写一个DDD的demo。Spring Data JPA 默认就是Hibernate实现的，本系列文章JPA就是指Spring Data JPA, 具体的ORM框架就是Hibernate。

## 从聚合根开始

上一篇已经把业务需求描述清楚了，现在我们来实现它。

## 环境

- JDK1.8+
- Maven3.5+
- Mysql8.0
- Intellij Idea lombok 插件（注意安装插件要给Idea配置代理，否则装不上）

1. 新建Spring Boot工程
2. 新建包结构
   我们知道DDD有四层架构。
   - 用户接口层
   - 应用层
   - 领域层
   - 基础设施层
     按照这个结构我们分别建4个包： `ui`, `application`, `domain`, `infrastructure`。
3. 实现模型
   没有表结构突然不知道从哪里开始了？以前因为已经有表结构了，我们一开始用工具自动生成entity，然后就开始写controller，service，dao了。
   DDD是以领域为核心的,领域里的模型是稳定的，不管外部怎么变化，我们的模型是保持不变的。注意这里说的“稳定”、“不变”是指项目上线后不变，在开发阶段，模型是要不断优化调整的。所以我们就从模型开始。当然如果你的项目要先跟别人定好接口再开发，那你可以先从controller开始。然后构建模型。

在``domain`下新建`model.product.Product`类。

```java
/**
 * 商品聚合根
 * 没必要把所有的属性一股脑儿暴露出来，要修改“我”的属性，请调用“我”的方法，因为“我”的属性我自己最清楚
 */
@Entity
@Getter
@EqualsAndHashCode(of = "productNo")
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 因为hibernate需要一个无参构造函数。且这里的访问权限给的是protected，这样是防止外部直接`new Product()`创建一个空的商品。
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Product implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    /** 唯一的产品编码 */
    @Column(name = "product_no", length = 32, nullable = false, unique = true)
    private String productNo;
    /** 名称 */
    @Column(name = "name", length = 64, nullable = false)
    private String name;
    /** 价格 */
    @Column(name = "price", precision = 10, scale = 2)
    private BigDecimal price;
    /** 类目 */
    @Column(name = "category_id", nullable = false)
    private Integer categoryId;
    /** 状态 */
    @Column(name = "product_status", nullable = false)
    private Integer productStatus;
    /** 是否允许跨类目 */
    @Column(name = "allow_across_category", nullable = false)
    private Boolean allowAcrossCategory;
    /** 备注字段 */
    @Column(name = "remark", length = 256)
    private String remark;

    public static Product of(String productNo, String name, BigDecimal price, Integer categoryId, Integer productStatus, String remark, Boolean allowAcrossCategory) {
        return new Product(null, productNo, name, price, categoryId, productStatus, remark, allowAcrossCategory);
    }
}
```

类的属性用JPA的@Column跟db表的字段对应起来，并且类的属性跟业务密切相关。
此外，除了一个自增的主键，商品应该还一个唯一的产品编码。这个唯一的产品编码就是业务主键，跟外部交互的时候都使用这个业务主键。这至少有3个好处：

- 对前端不会暴露我们的实现
- 如果有一天需要迁移数据的时候，因为业务主键是稳定的，很好迁移。而物理主键是会变的，迁移到另一张表可能还会有主键冲突,到时候就很难受。
- 业务主键是可读的，并且其本身包含了一些有用信息。

如果有参构造函数访问权限是public。这意味着，其他地方可以随意的创建一个商品。问题是他们知道如何正确的创建一个商品吗？
也许你会说，我们把创建商品需要的业务规则都放在这个构造函数里不就行了吗？ 行是行，就是不灵活了。假如某一天我们想返回Product的一个子类怎么办？

所以我们应该提供一个工厂方法。由这个工厂方法统一创建商品。 双击构造函数名称，右击鼠标 Refactor >> Replace Constructor with Factory Method
输入工厂方法名`of`。 你会看到，idea自动把构造函数变成了私有的方法。再看看代码，好像有点“坏味道”，既然已经用了lombok，为什么还要自己写一个构造函数呢。
把有参构造函数删掉， 在类上加一个 `@AllArgsConstructor(access = AccessLevel.PRIVATE)`

有参数构造函数的访问权限是private。 第一次见到这个你可能会觉得不可思议，因为以前你从来没想过要把构造函数变成私有的。
不仅如此，setter和getter也是随便给。这是不对的，DDD的代码要严格控制访问权限，这样才能最大程度上保证模型的稳定。不然就会出现一个属性的值不知道在什么地方被改了，你却不知道的情况。一旦出现这样的bug，简直就是灾难。

虽然实体是可被修改的，但不代表所有属性都随便调用setter轻轻松松就改掉了。
如果确实需要修改某个属性，请提供一个具体的方法，比如`changeProductStatus`，这个方法跟业务上也应该有对应关系，否则就没必要单独写一个方法。
这才叫封装嘛，你说是不是？

**没有setter和getter，hibernate还能实现持久化吗？**
以前的hibernate要求entity必须有setter和getter，现在不需要了。

工厂方法也有点问题。主键id是自动生成的，怎么能让程序传进来呢。所以工厂方法删除id这个参数，在调用Product有参构造函数的时候id传一个null。

1. 启动项目

   在mysql里创建一个名为`product_center`的库，启动项目。hibernate自动为我们生成了一个product表。

2. 复写equals和hashCode方法(重要)
   使用`productNo`生成的equals和hashCode方法。`Product`的`productNo`是唯一的，两个实体，只要这个字段相同，就认为是同一个实体。

## 构建多对多关系

前面已经分析过，一个商品可以包含一个或多个课程明细。课程明细可以单独编辑，有自己的生命周期，课程明细也是一个聚合根。

1. 在`domain.model`包下创建 `courseitem.CourseItem`类，内容如下：

   ```java
   /**
    * 课程明细
    */
   @Entity
   @Getter
   @EqualsAndHashCode(of = "itemNo")
   @NoArgsConstructor(access = AccessLevel.PROTECTED)
   @AllArgsConstructor(access = AccessLevel.PRIVATE)
   public class CourseItem implements Serializable {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;
       /** 唯一的明细编码 */
       @Column(name = "item_no", length = 32, nullable = false, unique = true)
       private String itemNo;
       /** 名称 */
       @Column(name = "name", length = 64, nullable = false)
       private String name;
       /** 类目 */
       @Column(name = "category_id", nullable = false)
       private Integer categoryId;
       /** 价格 */
       @Column(name = "price", precision = 10, scale = 2)
       private BigDecimal price;
       @Column(name = "remark", length = 256)
       private String remark;
       @Column(name = "study_type", nullable = false)
       private Integer studyType;
       /** 下单后xx月 */
       @Column(name = "period")
       private Integer period;
       /** 截止日期 */
       @Temporal(TemporalType.TIMESTAMP)
       @Column(name = "deadline")
       private Date deadline;
   
       public static CourseItem of(String itemNo, String name, Integer categoryId, BigDecimal price, String remark, Integer studyType, Integer period, Date deadline) {
           return new CourseItem(null, itemNo, name, categoryId, price, remark, studyType, period, deadline);
       }
   }
   ```

   产品跟课程明细是多对多的关系，这个关系怎么处理？是不是要配置 `@ManyToMany`啊？
   不要，因为**模型里的代码应该是框架无关的**。 `@ManyToMany`是hibernate的注解，我们应该避免使用JPA具体实现的注解，而应该多用JPA通用的注解。
   也许你会反驳我说，既然这样，Entity类就应该保持纯洁性，为什么我还在Entity类里使用JPA相关的注解？JPA虽然不是框架，但是在实体类里写`@Column`这种DB相关的东西真的好吗？

   这是个好问题。用JPA的原因是不给自己找麻烦。既然使用了Spring这个框架，框架提供了Spring Data JPA这么成熟好用的工具我们为什么不用呢。
   没必要自己再写一套东西，把非常纯洁的实体对象转成持久化对象后再持久化它。 有种重复造轮子的感觉不说，还容易出错。
   个人觉得实体里加一些JPA的注解是可以忍受的，不是什么很严重的问题。油管上看到的视频，有人问过大神这个问题，大神就是这么回答的。

   我们知道要描述多对多的关系需要维护一张中间表。@Entity注解的类可以直接生成表，那么商品-明细这个中间表怎么生成呢？

   需要使用JPA的2个注解。`@Embeddable`和`@ElementCollection`。

2. 在product包下新建`ProductCourseItem`类，内容如下：

   ```java
   @Embeddable
   @Getter
   @EqualsAndHashCode
   @NoArgsConstructor(access = AccessLevel.PROTECTED)
   @AllArgsConstructor(access = AccessLevel.PRIVATE)
   public class ProductCourseItem implements Serializable {
       @Column(name = "course_item_no", length = 32, nullable = false)
       private String courseItemNo;
       @Column(name = "new_price", precision = 10, scale = 2)
       private BigDecimal newPrice;
   
       public static ProductCourseItem of(String courseItemNo, BigDecimal retakePrice) {
           return new ProductCourseItem(courseItemNo, retakePrice);
       }
   }
   ```

   注意，ProductCourseItem是一个值对象，值对象是不能被修改的。所以这个类只提供了getter，并没有提供setter。

   `Product`类添加如下：

   ```java
   @ElementCollection(targetClass = ProductCourseItem.class)
   @CollectionTable(
           name = "product_course_item",
           uniqueConstraints = @UniqueConstraint(columnNames = {"product_no", "course_item_no"}),
           joinColumns = {@JoinColumn(name = "product_no", referencedColumnName = "product_no")}
       )
   private Set<ProductCourseItem> productCourseItems;
   ```

   并且修改`of`工厂方法：

   ```java
   public static Product of(String productNo, String name, BigDecimal price, Integer categoryId, Integer productStatus, String remark, 
                                              Boolean allowAcrossCategory, Set<ProductCourseItem> productCourseItems) {
       return new Product(null, productNo, name, price, categoryId, productStatus, remark, allowAcrossCategory, productCourseItems);
   }
   ```

   商品的课程明细不能重复，所以我们使用Set集合。
   中间表的名称是`product_course_item`，并且给中间表加一个唯一复合索引 —— 商品的`product_no`和明细的`course_item_no`组成一个唯一索引。

   到这里也许你会奇怪，中间表`product_course_item`里并没有声明`product_no`这个字段啊。 别担心，因为Product类里有一个`@ElementCollection`。这个注解会帮我们在中间表里生成`product_no`这个字段。

   **为什么不在Product里直接引用CourseItem呢？**
   聚合根可以直接引用实体，值对象。 不能直接引用其它聚合根，要通过唯一标识来关联。

   **就算用唯一标识来关联，为什么不用物理主键而用业务主键关联呢？**
   哈哈，能问出这个问题，说明你真的在认真看我的文章了。通常我们都使用物理主键来做关联。 但其实db规范里并没有强制要求我们使用物理主键来做关联。
   正如我在上一篇文章里说的，使用业务主键有很多好处，用业务主键做关联除了多占了一些空间外，我实在想不通有什么不好？

3. 启动项目，hibernate会删除之前的表，重新生成新的表结构.

   中间表有了一个唯一复合索引，这样可以在db层面上保证不会重复。

4. 问题解答

   ①中间表为什么会有一个`new_price`字段？

   因为同一个课程明细在不同的商品下价格不同。

   ②`ProductCourseItem`类的equals方法是由`@EqualsAndHashCode`注解实现的。`ProductCourseItem`类只有2个字段，那么注解自动生成的equals方法里只会比较这2个字段。 为什么没有算上`productNo`?

   好问题。 不需要算上`productNo`，因为**ProductCourseItem**不会单独使用，它只会存在于某个**Product**里，这天然地保证了它们的productNo都是一样的，所以equals方法也就没必要算上productNo了。

## 优化Entity,类型改为值对象

前面我们已经定义了2个聚合根，定义了2个聚合根之间的关系，并且自动生成了表结构。
在实现具体的业务前，优化一下我们的Entity。

```java
@Column(name = "product_no", length = 32, nullable = false, unique = true)
private String productNo;
@Column(name = "name", length = 64, nullable = false)
private String name;
@Column(name = "price", precision = 10, scale = 2)
private BigDecimal price;
@Column(name = "category_id", nullable = false)
private Integer categoryId;
@Column(name = "product_status", nullable = false)
private Integer productStatus;
```

咦？是不是有点眼熟？跟之前三层架构写的entity类有啥区别？没有区别，因为都是一些简单的字段跟DB对应一下就完事了。
这正是我们需要优化的地方，在实现DDD的时候我们应该尽量多使用**值对象**。

- 比如`productNo`这个字段，生成商品码这个方法放在哪里比较合适？放在`Product`里？
- 比如`price`这个字段，假如我们希望加一个币种字段怎么办？ 直接再加一个`@Column`？
- 比如`productStatus`这个字段，它应该是一个枚举对不对？定义成Integer类型我们看代码根本就不知道这个数字代表什么对不对？

把它们定义成值对象问题就迎刃而解了。解决问题的同时还收获了额外的好处：
我们的代码更加`OO（面向对象）`了。Entity类不再是一个简单的ORM类了，它是一个真正的模型对象了。

生成商品编码的方法放在ProductNumber里再适合不过了。
①新建`domain.model.product.ProductNumber`：

```java
@Getter
@EqualsAndHashCode
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class ProductNumber implements Serializable {
    private String value;

    public static ProductNumber of(Integer categoryId) {
        checkArgument(categoryId != null, "商品类目不能为空");
        checkArgument(categoryId > 0, "商品类目id不能小于0");
        return new ProductNumber(generateProductNo(categoryId));
    }

    public static ProductNumber of(String value) {
        checkArgument(!StringUtils.isEmpty(value), "商品编码不能为空");
        return new ProductNumber(value);
    }

    private static String generateProductNo(Integer categoryId) {
        // TODO 生成商品编码
    }
}
```

四个注意点（**非常重要**）：

- 商品编码是业务主键，它应该是用户可读的，并且本身包含了一些有用信息。
  我们定义商品码的生成规则为：PRODUCT + 4位类目 + 当前时间 + 4位随机数 共32位。
- 检查参数的时候，我们全部使用guava包的`checkArgument`方法，而不是`checkNotNull`方法。因为我们这是业务代码，不能把空指针异常返回给客户端。
  我们要提供用户可读的错误信息。
- 值对象是不可修改的，只提供getter就行了。
- 值对象的`equals`和`hashCode`方法，与实体有唯一标识不同，值对象没有唯一标识，两个值对象所有的属性值相等才能判定相等。

然后将`private String productNo;` 替换成 `private ProductNumber productNo;`。

②新建`domain.model.product.ProductStatusEnum`:

```java
@AllArgsConstructor
public enum ProductStatusEnum {
    // 新建
    DRAFTED(1000111, "草稿"),
    // 待审核
    AUDIT_PENDING(1000112, "待审核"),
    // 已上架
    LISTED(1000113, "已上架"),
    // 已下架
    UNLISTED(1000114, "已下架"),
    // 已失效
    EXPIRED(1000115, "已失效");

    @Getter
    // @JsonValue
    private Integer code;

    @Getter
    private String remark;

    public static ProductStatusEnum of(Integer code) {
        ProductStatusEnum[] values = ProductStatusEnum.values();
        for (ProductStatusEnum val : values) {
            if (val.getCode().equals(code)) {
                return val;
            }
        }
        // throw new InvalidParameterException(String.format("【%s】无效的产品状态", code));
        return null;
    }
}
```

**为什么是枚举而不是字典？**
个人觉得符合以下特征才应该使用字典，否则就应该用枚举：

- 子项可动态修改，而且修改比较频繁
- 修改子项不影响现有业务逻辑，也就是说代码不用动

像商品状态这种字段，每个状态都很业务密切相关。如果你把它放在字典里，只在字典里新加了一个状态没有用，因为代码里还得修改相关业务逻辑。

将`private Integer productStatus;`替换成`private ProductStatusEnum productStatus;`

对应调整一下`of`工厂方法。

③新建Price值对象
商品和课程明细都有价格，我们可以把Price放在一个公共的地方。
在domain下新建`common.model.Price`, 内容如下：

```java
@Embeddable
@Getter
@EqualsAndHashCode
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Price implements Serializable {

    //@Convert(converter = CurrencyConverter.class)
    @Column(name = "currency_code", length = 3)
    private Currency currency;
    @Column(name = "price", nullable = false, precision = 10, scale = 2)
    private BigDecimal value;

    public static Price of(String currencyCode, BigDecimal value) {
        checkArgument(!StringUtils.isEmpty(currencyCode), "币种不能为空");
        checkArgument(value != null, "价格不能为空");
        checkArgument(value.compareTo(BigDecimal.ZERO) > 0, "价格必须大于0");
        try {
            return new Price(Currency.getInstance(currencyCode), value);
        } catch (IllegalArgumentException e) {
            throw new InvalidParameterException(String.format("【%s】不是有效的币种", currencyCode));
        }
    }
}
```

在值对象里验证币种的有效性很合理对不对？否则每次用到币种的时候都得判断一下是否有效。一个处理业务逻辑的方法里到处都是if判断，不雅观不说，
还影响看代码的思路。

将`Product`的

```java
@Column(name = "price", precision = 10, scale = 2)
private BigDecimal price;
```

替换成

```java
@Embedded
private Price price;
```

④自定义异常
定义一个通用的运行时异常：

```java
@NoArgsConstructor
@AllArgsConstructor
@Setter
@Getter
public class BusinessException extends RuntimeException {
    private String code;
    private String message;
}
```

具体的业务异常：

```java
public class InvalidParameterException extends BusinessException {
    private static final String CODE = "invalid-parameter";

    public InvalidParameterException(String message) {
        super(CODE, message);
    }

}
```

异常code定义成String类型，这样看到异常编码就能知道是哪种异常，如果定义成int类型，还得查表之后才能知道是哪种异常。

CourseItem类同理，这里就不再重复了。

## 实现功能

篇幅所限，我们以创建商品、上下架商品 这两个功能为例：

### domain

我们已经有了一个创建商品的工厂方法`of`，但是里面没有业务逻辑，现在来补充业务逻辑。
`of`方法了参数太多了，我们把它放在Command类里。Command不属于领域对象，应该放在哪个包下面呢？
放在`application`包下。在`appliction`这个包下新建一个`command`包，再新建一个`CreateProductCommand`类：

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class CreateProductCommand {
    private String name;
    private Integer categoryId;
    private String currencyCode;
    private BigDecimal price;
    private String remark;
    private Boolean allowAcrossCategory;
    private Set<ProductCourseItem> productCourseItems;

    public static CreateProductCommand of(String name, Integer categoryId, String currencyCode, BigDecimal price, String remark, Boolean allowAcrossCategory, Set<ProductCourseItem> productCourseItems) {
        return new CreateProductCommand(name, categoryId, currencyCode, price, remark, allowAcrossCategory, productCourseItems);
    }
}
```

这个`of`方法的参数太多了，用起来非常不方便不说，看起来也不向面向对象的写法。改成如下：

```java
public static Product of(CreateProductCommand command) {
    Integer categoryId = command.getCategoryId();
    checkArgument(!StringUtils.isEmpty(command.getName()), "商品名称不能为空");
    checkArgument(categoryId != null, "商品类目不能为空");
    checkArgument(categoryId > 0, "商品类目id不能小于0");
    // 生成产品码时有限制，该字段不能超过4位
    checkArgument(categoryId < 10000, "商品类目id不能超过10000");
    checkArgument(command.getAllowAcrossCategory() != null, "是否跨类目不能为空");

    Price price = Price.of(command.getCurrencyCode(), command.getPrice());
    if("CAD".equalsIgnoreCase(price.getCurrency().getCurrencyCode())){
        throw new NotSupportedCurrencyException(String.format("【%s】对不起，暂不支持该币种", command.getCurrencyCode()));
    }
    ProductNumber newProductNo = ProductNumber.of(categoryId);
    ProductStatusEnum defaultProductStatus = ProductStatusEnum.DRAFTED;

    Product product = new Product(null, newProductNo, command.getName(), price, categoryId, defaultProductStatus, 
                                        command.getRemark(), command.getAllowAcrossCategory(), command.getProductCourseItems());
    return product;
}
```

等等，我们创建商品的时候似乎缺了点什么。需求里有一句“明细的类目可以跟商品保持一致，也可以不保持一致”，这条业务规则我们好像还没有实现。
当允许跨类目的时候，商品和明细的类目不用保持一致，但是当不允许跨类目的时候，商品和明细的类目必须保持一致。
很明显我们需要一个判断商品及明细类目是否一致的方法。问题来了，这个方法放在哪里合适？ 放在商品里，然后把明细集合传到`of`方法里？
不行，前面说过了，聚合根和聚合根之间不要直接引用。 那怎么办？

两种办法：

- 将课程这个实体转成一个值对象作为参数传给商品
- 使用域服务（个人推荐使用这种方式）

当某些功能放在任何一个实体里都不合适的时候，我们需要把它放在域服务(domain service)里。
在域服务里将明细实体查出来，然后挨个比对类目是否一致。

**域服务里能使用repository吗？**
可以。但是一般不推荐。那为什么我还要在域服务里注入repository呢？ 因为我想让application service尽可能地薄一点。

新建`domain.model.product.ProductManagement`

```java
@Component
public class ProductManagement {
    private CourseItemRepository courseItemRepository;

    // 使用构造器的方式注入，因为@Autowired等注解注入方式容易上瘾:)
    public ProductManagement(CourseItemRepository courseItemRepository) {
        this.courseItemRepository = courseItemRepository;
    }

    /**
     * 检查明细的项目跟商品的项目是否保持一致
     * 因为涉及了另一个聚合根CourseItem，把CourseItem实体转成值对象好麻烦
     * 所以把这段逻辑放在domain service里
     *
     * @param allowCrossCategory 是否允许跨类目
     * @param categoryId         商品类目id
     * @param productCourseItems 明细信息
     */
    public void checkCourseItemCategoryConsistence(Boolean allowCrossCategory, Integer categoryId, Set<ProductCourseItem> productCourseItems) {
        checkArgument(allowCrossCategory != null, "是否允许跨类目不能为空");
        checkArgument(categoryId != null, "商品类目不能为空");

        // 检查编码对应的明细是否存在，这个不算business logic
        List<CourseItemNumber> itemNos = productCourseItems.stream().map(item -> CourseItemNumber.of(item.getCourseItemNo())).collect(Collectors.toList());
        List<CourseItem> courseItems = courseItemRepository.findByItemNos(itemNos);
        Map<CourseItemNumber, List<CourseItem>> courseItemMap = courseItems.stream().collect(groupingBy(CourseItem::getItemNo));
        List<String> notFoundItemNos = itemNos.stream().filter(itemNo -> !courseItemMap.containsKey(itemNo))
                .map(item -> item.getValue())
                .collect(Collectors.toList());
        if (!CollectionUtils.isEmpty(notFoundItemNos)) {
            throw new NotFoundException(String.format("明细【%s】未找到", String.join(",", notFoundItemNos)));
        }

        // 不允许跨类目时才需要检查类目是否一致，这个是business logic，前面的查询就是为这里服务的
        if (!allowCrossCategory) {
            List<CourseItem> unmatchedCourseItems = getUnmatchedCourseItems(categoryId, courseItems);
            if (!CollectionUtils.isEmpty(unmatchedCourseItems)) {
                List<String> unmatchedItemNos = unmatchedCourseItems.stream().map(item -> 
                                           item.getItemNo().getValue()).collect(Collectors.toList());
                throw new CategoryNotMatchException(String.format("明细【%s】类目不匹配", String.join(",", unmatchedItemNos)));
            }
        }
    }

    private List<CourseItem> getUnmatchedCourseItems(Integer productCategoryId, List<CourseItem> courseItems) {
        return courseItems.stream().filter(item -> !item.getCategoryId().equals(productCategoryId))
                .collect(Collectors.toList());
    }

}
```

注意，`Product.of`方法和`ProductManagement.checkCourseItemCategoryConsistence`方法加起来才是完整的创建商品的逻辑。看起来有点散，
但是别忘了，创建商品时会先经过application service。 application service提供了创建商品的统一入口。从外部看来，它只需要调用applicaton service
的`createProduct`方法即可。 至于真正创建商品时用了几个domain service外部是不知道的，也不需要知道。

**还有一条业务规则没实现？**
很好，被细心的你发现了。 “商品的价格是明细价格的总和”这条业务规则还没实现。 这个我就不写了，留给读者自己实现。TODO

商品上下架功能：

```java
public void listing() {
    if(this.productStatus.getCode() < ProductStatusEnum.APPROVED.getCode()){
        throw new NotAllowedException("已审核通过的商品才允许上架");
    }
    this.productStatus = ProductStatusEnum.LISTED;
}
public void unlisting() {
    if(!this.productStatus.equals(ProductStatusEnum.LISTED)){
        throw new NotAllowedException("已上架的商品才允许下架");
    }
    this.productStatus = ProductStatusEnum.UNLISTED;
}
```

### application

domain层的代码写完了，在应用层调用它。application service是很薄的一层，做的工作比较少。

新建`application.ProductService`：

```java
public interface ProductService {
    Product createProduct(CreateProductCommand command);
}
```

新建`application.impl.ProductServiceImpl`：

```java
@Service
public class ProductSerivceImpl implements ProductService {

    private ProductRepository productRepository;
    private ProductManagement productManagement;

    public ProductSerivceImpl(ProductRepository productRepository, ProductManagement productManagement) {
        this.productRepository = productRepository;
        this.productManagement = productManagement;
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Product createProduct(CreateProductCommand command) {
        Set<ProductCourseItem> productCourseItems = command.getProductCourseItems();
        if (CollectionUtils.isEmpty(productCourseItems)) {
            throw new IllegalArgumentException("明细不能为空");
        }

        // 不允许跨类目的商品，明细类目要跟商品类目保持一致。思来想去，这个逻辑还是放在domain service里好
        productManagement.checkCourseItemCategoryConsistence(command.getAllowAcrossCategory(), command.getCategoryId(), 
                                                                                               productCourseItems);
        Product product = Product.of(command);
        productRepository.save(product);
        return product;
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Integer unlistingProduct(String productNo) {
        checkArgument(!StringUtils.isEmpty(productNo), "商品编号不能为空");
        Product product = productRepository.findByProductNo(ProductNumber.of(productNo));
        if (product == null) {
            throw new NotFoundException(String.format("商品【%s】未找到", productNo));
        }
        ProductStatusEnum oldStatus = product.getProductStatus();
        product.unlisting();
        productRepository.update(product);
        return oldStatus.getCode();
    }

}
```

### repository

在`model.product`包下新建接口：

```java
public interface ProductRepository {
    void save(Product product);
    void update(Product product);
    Product findByProductNo(ProductNumber productNo);
}
```

在`infrastructure`包新新建实现类

```java
@Repository
public class HibernateProductRepository extends HibernateSupport<Product> implements ProductRepository {
    HibernateProductRepository(EntityManager entityManager) {
        super(entityManager);
    }

    @Override
    public Product findByProductNo(ProductNumber productNo) {
        if (StringUtils.isEmpty(productNo)) {
            return null;
        }
        Query<Product> query = getSession().createQuery("from Product where productNo=:productNo and isDelete=0", Product.class).setParameter("productNo", productNo);
        return query.uniqueResult();
    }
}

abstract class HibernateSupport<T> {

    private EntityManager entityManager;

    HibernateSupport(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    Session getSession() {
        return entityManager.unwrap(Session.class);
    }

    public void save(T object) {
        entityManager.persist(object);
        entityManager.flush();
    }

    public void update(T object) {
        entityManager.merge(object);
        entityManager.flush();
    }
}
```

`@Entity`里的值对象如何持久化？
需要用到转换器。
以ProductNumber为例,在model里定义如下转换器：

```java
@Converter
public class ProductNumberConverter implements AttributeConverter<ProductNumber, String> {
    @Override
    public String convertToDatabaseColumn(ProductNumber productNumber) {
        return productNumber.getValue();
    }

    @Override
    public ProductNumber convertToEntityAttribute(String value) {
        return ProductNumber.of(value);
    }
}
```

### ui

这个就很简单了，跟以前一样使用`Controller`。
注意，这里接收参数的叫`payload`， `payload`要把自己转化成`command`之后再调用application service。

```java
@PostMapping("/api/v1/product/create")
public ApiResult<Product> createProduct(@RequestBody CreateProductPayload createProductPayload) {
    CreateProductCommand command = createProductPayload.toCommand();
    Product product = productService.createProduct(command);
    return ApiResult.ok(product);
}
```

payload:
看到没有，payload跟command不一样，payload有get，set方法，为了省事，我直接用`@Data`这个注解了。

```java
@Data
public class CreateProductPayload {
    private String name;
    private Integer categoryId;
    private String currencyCode;
    private BigDecimal price;
    private String remark;
    private Boolean allowAcrossCategory;
    private Set<ProductCourseItemPayload> productCourseItems;

    public CreateProductCommand toCommand() {
        Set<ProductCourseItem> itemRelations = productCourseItems.stream()
                .map(item -> ProductCourseItem.of(item.getCourseItemNo(),
                        item.getRetakeTimes(), item.getRetakePrice())).collect(Collectors.toSet());
        return CreateProductCommand.of(name, categoryId, currencyCode, price, remark, allowAcrossCategory, itemRelations);
    }

    @Data
    public static class ProductCourseItemPayload {
        private String courseItemNo;
        private Integer retakeTimes;
        private BigDecimal retakePrice;

    }
}
```

### Restful or Not?

我不推荐使用restful。
可以看看[淘宝商品中心开发api](https://open.taobao.com/api.htm?docId=4&docType=2)。它采用的Richardson Maturity Model(成熟度模型)是level 1。所有的请求都是post请求。
原因有三：

- 这种api兼容性最好，因为其它语言的框架可能不支持 `PUT`， `DELETE`这样的方法。
- 每个url都是由动词结尾，意思很明确。
- 资源(名词)单复数分的很清楚。操作单个资源就用单数，操作多个资源就用复数。 而Restful单复数就很难分清楚

### 小结

- 本系列文章旨在说明如何使用Spring Boot+JPA实现DDD，关于DDD战术工具（聚合、实体、值对象、域服务、仓储、领域事件）细节没有详细说明。
  代码只是演示了如何使用这些战术工具。如果你懒得看这些战术工具的定义，不妨直接从代码里感受一下，然后回过头来再看定义可能印象更深刻。
- 战略上如何划分子领域，如何构建上下文映射图也没有说。这个其实是非常非常重要的，如果一开始领域都划分错了，后面写出来的代码也是有问题的。作者水平有限，实在不知道怎么说这个东西。作为一个IT民工，能用好DDD战术工具就很不错了。战略上的东西更多是领导层面决定的。
- 如果你仔细看完本系列文章会发现这个demo项目不完整。商品查询怎么办？尤其是关联查询怎么办？这个就要提一下DDD的架构风格了。其中一种架构风格是CQRS（读写分离），商品中心就很适合用这个。这就是说，应该再起一个项目，可以使用mybatis或者jdbc，这个项目专门用来查询。 另一种架构风格是事件驱动。比如订单系统比较复杂，关联的领域比较多，事件也多，非常适合用事件驱动。这些东西就有待大家自己探索了。

## 润色一下

### 记录sql语句及sql的执行时间

```xml
<properties>
    <p6spy.version>3.9.0</p6spy.version>
</properties>
<dependency>
    <groupId>p6spy</groupId>
    <artifactId>p6spy</artifactId>
    <version>${p6spy.version}</version>
</dependency>
```

src/main/resources下新建spy.properties配置文件：

```properties
driverlist=com.mysql.cj.jdbc.Driver
logfile=spy.log
dateformat=yyyy-MM-dd HH:mm:ss.SS
logMessageFormat=com.p6spy.engine.spy.appender.CustomLineFormat
customLogMessageFormat=- %(currentTime) | took %(executionTime)ms | connection %(connectionId) \nEXPLAIN %(sql);\n
filter=true
exclude=select 1 from dual
```

application.properties修改成：

```properties
#spring.datasource.url=jdbc:mysql://localhost:3306/product_center?\ useSSL=false&serverTimezone=Asia/Shanghai&zeroDateTimeBehavior=convertToNull&useUnicode=true&characterEncoding=UTF-8
spring.datasource.url=jdbc:p6spy:mysql://localhost:3306/product_center?\ useSSL=false&serverTimezone=Asia/Shanghai&zeroDateTimeBehavior=convertToNull&useUnicode=true&characterEncoding=UTF-8
spring.datasource.username=<your username>
spring.datasource.password=<your password>
#spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver
```

### 增加全局异常处理

```java
@ControllerAdvice
@ResponseBody
public class GlobalDefaultExceptionHandler {
    private static final String UNKNOWN_ERROR_CODE = "unknown-error";
    private static final String SYSTEM_ERROR_INFO = "系统异常，请联系管理员";
    private static final String ILLEGAL_PARAM_CODE = "illegal-param";

    @ExceptionHandler(value = IllegalArgumentException.class)
    public ApiResult illegalParamExceptionHandler(IllegalArgumentException e) throws Exception {
        // log todo
        return ApiResult.error(ILLEGAL_PARAM_CODE, e.getMessage());
    }

    @ExceptionHandler(value = BusinessException.class)
    public ApiResult businessExceptionHandler(BusinessException e) throws Exception {
        // log todo
        return ApiResult.error(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(value = Exception.class)
    public ApiResult defaultErrorHandler(Exception e) throws Exception {
        // log todo
        e.printStackTrace();
        return ApiResult.error(UNKNOWN_ERROR_CODE, SYSTEM_ERROR_INFO);
    }
}
```

### 数据库添加自定义的审计字段

domain.common.model:

```java
@MappedSuperclass
@Data
public abstract class AuditEntity implements Serializable {
    @Column(name = "is_delete", columnDefinition = "TINYINT(1) DEFAULT 0")
    protected Boolean isDelete;
    @Column(name = "created_by", length = 11, nullable = false)
    protected Integer createdBy;
    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "created_at", columnDefinition = "DATETIME NULL DEFAULT CURRENT_TIMESTAMP")
    protected Date createdAt;
    @Column(name = "updated_by", length = 11, nullable = false)
    protected Integer updatedBy;
    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "updated_at", columnDefinition = "DATETIME NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP")
    protected Date updatedAt;

    @PrePersist
    protected void onCreate() {
        updatedAt = createdAt = new Date();
        isDelete = false;
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = new Date();
    }
}
public class Product extends AuditEntity implements Serializable {
      ...
}
```

确定需求 -> 构建模型 -> 实现模型 -> 调整模型 -> 实现模型...，走完这个过程，相信你对DDD的玩法已经有了一定的了解，而且我相信你大概能领略到DDD代码的优美之处。好的代码应该是高内聚低耦合的，DDD的代码就是要让高内聚低耦合落地。

高内聚体现在业务代码都集中在领域对象里了（聚合根，实体，值对象，域服务）。业务规则在代码里都有非常清楚的对应关系。代码真正体现了面向对象的思想。
低耦合体现在聚合根不直接引用其它聚合根。 低耦合还有一个很关键的点是**领域事件**。 这个跟DDD事件驱动的架构风格分不开。

说起DDD的架构风格，最常用的就是CQRS（读写分离）和事件驱动。
事件驱动又分 event storming和event sourcing（个人理解，不对请指正），event sourcing看起来比较极端，似乎应用场景针对性太强，也就是说比较少见。

如果你的业务很复杂，事件比较多，可以使用event storming。商品中心相对比较简单，用读写分离就差不多了。本人水平有限，本系列文章旨在抛砖引玉，希望大家能留言讨论。

# DDD 总结

## 概念

### 聚合根

独立的生命周期

## 四层架构(含包示例)

### 用户接口层

**常见约定包结构**

> `ui/interfaces`
> 	web
> 		controller
> 			common

### 应用层

**常见约定包结构**

> `web`
> 	controller
> 		common

**通常工作**

> - 开启事务
> - 查询实体（调用其它方法需要用到这些实体）
> - 调用实体的方法，或者域方法
> - 调用repository方法，持久化
> - 权限控制
> - 接收领域事件

### 领域层

**常见约定包结构**

> `domain`
> 	某某领域 - `xxx`
> 		模型 - `model` - 不一定要将数据库实体和其他分开
> 			某某领域2 - `xxx2`

### 基础设施层

**常见约定包结构**

> `infrastructure`
> 	权限控制 - `acl`
> 		缓存 - `codis/redis`
> 		配置 - `configuration`
> 		工具类 - `kit`
> 		日志配置 - `logger`

## FAQ

### command连set方法都没有，外部怎么将参数传进来？
这里要说一下DDD四层架构的玩法：

1. 用户接口层使用`payload`接收参数，`payload`把自己转成`command`传给应用层（application service）
2. 应用层开启事务，查询聚合根，调用领域层方法，调用资源库(repository)持久化实体
3. 领域层实现业务逻辑
4. 基础服务层负责持久化

接收参数是用户接口层的工作。用户接口层的`payload`会提供set&get方法的。我们现在实现的是领域层的东西，还没到应用层和用户接口层呢。

注意：

- command虽然不是领域对象，但是它可以引用领域对象，比如这里我们引用了ProductCourseItem这个值对象。
- command也是不可修改的。这里只提供了getter

### 为什么不直接将payload传给application？

command是相对稳定的东西。不管外部端口如何变化，只要能把接收到的参数转成相应的command。我们的领域模型就能提供相应的服务。
我们早就说过领域模型是稳定的，也就是说它能适应变化。 适应变化是指核心业务逻辑不变的情况下能适应不同的端口。`payload`的字段名称和类型可能不符合模型的要求，所以需要转成command。

# 原文

**文本绝大部分摘录自一下文章，最终的代码参见原文末尾。**

- [DDD入门之理解面向对象(一）](https://www.cnblogs.com/ahau10/p/13461236.html)
- [DDD入门之解决了什么问题（二）](https://www.cnblogs.com/ahau10/p/13512879.html)
- [SpringBoot+JPA实现DDD（一）](https://www.cnblogs.com/ahau10/p/13513642.html)
- [Spring Boot+JPA实现DDD（二）](https://www.cnblogs.com/ahau10/p/13516707.html)
- [Spring Boot+JPA实现DDD（三）](https://www.cnblogs.com/ahau10/p/13517585.html)
- [Spring Boot+JPA实现DDD（四）](https://www.cnblogs.com/ahau10/p/13518953.html)
- [SpringBoot+JPA实现DDD（五）](https://www.cnblogs.com/ahau10/p/13522928.html)
- [SpringBoot+JPA实现DDD（六）](https://www.cnblogs.com/ahau10/p/13524644.html)
- [文章分类 - 领域模型](https://www.cnblogs.com/legend886/category/971021.html)

