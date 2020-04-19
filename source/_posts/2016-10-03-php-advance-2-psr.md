---
layout: post
title: PSR标准解读 - PHP进阶 (2)
category: 编程学习
tags: [PHP, 编程规范]
date: 2016-10-03
description: PSR标准解读
keywords: Modern PHP, PHP, Note, PSR, PHP-FIG
---

### _PSR-0 自动加载的标准结构 (虽然已不被推荐使用，但composer还在支持)_

#### 要点：

* 结构： `\<Vendor Name>\(<Namespace>\)*<Class Name>`
* 第一层级的命名为开发人员的标识
* 完整的命名空间可包含多个层级
* 在加载文件的时候，命名空间的分隔符`\`会被`DIRECTORY_SEPARATOR`替换
* 类名中的`_`会被`DIRECTORY_SEPARATOR`替换，所以`_`无任何意义
* 在加载文件的时候，命名空间会补全`.php`后缀
* 命名空间及类命名可以大小写随意
* 示例：

		\Symfony\Core\Request => /path/to/project/lib/vendor/Symfony/Core/Request.php

		\namespace\package_name\Class_Name => /path/to/project/lib/vendor/namespace/package_name/Class/Name.php
	* 需要注意的是，层级目录会重复vendor和namespace。

#### 加载器的实现示例：

	<?php

	function autoload($className)
	{
	    $className = ltrim($className, '\\');
	    $fileName  = '';
	    $namespace = '';
	    if ($lastNsPos = strrpos($className, '\\')) {
	        $namespace = substr($className, 0, $lastNsPos);
	        $className = substr($className, $lastNsPos + 1);
	        $fileName  = str_replace('\\', DIRECTORY_SEPARATOR, $namespace) . DIRECTORY_SEPARATOR;
	    }
	    $fileName .= str_replace('_', DIRECTORY_SEPARATOR, $className) . '.php';

	    require $fileName;
	}
	spl_autoload_register('autoload');



### PSR-1 基本代码样式
* 标签 (PHP tags)
	
	代码必须包含在<?php ?> 或者 <?= ?>中

* 编码 (Encoding)

	必须使用带BOM的UTF-8

* 对象化 (Objective)

	单个PHP文件只负责一件事。定义类、trait、function、常量或者执行一系列的动作。不应该同时包含定义和执行代码。

* 自动加载 (Autoloading)

	命名空间和类必须支持PSR-4标准。

* 类名 (Class names)

	使用驼峰格式，首字母大写。例：CoffeeBean、TitleCase。

* 常量名 (Constant names)

	必须全大写，并以下划线切分单词。例：PROJECT_NAME。

* 方法命名 (Method names)

	使用驼峰格式，首字母小写。例：testCode。

### PSR-2 严格代码样式

* 包含PSR-1中的所有要求

* 首行缩进 (Indentation)

	使用4个空格 (鄙人也是这个阵营的)

* 文件和行 (Files and lines)

	PHP文件必须使用Unix格式的换行结尾，最后必须多一个空行。必须省略"?>"结尾标签。

	单行字符最好不超过80，最多不能大于120.

	每行最后不能有多余的空格。

* PHP关键词 (Keywords)

	所有PHP的关键词使用小写。`true`、 `false`、 `null`等。

* 命名空间 (Namespaces)

	每个命名空间的声明必须跟着一空行。同理在使用`use`关键词的时候后边也需要跟一空行。

		<?php
		namespace MY\Component;

		use Symfony\Components\HttpFoundation\Request;
		use Symfony\Components\HttpFoundation\Response;

		class App
		{
			// Class defination body
		}
	
* 类 (Classes)

	定义类时使用的大括号必须在新的一行。在使用`extends`或`implements`时保证在同一行

		<?php
		namespace MY\Component;

		class App extends YourApp
		{
			// Class defination body
		}

* 方法 (Methods)

	大括号同样另起一行。参数以一个逗号加空格隔开，结尾不能有多余的空格。

		<?php
		namespace MY\Component;

		class App extends YourApp
		{
			public function doSomething($firstParam = true, $secondParam = 2)
			{
				// Method defination body
			}
		}

* 方法的可见行 (Visibility)

	使用关键词`public`、`protected`、`private`代替之前经常使用的`var`，或者变量以下划线开始来表示私有的方式。

	在定义方法为`abstract`或者`final`时，可见行关键词必须在前。

		<?php
		namespace MY\Component;

		class App extends YourApp
		{
			// Static property with visibility
			public static $numberOfBirds = 0;

			// Method with visibility	
			public function __construct()
			{
				static::$numberOfBirds++;
			}
		}

* 控制结构 (Control structures)
	
	所有的控制结构类关键词必须跟一个空格。例：`if`、`elseif`、`else`、`while`、`case`、`switch`、`do`、`for`、`foreach`、`try`、`catch`。

		<?php
		$gorilla = new \Animal\Gorilla;
		$ibis = new \Animal\StawNeckedIbis;

		if ($gorilla->isAwake() === true) {
			do {
				$gorilla->beatChest();
			} while($ibis->isAsleep() === true);

			$ibis->flyAway();
		}

### PSR-3 日志接口

* PSR-1和PSR-2是编码规范， PSR-3是接口定义，它定义了PHP的日志组件的一系列接口信息。

* 一个PHP的日志组件应该实现了`Psr\Log\LoggerInterface`接口的所有方法。该接口的定义参照了系统日志协议RFC5424。具体内容可参照： [LoggerInterface.php](https://github.com/php-fig/log/blob/master/Psr/Log/LoggerInterface.php)定义。

### PSR-4 自动加载器

* 同PSR-0，该标准描述了如何在运行时定位及加载PHP的类、接口以及traits。可以成为PSR-0的简化升级版。移除了对`_`转换成`DIRECTORY_SEPARATOR`的支持，目录结构也不需要再在`src/`下重复vendor和namespace。
* 比如`\Oreilly\ModernPHP`这个命名空间对应的物理地址`src/`。PHP就知道所有使用了这个命名空间的类、接口以及traits都在`src/`目录下边。而命名空间`\Oreilly\ModernPHP\Chapter1\`对应的物理文件应该是`src/Chapter1/`。

_本文档参考了《O'Reilly Modern PHP》以及PHP官方文档_

