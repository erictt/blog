---
layout: post
title: 部署和测试工具 - PHP进阶 (5)
category: 编程学习
tags: [PHP]
date: 2017-02-26
description: 部署和测试工具
keywords: 代码部署, 测试, PHP, Note
---

### 测试
#### 概念分类
* 单元测试(Unit Test) - 在项目中独立测试函数、方法、类等。
* TDD(Test-Driven Development) - 项目开始后，先写一部分测试用例，然后实现，然后继续写测试用例，继续实现，不断循环直至项目完成或终止。
* BDD(Behavior-Driven Development) - 基于行为流程来描述程序的实现。类似于单元测试，只是在实现上应用了比较友好的表达方式便于理解。
    * SpecBDD - 面向开发人员。使用编程语言表达。例： [phpspec](http://www.phpspec.net/en/stable/)
    * StoryBDD - 面向产品经理。基于英语语言表达。例：[behat](http://behat.org/en/latest/)

### 自动化部署工具
* [capistrano/capistrano](https://github.com/capistrano/capistrano)
* [magephp](http://magephp.com/)
* [Rocketeer](http://rocketeer.autopergamene.eu/)

### 代码分析工具
* [Xdebug](https://xdebug.org/)
* [blackfire](https://blackfire.io/)

