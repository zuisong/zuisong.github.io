# 单元测试 spock 实战

## 1 摘要

最近一段时间接触到了spock这个可以用于java和groovy项目的单元测试框架，写了一段时间单测之后认为这个框架不错，值得写一篇文章推广一下。

## 2 单元测试

很多人一谈到单元测试就会想到xUnit框架。对于一些java新人来说，会用jUnit就是会写单元测试，高级点的会捣鼓一下testng，然后就认为自己掌握了单元测试。

而实际上，很多人不怎么会写单元测试，甚至不知道单元测试究竟是干什么的。写单元测试要比写代码要难上许多，而这里说的难度跟框架没什么关系。

所以，在开始介绍spock之前，需要先抛开框架，谈谈单元测试本身的事情。在理解了单元测试之后才能更清楚spock框架是什么，以及它否能够更优雅的解决你的问题。

### 2.1 关于单元测试

#### 2.1.1 单元测试是什么

写代码免不了要做测试，测试有很多种，对于java来说，最初级的就是写个main函数运行一下看看结果，高级的可以用各种高大上的复杂的测试系统。每种测试都有它的关注点，比如测试功能是不是正确，或者运行状态稳不稳定，或者能承受多少负载压力，等等。

那么所谓的单元测试是什么？这里直接引用维基百科上的词条说明：

> 单元测试（又称为模块测试, Unit Testing）是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。程序单元是应用的最小可测试部件。在过程化编程中，一个单元就是单个程序、函数、过程等；对于面向对象编程，最小单元就是方法，包括基类（超类）、抽象类、或者派生类（子类）中的方法。

所以，我眼中的“合格的”单元测试需要满足几个条件：

1. 测试的是一个代码单元内部的逻辑，而不是各模块之间的交互。
2. 无依赖，不需要实际运行环境就可以测试代码。
3. 运行效率高，可以随时执行。

#### 2.1.2 单元测试的定位

了解了单元测试是什么之后，第二个问题就是：单元测试是用来做什么的？

很多人第一反应是“看看程序有没有问题”，或者“确保没有bug”。单元测试确实可以测试程序有没有问题，但是，从我个人编程的经验来看，大部分情况下只是使用单元测试来“看看程序有没有问题”的话，效率反而不如把程序运行起来直接查看结果。原因有两个：

1. 单元测试要写额外的代码，而不写单元测试，直接运行程序也可以测试程序有没有问题。
2. 即使通过了单元测试，程序在实际运行的时候仍然有可能出问题。

但是，很多时候直接启动程序测试会比较慢，所以一些同学为了解决这个问题，采用了一个折中的办法：只加载要测试的模块和它所有的依赖模块，比如在测试时只加载这个模块相关的spring的配置文件。这时所谓的单元测试实际上是用xUnit框架运行的集成测试，并没有体现“单元”的概念。

而关于“纯粹的单元测试”在介绍语言或者框架的书里很少被提起，反而是介绍重构或者敏捷开发的书里经常会看到各种各样的关于单元测试的介绍。

在这里我总结了一下几个比较常见的单元测试的几个典型场景：

1. 开发前写单元测试，通过测试描述需求，由测试驱动开发。
2. 在开发过程中及时得到反馈，提前发现问题。
3. 应用于自动化构建或持续集成流程，对每次代码修改做回归测试。
4. 作为重构的基础，验证重构是否可靠。

还有最重要的一点：编写单元测试的难易程度能够直接反应出代码的设计水平，能写出单元测试和写不出单元测试之间体现了编程能力上的巨大的鸿沟。无论是什么样的程序员，坚持编写一段时间的单元测试之后，都会明显感受到代码设计能力的巨大提升。

### 2.2 单元测试的痛点

对于新人来说，很容易在编写单元测试的时候遇到这几类问题：

#### 2.2.1 单元测试的资料不够全

这里不够全是相对于“编码”来说的。介绍如何编码、如何使用某个框架的书茫茫多，但是与编码同样重要的介绍单元测试的书却不多，翻来覆去好的也不多，并且都有一定年头了。（如果有这方面的好的资料，请推荐给我，多谢）

很多关于编程的书籍中并没有深入介绍如何进行单元测试，或者仅仅介绍了最基础的assert、jUnit里怎么定义一个测试函数之类，就没有然后了，给人的感觉是这样：


![book](./spock/book-actual.png)

#### 2.2.2 单元测试难以理解和维护

测试代码不像普通的应用程序一样有着很明确的作为“值”的输入和输出。举个例子，假如一个普通的函数要做下面这件事情：

- 接收一个user对象作为参数
- 调用dao层的update方法更新用户属性
- 返回true/false结果

那么，只需要在函数中声明一个参数、做一次调用、返回一个布尔值就可以了。但如果要对这个函数做一个“纯粹的”单元测试，那么它的输入和输出会有很多情况，比如其中一个测试是这样：

- 假设调用dao层的update方法会返回true。
- 程序去调用service层的update方法。
- 验证一下service是不是也返回了true。

无论是用什么样的单元测试框架，最后写出来的单元测试代码量也比业务代码只多不少，我在写代码过程中的经验值是：要在不作弊的情况下维持比较高的单元测试覆盖率，要有三倍于业务代码的单测代码。

更多的代码量，加上单测代码并不像业务代码那样直观，还有对单测代码可读性不重视的坏习惯，导致最终呈现出来的单测代码难以阅读，要维护更是难上加难。

同时，大部分单元测试的框架都有很强的代码侵入性。要理解单元测试，首先得学习他用的那个单元测试框架，这无形中又增加了单元测试理解和维护的难度。

#### 2.2.3 单元测试难以去除依赖

就像之前说的，如果要写一个纯粹的、无依赖的单元测试往往很困难，比如依赖了数据库、或者依赖了文件系统、再或者依赖了其它模块。

所以很多人在写单元测试时选择依赖一部分资源，比如在本机启动一个数据库。这类所谓的“单元测试”往往很流行，但是对于多人合作的项目，这类测试却经常容易造成混乱。

比如说要在本地读个文件，或者连接某个数据库，其他修改代码的人（或者持续集成系统中）并没有这些东西，所以测试也都没法通过。最后大部分这类测试代码的下场都是用不了、也舍不得删，只好被注释掉，扔在那里。

随着开源项目逐渐发展，对外部资源的依赖问题开始可以通过一些测试辅助工具解决，比如使用内存型数据库H2代替连接实际的测试数据库，不过能替代的资源类型始终有限。

而实际工作过程中，还有一类难以处理的依赖问题：代码依赖。比如一个对象的方法中调用了其它对象的方法，其它对象又调用了更多对象，最后形成了一个无比巨大的调用树。

很多比较旧的描述单元测试的书里写了一些传统的办法，这类方法基本上是先对耦合的部分做模拟，再对结果部分做断言。例如可以通过继承来自己做一个假的stub对象，最终用assert的方式验证正确性。但是这相当于对于每种假设都要做一个假的对象，而且对结果进行验证也比较复杂：比如我要验证“更新”操作是否真的调用了dao层，那么要自己在stub对象里对调用进行计数，验证时再对计数进行断言，非常繁琐。

后来出现了一些mock框架，比如java的JMockit、EasyMock，或者Mockito。利用这类框架可以相对比较轻松的通过mock方式去做假设和验证，相对于之前的方式有了质的飞跃，但是即使用上这类框架，遇到复杂的业务代码往往也无能为力。

而往往新人的代码质量往往不高，尤其是对代码的拆分和逻辑的抽象还处于懵懂阶段。要对这类代码写单测，即使是工作了3，4年的高级码农也是一个挑战，对新人来说几乎是不可能完成的任务。这也让很多新人有了“写单测很难”的感觉。

所以在这里需要强调一个观点，写单元测试的难易程度跟代码的质量关系最大，并且是决定性的。项目里无论用了哪个测试框架都不能解决代码本身难以测试的问题，所以如果你遇到的是“我的代码里依赖的东西太多了所以写不出来单测”这样的问题的话，需要去看的是如何设计和重构代码，而不是这篇文章。

### 2.3 推荐阅读

- 重构-改善既有代码的设计
- 修改代码的艺术
- 敏捷软件开发：原则、模式与实践

## 3 Spock是什么

### 3.1 简介

这里引用官方的介绍：

> Spock is a testing and specification framework for Java and Groovy applications. What makes it stand out from the crowd is its beautiful and highly expressive specification language. Thanks to its JUnit runner, Spock is compatible with most IDEs, build tools, and continuous integration servers. Spock is inspired from JUnit, jMock, RSpec, Groovy, Scala, Vulcans, and other fascinating life forms.

简单地说，spock是一个测试框架，它的核心特性有以下几个：

- 可以应用于java或groovy应用的单元测试框架。
- 测试代码使用基于groovy语言扩展而成的规范说明语言（specification language）。
- 通过junit runner调用测试，兼容绝大部分junit的运行场景（ide，构建工具，持续集成等）。
- 框架的设计思路参考了JUnit，jMock，RSpec，Groovy，Scala，Vulcans……

要理解spock的几个特性，还要理解几个关键名词：

#### 3.1.1 groovy

引用维基百科上的介绍：

> Groovy是Java平台上设计的面向对象编程语言。这门动态语言拥有类似Python、Ruby和Smalltalk中的一些特性，可以作为Java平台的脚本语言使用。
>
> Groovy的语法与Java非常相似，以至于多数的Java代码也是正确的Groovy代码。Groovy代码动态的被编译器转换成Java字节码。由于其运行在JVM上的特性，Groovy可以使用其他Java语言编写的库。

groovy是一门比较轻量，学习门槛也比较低的语言。对于只用过java语言的程序员来说，groovy是一个很不错的开拓视野的机会。如果你没有接触过groovy，那么可以参考这两条：

1. 可以用纯java的语法写groovy。
2. 参考这篇[快速入门](http://sysgears.com/articles/groovy-differences-java/)。

我个人比较喜欢groovy语言，在一些小项目中经常使用它。引用一下[R大在知乎的回复](http://www.zhihu.com/question/29818569)：

> Groovy比较讨好来自Java的程序员的一点是：用它写代码可以渐进的从接近Java的风格进化为接近Ruby的风格。使用接近Java风格写Groovy时，代码几乎跟Java一样，容易上手；而学习过程中可以逐渐用上各种类似Ruby的方便功能。

#### 3.1.2 specification language

如果接触过不同语言类型的开源项目的话，就会发现有些项目中找不到测试目录（test），取而代之的是一个叫“spec”的目录，比如用ruby写的项目[gitlab](https://github.com/gitlabhq/gitlabhq)。这里的spec实际是specification的缩写，它的背后是一种近些年来开始流行起来的编程思想：BDD（Behavior-driven development）。

关于BDD，同样是引用维基百科上的介绍：

> BDD：行为驱动开发是一种敏捷软件开发的技术，它鼓励软件项目中的开发者、QA和非技术人员或商业参与者之间的协作。BDD最初是由Dan North在2003年命名，它包括验收测试和客户测试驱动等的极限编程的实践，作为对测试驱动开发的回应。
>
> BDD的做法包括：
>
> - 确立不同利益相关者要实现的远景目标
> - 使用特性注入方法绘制出达到这些目标所需要的特性
> - 通过由外及内的软件开发方法，把涉及到的利益相关者融入到实现的过程中
> - 使用例子来描述应用程序的行为或代码的每个单元
> - 通过自动运行这些例子，提供快速反馈，进行回归测试
> - 使用“应当(should)”来描述软件的行为，以帮助阐明代码的职责，以及回答对该软件的功能性的质疑
> - 使用“确保(ensure)”来描述软件的职责，以把代码本身的效用与其他单元(element)代码带来的边际效用中区分出来。
> - 使用mock作为还未编写的相关代码模块的替身

BDD背后的编程思想超出了这篇文章的范围，这里就不再展开。上文说的specification language实际上是BDD其中一部分思想的实现手段：通过某种规范说明语言去描述程序“应该”做什么，再通过一个测试框架读取这些描述、并验证应用程序是否符合预期。

#### 3.1.3 单元测试的运行场景

测试只有被执行之后才会有价值，这里就涉及一个“什么时候执行单元测试”的问题。

1. 被接触最多的就是在IDE中执行单元测试，java程序员比较幸运，主流的java IDE都可以很好的集成了单元测试功能，单元测试代码自动生成、测试覆盖率检查等功能也都成了IDE的标配。这些功能都能让程序员在编写代码的时候直接可以运行单元测试得到反馈。

2. 其次，主流的构建工具（如maven、gradle）中也都实现了运行单元测试的功能，在生成二进制包之前可以对代码进行回归测试，这些构建工具都可以通过命令行调用，这是自动化构建的前提。

3. 在此之上，依托于构建工具提供的自动化特性，在持续集成、持续部署的过程中可以执行自动化构建，在自动化构建的过程中通过构建工具执行单元测试，这是持续集成的流程中的重要步骤。  


### 3.2 Spock与现有框架的对比

#### 3.2.1 已有的java单元测试框架

就像刚才说的，有很多已有的单元测试框架，稍微老一点的如JMockit、EasyMock，新一点的类似Mockito和PowerMock。我之前一直在用testng+Mockito作为主要的单元测试框架，用它写过大概上万行单元测试，它的写法相对来说比较易读，功能也能满足大多数场景。

但在使用mockito的过程中也总是有一些不是很方便的地方，比如代码的可读性总还是差那么一点，比如像这样：

```java
@Test
public void testIsUserEnabled_userStatusIsClosed_returnFalse() throws Exception {
    UserInfo userInfo = new UserInfo();
    userInfo.status = UserInfo.CLOSED;
    doReturn(userInfo).when(userDao).getUserInfo(anyLong());

    boolean isUserEnabled = userService.isUserEnabled(1l);

    Assert.assertFalse(isUserEnabled);
}
```

虽然能读懂，但是对于它所做的事情全来说感觉说了很多废话，单元测试代码总是里充斥着各种when(),anyXXX(),return()之类啰嗦的关键词，加上java本身就是一个啰嗦的强类型的语言，这让写单测和读单测成为了一种体力活。

其次是单测数据，大部分测试都要提供数据，比如“当输入a的时候应该返回b”，如果只有一组数据那么没什么问题，但是当需要测试很多边界条件，需要多组数据的时候就会比较纠结。

用jUnit或者testng的dataprovider可以实现这个需求，但是无论是通过xml定义还是通过函数返回数据，都非常不方便。

最后，因为这些框架都只是一些独立的函数，没有告诉你“应该怎么写单测”，所以不同的人最终写出来的单测也是五花八门：

- 有不用assert而是用system.out.println的
- 有单测一个函数写了好几百行的
- 有直接把单测当成main函数写的

最终，团队要接受“虽然确实写了单测，然而这并没有什么卵用”的结果。

#### 3.2.2 为什么使用spock

还是刚才的例子，如果用spock写的话：

```groovy
def "isUserEnabled should return true only if user status is enabled"() {
    given:
    UserInfo userInfo = new UserInfo(
            status: actualUserStatus
    );
    userDao.getUserInfo(_) >> userInfo;

    expect:
    userService.isUserEnabled(1l) == expectedEnabled;

    where:
    actualUserStatus   | expectedEnabled
    UserInfo.ENABLED   | true
    UserInfo.INIT      | false
    UserInfo.CLOSED    | false
}
```

这段代码实际是3个测试：当getUserInfo返回的用户状态分别为ENABLED、INIT和CLOSED时，验证各自isUserEnabled函数的返回是否符合期待。

我对于spock框架最直接的感受：

- spock框架使用标签分隔单元测试中不同的代码，更加规范，也符合实际写单元测试的思路
- 代码写起来更简洁、优雅、易于理解
- 由于使用groovy语言，所以也可以享受到脚本语言带来的便利
- 底层基于jUnit，不需要额外的运行框架
- 已经发布了1.0版本，基本没有比较严重的bug

#### 3.2.3 为什么不用spock

用了一段时间的spock后，我也总结了几个不用spock的理由：

- 框架相对比较新，IDE的支持（尤其是eclipse）不如其它成熟的框架
- groovy语言本身的compiler更新比较快，偶尔有坑（版本不兼容等）
- 需要了解groovy语言
- 与其它java的测试框架风格相差比较大，需要适应

当然，这些理由比起spock提供的易于开发和维护的单元测试代码来说，都是可以忽略的。

## 4 使用Spock

写到这里，还是要聚焦一下这篇文章要讨论的问题：如何用spock框架编写单元测试，在此之前再强调一下：

- 单元测试不一定非要使用spock，但是其它框架写出的单元测试代码远没有用spock框架优雅。
- spock框架并不只能写单元测试，它也可以写集成测试，甚至性能测试，但是后两者spock相对于其它框架来说没有什么优势。

### 4.1 关于开发环境

在使用spock框架时，我比较推荐的ide是IDEA，推荐的构建工具是gradle。

就算不使用spock框架，IDEA的顺手程度也比eclipse好太多，对新技术的响应速度快，也没有那么多莫名其妙的严重bug，社区版免费但主要功能都有，没有什么理由不试用一下。

而gradle相对于maven来说配置简化了很多，可定制的功能也更强，与其迷失在maven复杂的xml和一层套一层的依赖关系中，我宁愿把时间做一些更有意思的事情。

由于IDE基本可以自由选择，但构建工具大部分是由团队决定的，而maven现在还是处于构建工具的领导地位，所以这篇文章里的步骤都是基于IDEA+maven，当前的IDEA已经支持spock，不需要做什么特殊配置。

- 如果你的团队应用了gradle，spock官网中对于gradle如何配置说的比较完整，可以直接参考官网。

### 4.2 hello spock

前面做了那么多铺垫，终于到了真正编写一个hello world的时候。

到这里，我假设你是一位java开发者，并且已经了解基本的IDE及构建工具的使用。

1. 创建一个空白项目：hello\_spock，选择maven工程。
2. 在pom.xml中增加依赖：  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>hello</groupId>
    <artifactId>hello_spock</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <!-- Mandatory dependencies for using Spock -->
        <dependency>
             <groupId>org.spockframework</groupId>
            <artifactId>spock-core</artifactId>
            <version>1.3-groovy-2.5</version>
            <scope>test</scope>
        </dependency>
        <!-- Optional dependencies for using Spock -->
        <dependency> <!-- use a specific Groovy version rather than the one specified by spock-core -->
           <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy-all</artifactId>
            <version>2.5.3</version>
        </dependency>
        <dependency> <!-- enables mocking of classes (in addition to interfaces) -->
            <groupId>cglib</groupId>
            <artifactId>cglib-nodep</artifactId>
            <version>3.1</version>
            <scope>test</scope>
        </dependency>
        <dependency><!-- enables mocking of classes without default constructor (together with CGLIB) -->
            <groupId>org.objenesis</groupId>
            <artifactId>objenesis</artifactId>
            <version>2.1</version>
           <scope>test</scope>  
        </dependency>
    </dependencies>
</project>
```

3. 由于spock是基于groovy语言的，所以需要创建groovy的测试源码目录：首先在test目录下创建名为groovy的目录，之后将它设为测试源码目录。
4. 创建一个简单的类：  

```java
public class Sum {
    public int sum(int first, int second) {
        return first + second;
    }
}
```

5. 创建测试类，可以手工创建，也可以使用IDEA的辅助创建。

6. 编写测试代码，这里我们验证一下sum返回的结果是否正确：

```groovy
import spock.lang.Specification
class SumTest extends Specification {
    def sum = new Sum();
    def "sum should return param1+param2"() {
        expect:
        sum.sum(1,1) == 2
    }
}
```

7. 运行一下测试
至此，一个最简单的spock测试就写完了。

### 4.3 Spock中的概念

#### 4.3.1 Specification

在Spock中，待测系统(system under test; SUT) 的行为是由规格(specification) 所定义的。在使用Spock框架编写测试时，测试类需要继承自Specification类。

#### 4.3.2 Fields

Specification类中可以定义字段，这些字段在运行每个测试方法前会被重新初始化，跟放在setup()里是一个效果。

```groovy
def obj = new ClassUnderSpecification()
def coll = new Collaborator()
```

#### 4.3.3 Fixture Methods

预先定义的几个固定的函数，与junit或testng中类似，不多解释了

```groovy
def setup() {}          // run before every feature method
def cleanup() {}        // run after every feature method
def setupSpec() {}     // run before the first feature method
def cleanupSpec() {}   // run after the last feature method
```

#### 4.3.4 Feature methods

这是Spock规格(Specification）的核心，其描述了SUT应具备的各项行为。每个Specification都会包含一组相关的Feature methods，如要测试1+1是否等于2，可以编写一个函数：

```groovy
def "sum should return param1+param2"() {
    expect:
    sum.sum(1,1) == 2
}
```

#### 4.3.5 blocks

每个feature method又被划分为不同的block，不同的block处于测试执行的不同阶段，在测试运行时，各个block按照不同的顺序和规则被执行，如下图：

![blocks](http://blog.2baxb.me/wp-content/uploads/2015/08/Blocks2Phases.png)

下面分别解释一下各个block的用途。

#### 4.3.6 Setup Blocks

setup也可以写成given，在这个block中会放置与这个测试函数相关的初始化程序，如：

```groovy
setup:
def stack = new Stack()
def elem = "push me"
```

一般会在这个block中定义局部变量，定义mock函数等。

#### 4.3.7 When and Then Blocks

when与then需要搭配使用，在when中执行待测试的函数，在then中判断是否符合预期，如：

```groovy
when:
stack.push(elem)  

then:
!stack.empty
stack.size() == 1
stack.peek() == elem
```

##### 4.3.7.1 断言

条件类似junit中的assert，就像上面的例子，在then或expect中会默认assert所有返回值是boolean型的顶级语句。如果要在其它地方增加断言，需要显式增加assert关键字，如：

```groovy
def setup() {
  stack = new Stack()
  assert stack.empty
}
```

##### 4.3.7.2 异常断言

如果要验证有没有抛出异常，可以用thrown()，如下：

```groovy
when:
stack.pop()  

then:
thrown(EmptyStackException)
stack.empty
```

要获取抛出的异常对象，可以用以下语法：

```groovy
when:
stack.pop()  

then:
def e = thrown(EmptyStackException)
e.cause == null
```

如果要验证没有抛出某种异常，可以用notThrown()：

```groovy
when:
def x = Math.max(1, 2)  

then:
x == 2
```

#### 4.3.8 Expect Blocks

expect可以看做精简版的when+then，如：

```groovy
when:
def x = Math.max(1, 2)  

then:
x == 2
```

可以简化为：

```groovy
expect:
Math.max(1, 2) == 2
```

#### 4.3.9 Cleanup Blocks

函数退出前做一些清理工作，如关闭资源等。

#### 4.3.10 Where Blocks

做测试时最复杂的事情之一就是准备测试数据，尤其是要测试边界条件、测试异常分支等，这些都需要在测试之前规划好数据。但是传统的测试框架很难轻松的制造数据，要么依赖反复调用，要么用xml或者data provider函数之类难以理解和阅读的方式。比如说：

```
class MathSpec extends Specification {
    def "maximum of two numbers"() {
        expect:
        // exercise math method for a few different inputs
        Math.max(1, 3) == 3
        Math.max(7, 4) == 7
        Math.max(0, 0) == 0
    }
}
```

而在spock中，通过where block可以让这类需求实现起来变得非常优雅：

```groovy
class DataDriven extends Specification {
    def "maximum of two numbers"() {
        expect:
        Math.max(a, b) == c

        where:
        a | b || c
        3 | 5 || 5
        7 | 0 || 7
        0 | 0 || 0
    }
}
```

上述例子实际会跑三次测试，相当于在for循环中执行三次测试，a/b/c的值分别为3/5/5,7/0/7和0/0/0。如果在方法前声明@Unroll，则会当成三个方法运行。

更进一步，可以为标记@Unroll的方法声明动态的spec名：

```groovy
class DataDriven extends Specification {
    @Unroll
    def "maximum of #a and #b should be #c"() {
        expect:
        Math.max(a, b) == c

        where:
        a | b || c
        3 | 5 || 5
        7 | 0 || 7
        0 | 0 || 0
    }
}
```

运行时，名称会被替换为实际的参数值。

除此之外，where block还有两种数据定义的方法，并且可以结合使用，如：

```groovy
where:
a | _
3 | _
7 | _
0 | _

b << [5, 0, 0]

c = a > b ? a : b
```

### 4.4 Interaction Based Testing

对于测试来说，除了能够对输入-输出进行验证之外，还希望能验证模块与其他模块之间的交互是否正确，比如“是否正确调用了某个某个对象中的函数”；或者期望被调用的模块有某个返回值，等等。

各类mock框架让这类验证变得可行，而spock除了支持这类验证，并且做的更加优雅。如果你还不清楚mock是什么，最好先去简单了解一下，网上的资料非常多，这里就不展开了。

#### 4.4.1 mock

在spock中创建一个mock对象非常简单：

```groovy
class PublisherSpec extends Specification {
    Publisher publisher = new Publisher()
    Subscriber subscriber = Mock()
    Subscriber subscriber2 = Mock()

    def setup() {
        publisher.subscribers.add(subscriber)
        publisher.subscribers.add(subscriber2)
    }
}
```

而创建了mock对象之后就可以对它的交互做验证了：

```groovy
def "should send messages to all subscribers"() {
    when:
    publisher.send("hello")

    then:
    1 * subscriber.receive("hello")
    1 * subscriber2.receive("hello")
}
```

上面的例子里验证了：在publisher调用send时，两个subscriber都应该被调用一次receive(“hello”)。

示例中，表达式中的次数、对象、函数和参数部分都可以灵活定义：

```groovy
1 * subscriber.receive("hello")      // exactly one call
0 * subscriber.receive("hello")      // zero calls
(1..3) * subscriber.receive("hello") // between one and three calls (inclusive)
(1.._) * subscriber.receive("hello") // at least one call
(_..3) * subscriber.receive("hello") // at most three calls
_ * subscriber.receive("hello")      // any number of calls, including zero
1 * subscriber.receive("hello")     // an argument that is equal to the String "hello"
1 * subscriber.receive(!"hello")    // an argument that is unequal to the String "hello"
1 * subscriber.receive()            // the empty argument list (would never match in our example)
1 * subscriber.receive(_)           // any single argument (including null)
1 * subscriber.receive(*_)          // any argument list (including the empty argument list)
1 * subscriber.receive(!null)       // any non-null argument
1 * subscriber.receive(_ as String) // any non-null argument that is-a String
1 * subscriber.receive({ it.size() > 3 }) // an argument that satisfies the given predicate
                                          // (here: message length is greater than 3)
1 * subscriber._(*_)     // any method on subscriber, with any argument list
1 * subscriber._         // shortcut for and preferred over the above
1 * _._                  // any method call on any mock object
1 * _                    // shortcut for and preferred over the above
```

得益于groovy脚本语言的特性，在定义交互的时候不需要对每个参数指定类型，如果用过java下的其它mock框架应该会被这个特性深深的吸引住。

#### 4.4.2 Stubbing

对mock对象定义函数的返回值可以用如下方法：

```groovy
subscriber.receive(_) >> "ok"
```

> > 符号代表函数的返回值，执行上面的代码后，再调用subscriber.receice方法将返回ok。如果要每次调用返回不同结果，可以使用：

```groovy
subscriber.receive(_) >>> ["ok", "error", "error", "ok"]
```

如果要做额外的操作，如抛出异常，可以使用：

```groovy
subscriber.receive(_) >> { throw new InternalError("ouch") }
```

而如果要每次调用都有不同的结果，可以把多次的返回连接起来：

```groovy
subscriber.receive(_) >>> ["ok", "fail", "ok"] >> { throw new InternalError() } >> "ok"
```

### 4.5 mock and stubbing

如果既要判断某个mock对象的交互，又希望它返回值的话，可以结合mock和stub，可以这样：

```groovy
then:
1 * subscriber.receive("message1") >> "ok"
1 * subscriber.receive("message2") >> "fail"
```

注意，spock不支持两次分别设定调用和返回值，如果把上例写成这样是错的：

```groovy
setup:
subscriber.receive("message1") >> "ok"
when:
publisher.send("message1")
then: 1 * subscriber.receive("message1")
```

此时spock会对subscriber执行两次设定：

- 第一次设定receive(“message1”)只能调用一次，返回值为默认值（null）。
- 第二次设定receive(“message1”)会返回ok，不限制次数。

### 4.6 其它类型的mock对象

spock也支持spy,stub之类的mock对象，但是并不推荐使用。因为使用“正规的”bdd思路写出的代码不需要用这些方法来测试，官方的解释是：

> Think twice before using this feature. It might be better to change the design of the code under specification

具体的使用方法如果有兴趣可以参考官方文档。

### 4.7 更多

至此，读者应该对Spock的主要功能和使用方法应该有个粗略的认识。如果希望实际使用spock，推荐读一下官方的文档，写的比较清晰，并且其中引用的一些文档也都值得一读：

<http://spockframework.org/spock/docs/1.3/index.html>

另外一个值得一看的是spock-example工程：

<https://github.com/spockframework/spock-example>

## 5 结语

需要再强调一下：现实中的场景绝对会比文章中的例子复杂（比如要mock一个private函数，或者全局变量，或者静态函数，等等），但是此时更好的思路并不是压榨框架的功能，而应该是去思考代码的设计是否出了问题。

还是强调这个观点：单元测试的难度和代码设计的好坏息息相关，单元测试测的三分是代码，七分是设计。如果你觉得自己处于编码能力上升的瓶颈期，那么可以尝试一下为以前写的类编写“纯粹的”单元测试，在这个过程中，spock可以让你从重复的编码、繁重的维护工作中解脱出来，让编写测试回归为一件有幸福感和成就感的事情。
