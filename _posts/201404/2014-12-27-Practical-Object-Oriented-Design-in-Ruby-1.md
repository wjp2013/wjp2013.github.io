---
layout: post
title:  "面向对象设计实践指南 1"
date:   2014-12-27 16:30:00
categories: ruby
tags: learningnote refactoring
author: "Victor"
---

## 前言

随着代码的增长和对需求的变化，那些当前系统里还不存在的业务逻辑将会添加进去。**在所有的情况中，代码的可维护性在整个项目的生命周期中始终比优化其现有状态更加重要。**

常见的设计原则：SOLID原则，DRY原则，TRUE原则。
常见的设计模式：工厂模式，组合模式，模板方法。

TRUE原则：具有透明性、合理性、可用性和典范性的代码，不仅能满足今天的需求，也因便于修改而能满足未来的需求。

## 第一章 面向对象设计

OOD 的失败并非是编码技术的失败，实际上是视角的失败。

### 设计赞歌

#### 设计解决的问题

客户并不知道自己想要的是什么，所以需求总是会不停的更改和新增。即便该应用程序取得了成功，但是大家仍然期望得到更多新功能。正是这种变化的需求让设计变得如此重要。

#### 为何难以更改

面向对象设计与依赖关系管理相关。它是一套对依赖关系管理进行编排，以便各个对象能够容忍更改的编码技术。

在缺乏设计的情况下，对象彼此之间了解过多，任何一个对象的修改都会强制要求其合作者也随之发生变化。

#### 实用的设计定义

设计的目的是允许你以后可以进行设计，而设计的首要目标是降低变化所带来的成本。

### 设计工具

#### 设计原则

* SOLID原则
  * 单一职责原则 Single Responsibility Principle, SRB
  * 开闭原则 Open-Closed Principle, OCP
  * 里氏替换原则 Liskov Substitution Principle, LSP
  * 接口隔离原则 Interface Segregation Principle, ISP
  * 依赖倒置原则 Dependency Inversion Principle, DIP
* 不重复原则 Don't Repeat Yourself, DRY
* 迪米特法则 Law of Demeter, LoD

#### 设计模式

设计模式指对常见问题进行命名，并使用常见的方法来解决这些问题。

### 设计行为

设计原则和模式都是设计工具，利用它们而产生的结果取决于程序员使用设计工具的经验。

#### 设计失败

1. 缺乏设计知识，不了解设计原则和模式。编写时很轻松，改起来则会变得越来越难。
2. 了解 OO 设计技术，但会过度设计。这样的程序员在面对变更请求时总会抱怨：“不行，我不能添加这项功能，它不是设计来干这事的。”
3. 设计和编程行为分开。因为设计是一个逐步发现的过程。不能一开始由“设计专家”一步到位的设计好。

#### 设计时机

大部分程序员习惯于预先设计，这让我们觉得软件开发的过程是可控的。我们只要按部就班的完成每一步开发任务即可。但这会造成客户和程序员之间的敌对关系。因为软件真正完成之前，任何预先设计都不可能是正确的。 然后客户要求修改，程序员拒绝修改。大家围绕着最初的设计文档争论不休，尽管这个文档的初衷是实现高质量软件的路线图，结果却成了双方逃避责任的争论焦点。

如果你也认同这一点，那么我们会发现两件事：
1. 大规模预先设计完全没意义，它不可能正确，因为客户都不清楚自己想要什么。
2. 没人能预测应用程序什么时候会完成，因为功能还不确定。

敏捷开发承认：在应用程序最终形成之前，确定性是遥不可及的。因此我们离不开简单、灵活和可塑性强的代码。

敏捷开发方法认为：想要生产出客户真正需要的程序，最好的方法是与他们一起合作，逐步构建软件。

#### 设计评判

OOD 度量无法辨别出那些“在方法上正确而在做法却是错误”的设计。

如果因时间压力而减少设计，就像是向未来借用时间。这就是技术债务，终归要偿还，并且还会带上利息。

因为设计也要占用时间和成本，因此我们要衡量设计和因设计而带来的利益。不能因为1年之后才可能带来好处的设计而浪费当下的时间。

### 面向对象编程简介

面向对象的应用程序由对象和它们之间传递的消息构成。**其中，消息相对更重要。**

* 对象拥有行为，包含可单独访问控制的数据，对象之间通过互相发送消息来调用彼此的行为。
* Ruby 有字符串对象，没有字符串数据类型。
* String 类可以反复的实例化(创建)新的字符串实例。每一个实例实现相同的方法，有相同的属性名称，但包含各自不同的数据和状态。
* OO 语言是开放式的，不会将你限制在一个很小的内建类型和实现预定义好的操作集合里。