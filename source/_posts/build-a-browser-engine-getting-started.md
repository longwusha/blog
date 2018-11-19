---
title: 从头创建一个浏览器引擎-准备
date: 2018-11-19 14:51:11
tags: [Rust, Browser Engine]
---

**注：本文为译文，非常感谢[Let's build a browser engine](https://limpet.net/mbrubeck/2014/08/08/toy-layout-engine-1.html)系列博客**


我正在构建一个玩具HTML渲染引擎，我认为你也应该这样做。
这是系列文章中的第一篇：

- Part 1: 准备
- [Part 2: HTML解析器](https://limpet.net/mbrubeck/2014/08/11/toy-layout-engine-2.html)
- [Part 3: CSS解析器](https://limpet.net/mbrubeck/2014/08/13/toy-layout-engine-3-css.html)
- [Part 4: 样式](https://limpet.net/mbrubeck/2014/08/23/toy-layout-engine-4-style.html)
- [Part 5: Boxes](https://limpet.net/mbrubeck/2014/09/08/toy-layout-engine-5-boxes.html)
- [Part 6: Block layout](https://limpet.net/mbrubeck/2014/09/17/toy-layout-engine-6-block.html)
- [Part 7: Painting 101](https://limpet.net/mbrubeck/2014/11/05/toy-layout-engine-7-painting.html)

在这整个系列里，我会讲解我编写的代码，并教大家如何自己做到这一切。首先，让我先解释一些疑问。


## 你在创建一个什么样的东西

**浏览器引擎**是运行在Web浏览器底层的一个组成部分，它从互联网上获取网页，将内容内容翻译成可以阅读、观看、收听的表单。

Blink、Gecko、WebKit和Trident是浏览器引擎。在此之上，浏览器包含自己的UI：选项卡，工具栏，菜单等。Firefox和SeaMonkey是两个具有不同的UI的浏览器，但它们都使用Gecko引擎。

浏览器引擎包括许多子组件：HTTP客户端、HTML解析器、CSS解析器、JavaScript引擎（本身由解析器，解释器和编译器组成）等等。 解析Web内容（如HTML和CSS）并将它们转换为您在屏幕上看到的内容所涉及的组件，有时称为布局引擎或渲染引擎。

## 为什么是一个“玩具”渲染引擎
功能齐全的浏览器引擎非常复杂。Blink，Gecko，WebKit--它们每个都有数百万行代码。
即使是更年轻、更简单的渲染引擎，如[Servo](https://github.com/servo/servo/)和[WeasyPrint](https://weasyprint.org/)，也有数万行代码。对于新手来说，理解这些代码并不是一件简单的事情！

说到非常复杂的软件，如果你在上关于编译器或操作系统的课，在某些时候你可能会创建或修改“玩具”编译器或内核。这是一个专为学习而设计的简单模型。除了编写它的人之外，它可能永远不会被任何人操作。
制作玩具系统是学习真实物品如何运作很有用的一个方法。即使您从未构建真实的编译器或内核，了解它们的工作方式也可以帮助您在编写自己的程序时更好地使用它们。

那么，如果你想成为一名浏览器开发者，或者只是想了解浏览器引擎内部发生了什么，为什么不建立一个玩具呢？
就像实现“真实”编程语言子集的玩具编译器一样，玩具渲染引擎可以实现HTML和CSS的一小部分。
它不会取代我们日常使用的浏览器中的引擎，但应该说明渲染简单HTML文档所需的基本步骤。

## 在家尝试一下
我希望说服你试一试。如果您已经拥有一些可靠的编程经验并且了解一些高级HTML和CSS概念，那么本系列将是最容易理解的。但是，如果你刚开始使用这些东西，或遇到你不理解的事情，请随意提问，我会尽量向大家解释清楚。

在开始之前，您可以先对一些选择做出你自己的评论：

### 编程语言
您可以使用任何你熟悉和喜爱的编程语言构建玩具布局引擎。以此为借口学习一门新语言，这听起来同样很有趣。

如果您想开始为Gecko或WebKit等主要浏览器引擎做贡献，您可能希望使用C++。因为它是这些引擎中使用的主要语言，使用它可以更容易地将代码与他们的代码进行比较。

我自己的玩具项目[robinson](https://github.com/mbrubeck/robinson)是用Rust编写的。我是Mozilla的Servo团队的一员，所以我非常喜欢Rust编程。此外，我在这个项目中的目标之一是更多地了解Servo的实现。Robinson有时会使用简化版本的Servo数据结构和代码。

### 库与快捷方式
在这样的学习练习中，你必须决定是否“作弊”（使用别人的代码而不是从头开始编写自己的代码）。我的建议是为你真正想要理解的部分编写自己的代码，但不要害怕将库用于其它部分。学习如何使用特定的库本身就是一项有价值的练习。

我正在写robinson，不仅仅是为了我自己，还要作为这些文章和练习的示例代码。由于这个原因和一些其他原因，我希望它尽可能地小而且独立。到目前为止，除了Rust标准库之外，我没有使用任何外部代码。但这条规则并非一成不变。例如，我可能稍后决定使用图形库而不是编写我自己的低级绘图代码。

避免编写太多的代码的另一种方法是先跳过一些事情。例如，robinson还没有网络代码，它只能读取本地文件。在玩具程序中，如果您愿意，可以跳过一些东西。我会指出这样的潜在快捷方式，所以你可以绕过不感兴趣的步骤，直接跳到关注的部分。如果你改变主意，你可以随时填补空白。

## 第一步：DOM
你准备好了写一些代码吗？我们将从一些小事做起：[DOM](http://dom.spec.whatwg.org/)的数据结构。让我们来看看Robinson的[dom模块](https://github.com/mbrubeck/robinson/blob/master/src/dom.rs)。

DOM是节点树，节点具有零个或多个子节点。它还有其他各种属性和方法，但我们暂时可以忽略其中的大部分属性和方法。

``` rust
struct Node {
    // data common to all nodes:
    children: Vec<Node>,

    // data specific to each node type:
    node_type: NodeType,
}
```

[节点类型](http://dom.spec.whatwg.org/#dom-node-nodetype)其实很多样，但是现在我们将忽略大多数节点类型，认为一个节点是Element或Text节点。
在具有继承的语言中，这些将是Node的子类型。在Rust中，它们可以是枚举（在Rust中用于“标记联合”或“求和类型”，关键字为enum）：

``` rust
enum NodeType {
    Text(String),
    Element(ElementData),
}
```

元素包括标签名称和任意数量的属性，这些属性可以存储为从名称到值的映射。Robinson不支持名称空间，因此它只将标签和属性名称存储为简单字符串。

``` rust
struct ElementData {
    tag_name: String,
    attributes: AttrMap,
}

type AttrMap = HashMap<String, String>;
```

最后，我们可以使用构造函数轻松的创建新节点：

``` rust
fn text(data: String) -> Node {
    Node { children: Vec::new(), node_type: NodeType::Text(data) }
}

fn elem(name: String, attrs: AttrMap, children: Vec<Node>) -> Node {
    Node {
        children: children,
        node_type: NodeType::Element(ElementData {
            tag_name: name,
            attributes: attrs,
        })
    }
}
```

一个完整的DOM实现将包括更多的数据和许多方法，但这些代码是我们开始的时候需要的全部内容。

## 练习
给您一个建议：**做你感兴趣的练习，跳过其它的练习**。

1. 以您选择的语言启动一个新程序，并编写代码来表示DOM文本节点和元素的树。

2. 安装最新版本的[Rust](https://www.rust-lang.org)，然后下载并构建[robinson](https://github.com/mbrubeck/robinson)。
   打开dom.rs并扩展NodeType以包含注释节点等其他类型。

3. 编写代码以更漂亮地打印DOM节点树。

在下一篇文章中，我们将添加一个解析器，将HTML源代码转换为这些DOM节点的树。

## 引用
有关浏览器引擎内部的更多详细信息，请参阅Tali Garsiel精彩的[How Browsers Work](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)。

作为示例代码，这里是“小”开源Web渲染引擎的简短列表。他们中的大多数比robinson大很多倍，但仍然比Gecko或WebKit小。WebWhir，2000行代码，是另一个唯一被我称之为“玩具”的引擎。

- [CSSBox](https://github.com/philborlin/CSSBox)(Java)
- [Cocktail](https://github.com/silexlabs/Cocktail)(Haxe)
- [gngr](https://gngr.info/)(Java)
- [litehtml](https://github.com/tordex/litehtml)(C++)
- [LURE](https://github.com/admin36/LURE)(Lua)
- [NetSurf](http://www.netsurf-browser.org/)(C)
- [Servo](https://github.com/servo/servo/)(Rust)
- [Simple San Simon](http://hsbrowser.wordpress.com/3s-functional-web-browser/)(Haskell)
- [WeasyPrint](https://github.com/Kozea/WeasyPrint)(Python)
- [WebWhirr](https://github.com/reesmichael1/WebWhirr)(C++)

您可能会发现这些项目对于灵感或参考有用。如果你知道任何其他类似的项目，或者你有自己的项目，请告诉我！
