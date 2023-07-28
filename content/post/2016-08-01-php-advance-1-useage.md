---
layout: post
title: 常规用法 - PHP进阶 (1)
categories: [编程学习]
tags: [PHP]
date: 2016-08-01
description: PHP的一些知识梳理
keywords: Modern PHP, PHP, Namespace, traits, closure, 命名空间, 闭包, Hiphop
---

### 命名空间 Namespace (PHP 5.3+)

沙盒机制。虚拟的层级关系，更好的隔离不同的组建。PS：以下仅为推荐用法。

* 声明、引用、别名 (Declear, Import, Alias)

* 声明：

  <?php 
  // 必须在首行，推荐保持每个命名空间一行
  namespace Symfony\Component\HttpFoundation;

* 引用 & 别名
 // 推荐保持每个引用一行
 
### Class

 <?php
 // 引用
 use Symfony\Component\HttpFoundation\Response;

 $response = new Response('Oops', 400);
 $response->send();

 // 别名
 use Symfony\Component\HttpFoundation\Response as Res;

 $r = new Res('Oops', 400);
 $r->send();

### Function (PHP 5.6+)

 <?php 
 // 引用
 use function My\Full\functionName;

 // 别名
 use function My\Full\functionName as func;

### Constant

 // 引用
 use const My\Full\CONSTANT;

### PHP 7+ code

 use some\namespace\{ClassA, ClassB, ClassC as C};
 use function some\namespace\{fn_a, fn_b, fn_c};
 use const some\namespace\{ConstA, ConstB, ConstC};

### 全局命名空间
 
* 引用时不加入命名空间，程序默认为与当前命名空间相同
* 没有命名空间的程序，默认在global下，例如Exception。使用时需要在前边加上"/"：
 
  <?php
  namespace My\App;

  class Foo
  {
   public function doSomething()
   {
    $exception = new \Exception();
   }
  }

### 代码设计接口化

* 面向对象思维中的典型应用，在开发组件时尽可能的使用接口来统一功能，能为复用者或者后继开发人员节省非常多的时间

  // 实例来自PHP官方文档
  interface iTemplate
  {
      public function setVariable($name, $var);
      public function getHtml($template);
  }

  // Implement the interface
  class Template implements iTemplate
  {
      private $vars = array();
    
      public function setVariable($name, $var)
      {
          $this->vars[$name] = $var;
      }
    
      public function getHtml($template)
      {
          foreach($this->vars as $name => $value) {
              $template = str_replace('{' . $name . '}', $value, $template);
          }
   
          return $template;
      }
  }

### Traits (PHP 5.4+)

* 大部分编程语言都是采用继承(extend)来实现类的扩展，Traits存在的目的是注入代码片段。
* 简单的理解就是一段需要复用的代码，我们不再复制粘贴到各个文件当中，而仅仅是使用traits的方式来引入。
* 其实include好像也可以实现同样的功能？

  <?php
  trait SayWorld {
      public function sayHello() {
          echo 'Hello World!';
      }
  }

  class MyHelloWorld {
      use SayWorld;
  }

### 生成器 (Generator) (PHP 5.5+)

* 不同于JS中的Generator，PHP更单纯的是个迭代器，封装了自己的状态。目的是节省性能以及降低代码复杂度。
 
  <?php 
  // 如下代码在执行的时候会依次输出三个值
  function myGenerator()
  {
   yield 'value1';
   yield 'value2';
   yield 'value3';
  }

* 常与foreach循环一起连用。使用场景如逐行读取文件，重复的计算等
  
  <?php
  function getLines($file) {
      $f = fopen($file, 'r');
      try {
          while ($line = fgets($f)) {
              yield $line;
          }
      } finally {
          fclose($f);
      }
  }

  foreach (getLines("file.txt") as $n => $line) {
      if ($n > 5) break;
      echo $line;
  }

### 闭包(Closures) (PHP 5.3+)

* 通常会用在需要回调函数时，比如array_map()和preg_replace_callback()。

  <?php
  function cube($n)
  {
      return($n * $n * $n);
  }

  $a = array(1, 2, 3, 4, 5);
  $b = array_map("cube", $a);
  print_r($b);

* 另一种情况是动态追加函数或变量到当前函数/类中。

  * 低级用法

   <?php
   function enclosePerson($name) {
       return function ($doCommand) use ($name) {
           return sprintf('%s, %s', $name, $doCommand);
       };
   }
   // 在闭包函数中封装了字符串'Clay'
   $clay = enclosePerson('Clay');
   echo $clay('get me sweet tea!');

  * 高级用法
  
   <?php
   class App
   {
       protected $routes = [];
       protected $responseStatus = '200 OK';
       protected $responseContentType = 'text/html';
       protected $responseBody = 'Hello world';
       public function addRoute($routePath, $routeCallback)
       {
        // bindTo 将routeCallback函数追加给当前的app对象，然后将其赋值给$routePath键
        // __CLASS__ 为当前类名，并包含当前声明的命名空间，使用在此处，意为$routeCallback的作用域在当前类，
        // 想了解更多作用域相关信息请移步：http://php.net/manual/zh/closure.bindto.php#119410
           $this->routes[$routePath] = $routeCallback->bindTo($this, __CLASS__);
       }
       public function dispatch($currentPath)
       {
           foreach ($this->routes as $routePath => $callback) {
               if ($routePath === $currentPath) {
                // 该函数是在addRoute函数执行时被追加
                // 执行了回调函数后，赋值了responseContentType和responseBody
                   $callback();
               }
           }
           header('HTTP/1.1 ' . $this->responseStatus);
           header('Content-type: ' . $this->responseContentType);
           header('Content-length: ' . mb_strlen($this->responseBody));
           echo $this->responseBody;
       }
   }
   $app = new App();
   $app->addRoute('/users/josh', function () {
       $this->responseContentType = 'application/json;charset=utf8';
       $this->responseBody = '{"name": "Josh"}';
   });
   $app->dispatch('/users/josh');
   
### Zend OPcache (PHP 5.5+)

* 类似于APC、XCache等第三方cache扩展，预编译PHP代码为二进制程序，缓存在内存中。

_本文档参考了《O'Reilly Modern PHP》以及PHP官方文档_
