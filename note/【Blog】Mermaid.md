---
mermaid: true
---

# Mermaid

Mermaid 是一个基于 JavaScript 的图表绘制工具，可渲染 Markdown 启发的文本定义以动态创建和修改图表。

使用 Mermaid 可以让你的博客更加美观和生动。

关于 Mermaid 语法的详细内容，请查看[官方文档](https://mermaid.nodejs.cn/intro/)。

你也可以使用[官方的编辑器](https://mermaid-live.nodejs.cn/edit)绘制图表，然后把代码复制到博客中。

## 如何开启

[参考文章](https://zshellman.github.io/2019-06-10-Github-pages-%E5%A2%9E%E5%8A%A0mermaid%E8%AF%AD%E6%B3%95%E6%94%AF%E6%8C%81/)

*很遗憾，按照上文的方法似乎并不能在浏览器上正确渲染 Mermaid 图表，我会将这个问题添加到个人代办事项中，并在之后寻找解决办法*

## 示例

下面是第一个示例，用于测试 Mermaid 图表在 GFM 渲染器渲染的 GitHub Pages 博客中能否正确绘制：

```mermaid
---
title: Node
---
flowchart LR
    id

```

下面是第二个示例，用于测试 Mermaid 最新的 C4 图能否被正确渲染：

```mermaid
    C4Context
      title System Context diagram for Internet Banking System
      Enterprise_Boundary(b0, "BankBoundary0") {
        Person(customerA, "Banking Customer A", "A customer of the bank, with personal bank accounts.")
        Person(customerB, "Banking Customer B")
        Person_Ext(customerC, "Banking Customer C", "desc")

        Person(customerD, "Banking Customer D", "A customer of the bank, <br/> with personal bank accounts.")

        System(SystemAA, "Internet Banking System", "Allows customers to view information about their bank accounts, and make payments.")

        Enterprise_Boundary(b1, "BankBoundary") {

          SystemDb_Ext(SystemE, "Mainframe Banking System", "Stores all of the core banking information about customers, accounts, transactions, etc.")

          System_Boundary(b2, "BankBoundary2") {
            System(SystemA, "Banking System A")
            System(SystemB, "Banking System B", "A system of the bank, with personal bank accounts. next line.")
          }

          System_Ext(SystemC, "E-mail system", "The internal Microsoft Exchange e-mail system.")
          SystemDb(SystemD, "Banking System D Database", "A system of the bank, with personal bank accounts.")

          Boundary(b3, "BankBoundary3", "boundary") {
            SystemQueue(SystemF, "Banking System F Queue", "A system of the bank.")
            SystemQueue_Ext(SystemG, "Banking System G Queue", "A system of the bank, with personal bank accounts.")
          }
        }
      }

      BiRel(customerA, SystemAA, "Uses")
      BiRel(SystemAA, SystemE, "Uses")
      Rel(SystemAA, SystemC, "Sends e-mails", "SMTP")
      Rel(SystemC, customerA, "Sends e-mails to")

      UpdateElementStyle(customerA, $fontColor="red", $bgColor="grey", $borderColor="red")
      UpdateRelStyle(customerA, SystemAA, $textColor="blue", $lineColor="blue", $offsetX="5")
      UpdateRelStyle(SystemAA, SystemE, $textColor="blue", $lineColor="blue", $offsetY="-10")
      UpdateRelStyle(SystemAA, SystemC, $textColor="blue", $lineColor="blue", $offsetY="-40", $offsetX="-50")
      UpdateRelStyle(SystemC, customerA, $textColor="red", $lineColor="red", $offsetX="-50", $offsetY="20")

      UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")



```