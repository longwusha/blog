---
title: 从头创建一个浏览器引擎-准备
date: 2018-11-19 14:51:11
tags: [Rust, Browser Engine]
---

**注：本文为译文，非常感谢[Let's build a browser engine](https://limpet.net/mbrubeck/2014/08/08/toy-layout-engine-1.html)系列博客**


我正在构建一个玩具HTML渲染引擎，我认为你也应该这样做。 这是一系列文章中的第一篇：

- Part 1: Getting started
- Part 2: HTML
- Part 3: CSS
- Part 4: Style
- Part 5: Boxes
- Part 6: Block layout
- Part 7: Painting 101

完整系列将描述我编写的代码，并展示如何制作自己的代码。 但首先，让我解释一下原因。

## 你在建一个什么？
我们来谈谈术语。 浏览器引擎是网络浏览器的一部分，它可以“在引擎盖下”从互联网上获取网页，并将其内容翻译成可以阅读，观看，收听等形式的表单.Blink，Gecko，WebKit和Trident 是浏览器引擎。 相比之下，浏览器自己的UI选项卡，工具栏，菜单等称为chrome。 Firefox和SeaMonkey是两个浏览器，具有不同的chrome但具有相同的Gecko引擎。

浏览器引擎包括许多子组件：HTTP客户端，HTML解析器，CSS解析器，JavaScript引擎（本身由解析器，解释器和编译器组成）等等。 解析Web格式（如HTML和CSS）以及将它们转换为您在屏幕上看到的内容所涉及的组件有时称为布局引擎或渲染引擎。

## 为什么一个“玩具”渲染引擎？
功能齐全的浏览器引擎非常复杂。 Blink，Gecko，WebKit--每个都有数百万行代码。 即使是更年轻，更简单的渲染引擎，如Servo和WeasyPrint，也都是数万行。 对于新手来说，理解并不是最简单的事情！

说到非常复杂的软件：如果你在编译器或操作系统上上课，在某些时候你可能会创建或修改“玩具”编译器或内核。 这是一个专为学习而设计的简单模型; 除了编写它的人之外，它可能永远不会被任何人操作。 但制作玩具系统是学习真实物品如何运作的有用工具。 即使您从未构建真实的编译器或内核，了解它们的工作方式也可以帮助您在编写自己的程序时更好地使用它们。

那么，如果你想成为一名浏览器开发者，或者只是想了解浏览器引擎内部发生了什么，为什么不建立一个玩具呢？ 就像实现“真实”编程语言子集的玩具编译器一样，玩具渲染引擎可以实现HTML和CSS的一小部分。 它不会取代日常浏览器中的引擎，但应该说明渲染简单HTML文档所需的基本步骤。

## 在家尝试一下。
我希望我说服你试一试。 如果您已经拥有一些可靠的编程经验并且了解一些高级HTML和CSS概念，那么本系列将是最容易理解的。 但是，如果你刚开始使用这些东西，或遇到你不理解的事情，请随意提问，我会尽量让它更清楚。

在开始之前，您可以对一些选择做一些评论：

### 编程语言
您可以使用任何编程语言构建玩具布局引擎。 真！ 来吧，使用你熟悉和喜爱的语言。 或者以此为借口学习一门新语言，如果这听起来很有趣。

如果您想开始为Gecko或WebKit等主要浏览器引擎做贡献，您可能希望使用C ++，因为它是这些引擎中使用的主要语言，使用它可以更容易地将代码与他们的代码进行比较。

我自己的玩具项目robinson是用Rust编写的。 我是Mozilla的Servo团队的一员，所以我非常喜欢Rust编程。 此外，我在这个项目中的目标之一是更多地了解Servo的实施。 Robinson有时会使用Servo数据结构和代码的简化版本。

### 库与工具
在这样的学习练习中，你必须决定是否“欺骗”使用别人的代码而不是从头开始编写自己的代码。 我的建议是为你真正想要理解的部分编写自己的代码，但不要害怕将库用于其他所有部分。 学习如何使用特定的库本身就是一项有价值的练习。

我正在写robinson，不仅仅是为了我自己，还要作为这些文章和练习的示例代码。 由于这个原因和其他原因，我希望它尽可能地小而且独立。 到目前为止，除了Rust标准库之外，我没有使用任何外部代码。 （这也解决了在使用相同版本的Rust时，使用相同版本的Rust构建多个依赖项的轻微麻烦，但语言仍在开发中。）但这条规则并非一成不变。 例如，我可能稍后决定使用图形库而不是编写我自己的低级绘图代码。

避免编写代码的另一种方法是将事情遗漏。 例如，robinson还没有网络代码; 它只能读取本地文件。 在玩具程序中，如果您愿意，可以跳过一些东西。 我会指出这样的潜在快捷方式，所以你可以绕过不感兴趣的步骤，直接跳到好东西。 如果你改变主意，你可以随时填补空白。

## 第一步：DOM
你准备好写一些代码吗？ 我们将从一些小事做起：DOM的数据结构。 让我们来看看罗宾逊的dom模块。

DOM是节点树。 节点具有零个或多个子节点。 （它还有其他各种属性和方法，但我们暂时可以忽略其中的大部分属性和方法。）

``` rust
struct Node {
    // data common to all nodes:
    children: Vec<Node>,

    // data specific to each node type:
    node_type: NodeType,
}
```

有几种节点类型，但是现在我们将忽略大多数节点类型，并说节点是Element或Text节点。 在具有继承的语言中，这些将是Node的子类型。 在Rust中，它们可以是枚举（Rust的关键字用于“标记联合”或“求和类型”）：

``` rust
enum NodeType {
    Text(String),
    Element(ElementData),
}
```

元素包括标记名称和任意数量的属性，这些属性可以存储为从名称到值的映射。 Robinson不支持名称空间，因此它只将标记和属性名称存储为简单字符串。

``` rust
struct ElementData {
    tag_name: String,
    attributes: AttrMap,
}

type AttrMap = HashMap<String, String>;
```

最后，一些构造函数可以轻松创建新节点：

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

就是这样！ 一个完整的DOM实现将包括更多的数据和许多方法，但这是我们开始需要的全部内容。

## 练习
这些只是在家中遵循的一些建议方法。 做你感兴趣的练习，跳过任何没有的练习。

1. 以您选择的语言启动一个新程序，并编写代码来表示DOM文本节点和元素的树。

2. 安装最新版本的Rust，然后下载并构建robinson。 打开dom.rs并扩展NodeType以包含注释节点等其他类型。

3. 编写代码以漂亮打印DOM节点树。

在下一篇文章中，我们将添加一个解析器，将HTML源代码转换为这些DOM节点的树。

## 引用
有关浏览器引擎内部的更多详细信息，请参阅Tali Garsiel的精彩浏览器工作方式及其与更多资源的链接。

例如代码，这里是“小”开源Web渲染引擎的简短列表。 他们中的大多数比鲁宾逊大很多倍，但仍然比Gecko或WebKit小。 WebWhir，2000行代码，是我称之为“玩具”引擎的唯一另一个。

- CSSBox (Java)
- Cocktail (Haxe)
- gngr (Java)
- litehtml (C++)
- LURE (Lua)
- NetSurf (C)
- Servo (Rust)
- Simple San Simon (Haskell)
- WeasyPrint (Python)
- WebWhirr (C++)

您可能会发现这些对于灵感或参考有用。 如果你知道任何其他类似的项目 - 或者如果你自己开始 - 请告诉我！

