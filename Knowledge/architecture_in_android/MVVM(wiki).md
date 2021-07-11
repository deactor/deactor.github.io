---
title:  "MVVM wiki介绍"
---

## Model–view–viewmodel
Model–view–viewmodel (MVVM) is a software architectural pattern that facilitates the separation of the development of the graphical user interface (the view) – be it via a markup language or GUI code – from the development of the business logic or back-end logic (the model) so that the view is not dependent on any specific model platform. The viewmodel of MVVM is a value converter, meaning the viewmodel is responsible for exposing (converting) the data objects from the model in such a way that objects are easily managed and presented. In this respect, the viewmodel is more model than view, and handles most if not all of the view's display logic.The view model may implement a mediator pattern, organizing access to the back-end logic around the set of use cases supported by the view.  
>模型-视图-视图模型 (MVVM) 是一种软件架构模式，它有助于将图形用户界面（视图）的开发（无论是通过标记语言还是 GUI 代码）与业务逻辑的开发或后台分离，以便视图不依赖于任何特定的模型平台。 MVVM 的viewmodel是一个值转换器， 意味着viewmodel负责以易于管理和呈现对象的方式公开（转换）model中的数据对象。在这方面，viewmodel 中modle比view多，并且处理大部分（如果不是全部）视图的显示逻辑。 
viewmodel可以实现中介模式，围绕视图支持的用例集组织对后端逻辑的访问。

MVVM is a variation of Martin Fowler's Presentation Model design pattern. It was invented by Microsoft architects Ken Cooper and Ted Peters specifically to simplify event-driven programming of user interfaces. The pattern was incorporated into Windows Presentation Foundation (WPF) (Microsoft's .NET graphics system) and Silverlight (WPF's Internet application derivative). John Gossman, one of Microsoft's WPF and Silverlight architects, announced MVVM on his blog in 2005.
>MVVM 是 Martin Fowler 的演示模型设计模式的变体。 它由 Microsoft 架构师 Ken Cooper 和 Ted Peters 发明，专门用于简化用户界面的**事件驱动**编程。该模式被合并到 Windows Presentation Foundation (WPF)（Microsoft 的 .NET 图形系统）和 Silverlight（WPF 的 Internet 应用程序衍生物）中。微软 WPF 和 Silverlight 架构师之一的 John Gossman 于 2005 年在他的博客上宣布了 MVVM。

Model–view–viewmodel is also referred to as model–view–binder, especially in implementations not involving the .NET platform. ZK (a web application framework written in Java) and KnockoutJS (a JavaScript library) use model–view–binder.
>Model–view–viewmodel也称为model–view–binder，尤其是在不涉及 .NET 平台的实现中。 ZK（一个用 Java 编写的 Web 应用程序框架）和 KnockoutJS（一个 JavaScript 库）使用模型-视图-绑定器。


## Components of MVVM pattern
### Model
Model refers either to a domain model, which represents real state content (an object-oriented approach), or to the data access layer, which represents content (a data-centric approach).
>Model是指代表真实状态内容的领域模型（一种面向对象的方法），或者指代表内容的数据访问层（一种以数据为中心的方法）。 

### View
As in the model–view–controller (MVC) and model–view–presenter (MVP) patterns, the view is the structure, layout, and appearance of what a user sees on the screen. It displays a representation of the model and receives the user's interaction with the view (mouse clicks, keyboard input, screen tap gestures, etc.), and it forwards the handling of these to the view model via the data binding (properties, event callbacks, etc.) that is defined to link the view and view model.
>正如在模型-视图-控制器 (MVC) 和模型-视图-展示器 (MVP) 模式中一样，视图是用户在屏幕上看到的内容的结构、布局和外观。 它显示模型的表示并接收用户与视图的交互（鼠标点击、键盘输入、屏幕点击手势等），并通过数据绑定（属性、事件回调）将这些处理转发给视图模型等），它被定义为链接视图和视图模型。

### View model
The view model is an abstraction of the view exposing public properties and commands. Instead of the controller of the MVC pattern, or the presenter of the MVP pattern, MVVM has a binder, which automates communication between the view and its bound properties in the view model. The view model has been described as a state of the data in the model.
The main difference between the view model and the Presenter in the MVP pattern is that the presenter has a reference to a view, whereas the view model does not. Instead, a view directly binds to properties on the view model to send and receive updates. To function efficiently, this requires a binding technology or generating boilerplate code to do the binding.
>View model是视图的抽象，公开了公共属性和命令。 MVVM 不是 MVC 模式的控制器，也不是 MVP 模式的展示者，而是有一个**绑定器**，它可以在视图模型中自动进行视图及其绑定属性之间的通信。视图模型已被描述为模型中数据的状态。
MVP 模式中viewmodel和presenter之间的主要区别在于presenter具有对view的引用，而viewmodel没有。相反，view直接绑定到viewmodel上的属性以发送和接收更新。为了有效地运行，这需要绑定技术或生成样板代码来进行绑定。

### Binder
Declarative data and command-binding are implicit in the MVVM pattern. In the Microsoft solution stack, the binder is a markup language called XAML. The binder frees the developer from being obliged to write boiler-plate logic to synchronize the view model and view. When implemented outside of the Microsoft stack, the presence of a declarative data binding technology is what makes this pattern possible, and without a binder, one would typically use MVP or MVC instead and have to write more boilerplate (or generate it with some other tool).
>声明性数据和命令绑定在 MVVM 模式中是隐含的。在 Microsoft 解决方案堆栈中，活页夹是一种称为 XAML 的标记语言。 绑定器使开发人员不必编写样板逻辑来同步视图模型和视图。当在 Microsoft 堆栈之外实现时，声明性数据绑定技术的存在使这种模式成为可能， 如果没有绑定器，人们通常会改用 MVP 或 MVC，并且必须编写更多样板（或使用其他工具生成它）。

## Rationale
MVVM was designed to remove virtually all GUI code ("code-behind") from the view layer, by using data binding functions in WPF (Windows Presentation Foundation) to better facilitate the separation of view layer development from the rest of the pattern. Instead of requiring user experience (UX) developers to write GUI code, they can use the framework markup language (e.g., XAML) and create data bindings to the view model, which is written and maintained by application developers. The separation of roles allows interactive designers to focus on UX needs rather than programming of business logic. The layers of an application can thus be developed in multiple work streams for higher productivity. Even when a single developer works on the entire code base, a proper separation of the view from the model is more productive, as the user interface typically changes frequently and late in the development cycle based on end-user feedback.[citation needed]
>MVVM 旨在通过使用 WPF（Windows Presentation Foundation）中的数据绑定函数从视图层中删除几乎所有 GUI 代码（“代码隐藏”），以更好地促进视图层开发与模式其余部分的分离。 3]无需用户体验 (UX) 开发人员编写 GUI 代码，他们可以使用框架标记语言（例如 XAML）并创建数据绑定到由应用程序开发人员编写和维护的视图模型。角色的分离使交互设计师能够专注于 UX 需求而不是业务逻辑的编程。因此，可以在多个工作流中开发应用程序的层，以提高生产力。即使一个开发人员在整个代码库上工作，将视图与模型适当分离也会更有效率，因为用户界面通常会根据最终用户的反馈在开发周期的后期频繁更改。

The MVVM pattern attempts to gain both advantages of separation of functional development provided by MVC, while leveraging the advantages of data bindings and the framework by binding data as close to the pure application model as possible.[clarification needed] It uses the binder, view model, and any business layers' data-checking features to validate incoming data. The result is that the model and framework drive as much of the operations as possible, eliminating or minimizing application logic which directly manipulates the view (e.g., code-behind).
>MVVM 模式试图同时获得 MVC 提供的功能开发分离的优点，同时通过将数据绑定到尽可能接近纯应用程序模型的方式来利用数据绑定和框架的优点。 [需要澄清] 它使用绑定器、视图模型和任何业务层的数据检查功能来验证传入数据。结果是模型和框架驱动尽可能多的操作，消除或最小化直接操作视图的应用程序逻辑（例如，代码隐藏）。

## Criticism
John Gossman and Vincent.L has criticized the MVVM pattern and its application in specific uses, stating that MVVM can be "overkill" when creating simple user interfaces. For larger applications, he believes that generalizing the viewmodel upfront can be difficult, and that large-scale data binding can lead to lower performance.
>John Gossman 和 Vincent.L 批评了 MVVM 模式及其在特定用途中的应用，指出在创建简单的用户界面时，MVVM 可能是“矫枉过正”。对于较大的应用程序，他认为预先泛化视图模型可能很困难，并且大规模数据绑定会导致性能降低。 