---
layout: post
title: "PHP闭包Closure（匿名函数）"
keywords: "PHP,闭包Closure,匿名函数,作用域,use"
description: ""
category: PHP
tags: [php, closure]
---
{% include JB/setup %}

闭包在php 5.3.0之后才被引入。

php的闭包是通过匿名函数实现的。

申明一个没有指定名称的函数即可实现闭包。




## 闭包基本用法

要使用闭包，需要将其赋值给变量才能使用闭包。

    <?php
    $greet = function($name)
    {
        printf("Hello %s\r\n", $name);
    };
    $greet("World");  //输出 Hello World
    $greet("PHP");  //输出Hello PHP
    ?>
    
## 闭包作用域

1.默认情况下，闭包无法使用所在代码块的上下文变量和值，如果要使用，需要使用关键字use进行申明。

2.默认情况下，无法在闭包内改变上下变量的值，因为通过use申明，只不过是将上下文变量复制了一个副本放在use中，所以无法修改。

3.如果要在闭包内改变上文变量的值，需要显式的在use中引用上下文变量。

    <?php
    $test = 1;
    $unrefer = function() use($test) 
    {
        $test;
        var_dump($test);
        $test++; 
    };
    $unrefer();  //输出 1 
    var_dump($test);  //输出 1: 未引用
    $refer = function() use(&$test) 
    {
        $test;
        var_dump($test);
        $test++; 
    };
    $refer();  //输出 1 
    var_dump($test);  //输出 2: 引用
    ?>

4.就算使用use引用了上下文变量，但是闭包还是会把use引用的变量保存限制在闭包内，外界是无法得到，需要赋值给变量才能得到。

    function getMoneyFunc()
    {
        $rmb = 1;
        $func = function() use ( &$rmb ) {
            echo $rmb;
            //把$rmb的值加1
            $rmb++;
        };
        return $func;
    }
    $getMoney = getMoneyFunc();
    $getMoney();  //输出 1
    $getMoney();  //输出 2
    $getMoney();  //输出 3

5.use延迟绑定变量

    <?php
    $result = 0;
    $one = function()
    {
        var_dump($result);
    };
    $two = function() use ($result)
    {
        var_dump($result);
    };
    $three = function() use (&$result)
    {
        var_dump($result);
    };
    $result++;
    $one();  //输出 null: Undefined variable(不在作用域内)
    $two();  //输出 0: 未引用，在申明闭包时赋值
    $three();  //输出 1：引用后，在调用闭包时赋值
    ?>

