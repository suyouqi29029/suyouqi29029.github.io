---
layout: post
title: 谈谈对MVC,MVP,MVVM设计模式的理解
date: 2019-10-21
author: su29029
cover: '/assets/img/MVC,MVP,MVVM设计模式/title.jpg'
tags: 软件工程
---

设计模式，emmmm，感觉很高深的样子，虽然感觉离我这个菜鸡还很遥远，不过了解一下还是没坏处。

## 0x01 关于MVC设计模式
MVC模式（Model–view–controller）是软件工程中的一种软件架构模式，把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）。

<img src='../assets/img/MVC,MVP,MVVM设计模式/MVC设计模式.jpg'>

MVC模式最早由Trygve Reenskaug在1978年提出，是施乐帕罗奥多研究中心在20世纪80年代为程序语言Smalltalk发明的一种软件架构。MVC模式的目的是实现一种动态的程序设计，使后续对程序的修改和扩展简化，并且使程序某一部分的重复利用成为可能。除此之外，此模式透过对复杂度的简化，使程序结构更加直观。软件系统透过对自身基本部分分离的同时也赋予了各个基本部分应有的功能。

MVC模式在概念上强调 Model, View, Controller 的分离，各个模块也遵循着由 Controller 来处理消息，Model 掌管数据源，View 负责数据显示的职责分离原则，因此在实现上，MVC 模式的 Framework 通常会将 MVC 三个部分分离实现：  

Model 负责数据访问，较现代的 Framework 都会建议使用独立的数据对象 (DTO, POCO, POJO 等) 来替代弱类型的集合对象。数据访问的代码会使用 Data Access 的代码或是 ORM-based Framework，也可以进一步使用 Repository Pattern 与 Unit of Works Pattern 来切割数据源的相依性。

Controller负责处理消息，较高端的 Framework 会有一个默认的实现来作为 Controller 的基础，例如 Spring 的 DispatcherServlet 或是 ASP.NET MVC 的 Controller 等，在职责分离原则的基础上，每个 Controller 负责的部分不同，因此会将各个 Controller 切割成不同的文件以利维护。

View负责显示数据，这个部分多为前端应用，而 Controller 会有一个机制将处理的结果 (可能是 Model, 集合或是状态等) 交给 View，然后由 View 来决定怎么显示。例如 Spring Framework 使用 JSP 或相应技术，ASP.NET MVC 则使用 Razor 处理数据的显示。

总的来讲，MVC是比较直观的架构模式，用户进行操作，View接收用户的输入操作，传递给Controller进行业务逻辑处理，Model实现数据持久化，并将结果反馈给View，完成一次MVC模式。

## 0x02 关于MVP设计模式
Model-View-Presenter (MVP) 是用户界面设计模式的一种，被广泛用于便捷自动化单元测试和在呈现逻辑中改良分离关注点。

Model 定义用户界面所需要被显示的数据模型，一个模型包含着相关的业务逻辑。

View 视图为呈现用户界面的终端，用以表现来自 Model 的数据，和用户命令路由再经过 Presenter 对事件处理后的数据。

Presenter 包含着组件的事件处理，负责检索 Model 获取数据，和将获取的数据经过格式转换与 View 进行沟通。

MVP 设计模式通常会再加上 Controller 做为整体应用程序的后端程序工作。

<img src='../assets/img/MVC,MVP,MVVM设计模式/MVP设计模式.jpg'>

MVP是把MVC中的Controller换成了Presenter（呈现），目的是为了完全切断View跟Model之间的联系，由Presenter充当桥梁，做到View-Model之间通信的完全隔离。

## 0x03 关于MVVM设计模式
MVVM（Model–view–viewmodel）是一种软件架构模式。

MVVM有助于将图形用户界面的开发与业务逻辑或后端逻辑（数据模型）的开发分离开来。MVVM的视图模型是一个值转换器， 这意味着视图模型负责从模型中暴露（转换）数据对象，以便轻松管理和呈现对象。在这方面，视图模型比视图做得更多，并且处理大部分视图的显示逻辑。视图模型可以实现中介者模式，组织对视图所支持的用例集的后端逻辑的访问。
<img src='../assets/img/MVC,MVP,MVVM设计模式/MVVM设计模式1.jpg'>

MVVM将“数据模型数据双向绑定”的思想作为核心，因此在View和Model之间没有联系，通过ViewModel进行交互，而且Model和ViewModel之间的交互是双向的，因此视图的数据的变化会同时修改数据源，而数据源数据的变化也会立即反应到View上。即，ViewModel 是一个 View 信息的存储结构，ViewModel 和 View 上的信息是一一映射关系。

 #### 使用MVVM模式的好处
低耦合。View可以独立于Model变化和修改，一个ViewModel可以绑定到不同的View上，当View变化的时候Model可以不变，当Model变化的时候View也可以不变。    
可重用性。可以把一些视图的逻辑放在ViewModel里面，让很多View重用这段视图逻辑。      
独立开发。开发人员可以专注与业务逻辑和数据的开发(ViewModel)。设计人员可以专注于界面(View)的设计。     
可测试性。可以针对ViewModel来对界面(View)进行测试   
使用 MVVM 模式，程序的 UI 和其背后的展现与业务逻辑将被分离至三个类中：    
1-视图，封装 UI 与 UI 逻辑    
2-模型视图，封装展示逻辑与状态    
3-模型，封装程序的业务逻辑以及数据    
在 MVVM 模式中，视图通过数据绑定以及命令行与视图模型交互，并改变事件通知。视图模型查询观察并协调模型更新，转换，校验以及聚合数据，从而在视图显示。   
下图展示了 MVVM 类以及它们之间的交互：
<img src='../assets/img/MVC,MVP,MVVM设计模式/MVVM设计模式2.jpg'>

## 0x04 总结
总的来讲，MVC，MVP，MVVM是历史不断进步发展的产物，随着业务需求的扩大，软件规模的提升，用户需求的提高，催生出了软件工程，催生出了这些设计模式。对每种设计模式，我们既应当看到它们的最大优点，也不能忽视它们存在的问题。例如，MVVM模式的数据双向绑定存在着Bug调试困难，大项目占用内存多等问题。对于设计模式，在不同的场合，应当具体问题具体分析，使用最适合项目开发的设计模式，而不能一味追求新技术或是守旧，我们需要保持的是一个拥抱变化的心，以及理性分析的态度。在新技术的面前，不盲从，不守旧，一切的决策都应该建立在认真分析的基础上，这样才能应对技术的变化。



