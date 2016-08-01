---
layout: post
title: "JavaScript中的变量提升"
keywords: ["JavaScript","变量提升包","js","var"]
description: "详细解读JavaScript中的变量提升"
category: "JavaScript"
tags: ["JavaScript"]
---
{% include JB/setup %}

##执行环境

执行环境（或称“环境”）是 JavaScript 中非常重要的概念。执行环境决定了变量或函数有权访问其它数据，决定了它们各自的行为。<span class="txt">每个执行环境都有一个与之关联的**变量对象**，环境中定义的所有变量和函数都保存在这个对象中。</span>这个对象是我们的代码无法访问到的，供解析器后台使用。

与执行环境关联的变量对象包含在当前执行环境中定义的：

1. 函数的形参
2. var声明的变量
3. 函数声明（不包括函数表达式）

可以这样理解：

<pre>
执行环境 = {
	变量对象:{
		//执行环境中的数据
	}
};
</pre>

最外层的执行环境是全局执行环境，window 对象与之关联，所有的全局变量和方法都作为 window 对象的属性和方法被创建。某个执行环境中所有代码执行完毕后，该环境被销毁，保存在其中的所有变量及函数定义也随之被销毁。全局执行环境在应用程序退出后被销毁，比如关闭网页。

每个函数都有自己的执行环境。<span class="txt">当代码在一个执行环境中执行时，会创建变量的一个**作用域链**。</span>

作用域链保证了执行环境中有权访问的变量及函数的有序访问。<span class="txt">作用域链的前端始终是当前执行代码所在环境的变量对象。</span>如果当前执行环境是函数的话，则将其**活动对象**（作用域链上正在被执行和引用的变量对象）作为变量对象。作用域链下一个变量对象来自外部环境，再下一个来自外部的外部，以此类推，直到尾端的全局执行环境的变量对象。

<pre>
var girl = "Selina";
function showGirl(){
    var girl2 = "Hebe";
    function swapGirl(){
        var temp = girl;
        girl = girl2;
        girl2 = temp;
        //可访问 temp,girl,girl2
    }
    //可访问 girl,girl2
    swapGirl();
}
//只能访问 girl
showGirl();

alert(girl);	//"Hebe"
alert(girl2);	//error
</pre>

这段代码的作用域链关系如下图：

![](http://cdn.saymagic.cn/o_1ap1bv7vgmni1opo1rhc1cs7178t9.png)

图中不同色块分别代表不同的执行环境，内部环境可以通过作用域链访问所有外部环境，但外部不能访问内部的变量和函数。例如，`swapGirl()`作用域链包含`swapGirl()`变量对象、`showGirl()`变量对象、全局变量对象这3个对象；`showGirl()`的作用域链包含它自己的变量对象和全局变量对象这2个对象，它不能访问内部函数`swapGirl()`的执行环境。

##函数级作用域

ES6之前，JavaScript 没有块级作用域。一个独立的执行环境不是由封闭的花括号而是函数来分界的。

<pre>
if(true){
    var girl = "Selina";
}
alert(girl);	//"Selina"
</pre>

这段代码中，执行`alert(girl);`没有返回`Uncaught ReferenceError: girl is not defined`这样的异常，而是弹出 if 语句中定义的"Selina"，这是为什么呢？

if 语句是一个由花括号封闭起来的“块”，在 Java、C 等语言中，块中定义的局部变量会在 if 语句执行完毕后被销毁，而在 JavaScript 中，if 语句中的变量声明会将变量添加到当前的执行环境。

for 语句同理。

<pre>
for(var i = 1; i < 10; i++){
    //do something...
}
alert(i);	//10
</pre>

如果你想像在 Java 中那样使用块级作用域，你可以用 ES6 新增的 let 声明块级作用域的变量，const 声明常量。不过这不是我们今天主要讨论的范围，暂不多做介绍。

##变量提升

熟悉 C 类语法的朋友应该知道，C、Java 这类语言中的变量必须“先声明，后使用”，否则就会报错。而 JavaScript 却能够在变量和函数声明之前就使用。

JavaScript 中，变量有4种方式进入作用域：

1. 语言内置。执行环境中的 this 和 arguments。（全局环境中无 arguments 对象）
2. 形式参数。函数的形参会作为作用域的一部分。
3. 函数声明。形如`function showGirl(){}`的形式。
4. 变量声明。形如`var girl`。

只有第三点提到的函数声明和第四点中 var 形式的变量声明才能进行提升。

提升，可以按照字面意思理解，就是把函数或变量声明在解析时提到执行环境最前面。

<pre>
name = "Selina";
var name;
alert(name);	//"Selina"
</pre>

JavaScript 引擎解析代码时其实把上面的代码变成：

<pre>
var name;
name = "Selina";
alert(name);	//"Selina"
</pre>

变量 name 的声明被提到了最前面，所以 alert 函数能弹出`Selina`而不是`undefined`。

再强调一下，只有用 var 声明的变量才存在变量提升，用 let、const 进行声明并不会提升。


