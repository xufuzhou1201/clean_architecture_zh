# 第二十二章 整洁的架构

在过去的几十年中，我们已经看到了关于系统架构的一系列想法。他们包括：

* 六角架构（也称为端口和适配器），由Alistair Cockburn开发，并在由Steve Freeman和Nat Pryce在他们的好书“用测试开发面向对象软件”[^1]中采编。
* DCI，来自James Coplien和Trygve Reenskaug
* BCE，介绍由Ivar Jacobson撰写的书：“面向对象的软件工程：一种用例驱动的方法”[^2]

尽管这些架构在细节上都有所不同，但它们非常相似。他们都有相同的目标，这是关注点的分离。他们都通过将软件分成几层来实现这种分离。每个业务规则至少有一个层，用户和系统接口至少有一个层。

这些体系结构中的每一个产生的系统具有以下特征：

* 独立于框架。这个架构并不依赖于一些功能强大的软件库的存在。这允许你使用这样的框架作为工具，而不是强迫你将系统塞进它有限的约束中。
* 可测试。业务规则可以在没有UI，数据库，Web服务器或任何其他外部元素的情况下进行测试。 
* 独立于用户界面。用户界面可以很容易地更改，而无需更改系统的其余部分。例如，Web UI可以替换为控制台UI，而无需更改业务规则。 
* 独立于数据库。您可以换出Oracle或SQL Server的Mongo，BigTable，CouchDB，或其他。你的业​​务规则不绑定到数据库。

* 独立于任何外部机构。事实上，你的商业规则根本不了解与外界的接口。

图22.1中的图表试图将所有这些架构整合为一个可行的想法。

![](/assets/22/Figure_22.1_The_clean_architecture.png)

图22.1 整洁的架构

## 依赖规则（the Dependency Rule）

图22.1中的同心圆表示软件的不同区域。一般来说，越深入圆心，软件层级就越高。外圈是机制。内圈是策略。

使此架构运作的重要规则是依赖规则：

> 源代码依赖关系必须仅指向内部，朝向更高层级的策略。

内圈没有任何东西对外圈的事情有所了解。特别是外圈的声明的名字不能在内圈的代码中提及。这包括函数，类，变量或任何其他指定的软件实体。

同样的道理，在外部圈子中声明的数据格式不应该被内部圈子使用，特别是如果这些格式是由外部圈子中的框架产生的话。我们不希望外圈的任何东西影响内圈。

### 实体

实体封装了企业级（enterprise-wide）的关键业务规则。一个实体可以是一个具有方法的对象，也可以是一组数据结构和函数。是哪个并不重要，只要这些实体可以被企业中的许多不同的应用程序使用。

如果你没有一个企业，而且只写一个应用程序，那么这些实体就是应用程序的业务对象。它们囊括了最一般和最高级的规则。当发生外部的变化时，他们是最不可能改变的。例如，你不希望这些对象受到页面导航或安全性更改的影响。任何特定应用程序的操作改变都不会影响实体层。

### 用例

用例层中的软件包含特定于应用程序的业务规则。它封装并实现了系统的所有用例。这些用例协调实体之间的数据流，并指导这些实体使用其关键业务规则来实现用例的目标。

我们不希望这一层的变化影响实体。我们也不希望这个层受到数据库，UI或任何通用框架等外部性的影响。用例层与这些问题是隔离的。

但是，我们确实希望对应用程序操作的更改会影响用例，从而影响此层中的软件。如果一个用例的细节发生了变化，那么这个层的一些代码肯定会受到影响。

### 接口适配器

接口适配器层中的软件是一组适配器，可以将数据从用例和实体最方便的格式转换为最适合某些外部机构（如数据库或Web）的格式。例如，这个层将完全包含GUI的MVC体系结构。展示器（presenters），视图（views）和控制器（controllers）都属于接口适配器层。模型可能只是从控制器传递给用例的数据结构，然后从用例返回到展示器和视图。

类似地，在这个层中，数据从最适合于实体和用例的形式转换为最适合于正在使用的任何持久性框架（即，数据库）的形式。这个圈子里面的任何代码都不应该对数据库有所了解。如果数据库是一个SQL数据库，那么所有的SQL都应该被限制在这个层上——特别是这个层中与数据库有关的部分。

在这个层中，还有其他必要的适配器，它们将数据从外部服务等外部形式的东西转换为用例和实体所使用的内部形式的东西。

### 框架和驱动程序

图22.1中模型的最外层通常由框架和工具（如数据库和Web框架）组成。一般来说，你不要在这个层上编写太多的代码，除了组织代码向内传递到下一个圆。

框架和驱动程序层是所有细节所在的地方。网络是一个细节。数据库是一个细节。我们把这些东西放在外面，不会有什么伤害。

### 只有四个圈子？

图22.1中的圆圈意图是示意图：你可能会发现，你不仅需要这四个。没有规定说你必须只有这四个。但是，依赖规则始终适用。源代码依赖关系始终指向内部。当你向内移动时，抽象和策略的层级会提高。最外面的圆圈由低层级的具体细节组成。随着你向内移动，软件变得越来越抽象，并且封装了更高层级的策略。最内层的圈子是最一般化最高的层级。

### 跨越边界

图22.1中的图右下方是我们如何跨越圆边界的一个例子。它显示了控制器和展示器与下一层中的用例进行通信。注意控制的流向：它从控制器开始，在用例中移动，然后在展示器中执行。还要注意源代码的依赖关系：每个都指向用例。

我们通常通过使用依赖倒置原则来解决这个明显的矛盾。例如，在像Java这样的语言中，我们将使用接口和继承关系，以便源代码依赖性在跨越边界的恰当点处颠倒控制流。

例如，假设用例需要调用展示器。这个调用不能是直接的，因为这会违反依赖规则：外层圈子中应该没有任何名字被内层圈子提及。所以我们有一个用例调用一个接口（如图22.1所示为“用例输出端口”），并在外层有个展示器来实现它。

同样的技术被用来跨越架构中的所有边界。我们利用动态多态来创建颠倒控制流的源代码依赖关系，以便我们可以遵循依赖关系规则，无论控制流在哪个方向传播。

### 哪种数据跨越边界

通常跨越边界的数据由简单的数据结构组成。如果你喜欢，你可以使用基本的结构或简单的数据传输对象。或者数据可以简单地作为函数调用的参数。或者你可以把它打包成一个hashmap，或者把它构造成一个对象。重要的是隔离的，简单的数据结构跨越边界传递。我们不想欺骗和传递实体对象或数据库行。我们不希望数据结构有任何违背依赖规则的依赖。

例如，许多数据库框架响应查询返回一个方便的数据格式。我们可以称之为“行结构”。我们不希望将行结构向内跨越边界。这样做会违反依赖规则，因为这会迫使内圈知道外圈的一些事情。

因此，当我们通过边界传递数据时，它总是以内圈最方便的形式出现。

## 一例典型场景

图22.2中的图显示了使用数据库的基于Web的Java系统的典型场景。Web服务器从用户那里收集输入数据，并将其交给左上角的Controller。Controller将这些数据打包成一个普通的旧Java对象，并通过InputBoundary将这个对象传递给UseCaseInteractor。UseCaseInteractor解释这些数据并使用它来控制实体的动向。它还使用DataAccessInterface将这些实体使用的数据从数据库中带入内存。完成后，UseCaseInteractor从实体收集数据，并将OutputData构造为另一个普通的旧Java对象。OutputData然后通过OutputBoundary接口传递给Presenter。

![](/assets/22/Figure_22.2_A_typical_scenario_for_a_web-based_Java_system_utilizing_a_database.png)

图22.2 使用数据库的基于Web的Java系统的典型场景

Presenter的工作是将OutputData重新打包为ViewModel，这是另一个普通的旧Java对象。ViewModel主要包含视图用于显示数据的Strings和标志。而OutputData可能包含Date对象，则Presenter将加载ViewModel，其中相应的Strings已经为用户正确格式化。货币对象或任何其他业务相关数据也是如此。按钮和MenuItem名称放置在ViewModel中，正如标志告诉View是否这些Buttons和MenuItems应该是灰色的。

除了将ViewModel中的数据移动到HTML页面之外，这几乎不需要做任何事情。

注意依赖的方向。根据依赖关系规则，所有依赖关系跨越指向内部的边界线。

## 小结

遵守这些简单的规则并不困难，而且会为你节省很多麻烦。通过将软件分为多个层级并符合依赖规则，你将创建一个本质上可测试的系统，并具有它蕴藏的所有的好处。当系统的任何外部部件过时（比如数据库或Web框架）时，可以用最少的代价取代那些过时的元素。

[^1]: Growing Object Oriented Software with Tests

[^2]: Object Oriented Software Engineering: A Use-Case Driven Approach
