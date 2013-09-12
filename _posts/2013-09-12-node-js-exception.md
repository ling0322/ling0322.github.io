--- 
layout: post
title: Node/Javascrit的坑爹的try…catch
---
这几天粗糙的完成了webdict.info这个小网页的设计，这个也是我第一次尝试使用node.js做真正的网站，之前几次都是toy的东西。因为之前都是使用Python做web，感觉node的异步调用方式很新奇，虽然之前小试过Tornado这个Python的异步web框架，但是Python都是sync的API，使用async方式感觉各种缺支持。

就在这种现学现卖的情况下完成了webdict.info，并且在实验室和同学之间内测的时候一点问题都没有，但是放到公网上的时候简直就是噩梦，最悲惨的时候服务器每隔10分钟当掉一次。后来分析log发现是有一个异常被抛出，然后因为没有被caught，程序就直接挂掉了。

虽然这个的解决方案很简单，在程序开头加上

    process.on('uncaughtException', function(err) {
      console.log('Caught exception: ' + err);
    });
    
就可以确保程序不会因为这个问题当掉了，但是有一个问题很奇怪，明明在handlerFunction外面有catch把这个exception抓住，然后显示一个500错误，为什么在这边直接让程序当掉了呢？

首先说一下exception的catch机制，一个exception被抛出，就开始回退函数的调用栈，如果当前栈顶函数中有try…catch那就首先停止栈回退，然后去执行catch中的代码；如果栈顶函数没有try…catch那么把当前函数从栈中弹出，然后重复以上过程。最终如果栈中函数没有一个能够catch到当前exception的话，那么就程序出错退出。

比如这个程序

    var f = function () {
      throw new Error('exception'); 
    };
    f();

错误时候的堆栈就是

    Error: exception
    at f (/Users/ling0322/Documents/test.js:2:9)
    at Object.<anonymous> (/Users/ling0322/Documents/test.js:5:1)
    at Module._compile (module.js:456:26)
    at Object.Module._extensions..js (module.js:474:10)
    at Module.load (module.js:356:32)
    at Function.Module._load (module.js:312:12)
    at Function.Module.runMain (module.js:497:10)
    at startup (node.js:119:16)
    at node.js:901:3
    
这个很明显在函数f里面抛出了一个异常，因此栈顶是函数f。但是看看异步调用和匿名函数的情况，间隔50毫秒以后再抛出异常会是什么情况呢

    var f = function () { 
      setTimeout(function () {
        throw new Error('exception'); 
      }, 50);
    };
    f();
    
这个在异常抛出时候堆栈是

    Error: exception
        at null._onTimeout (/Users/ling0322/Documents/test.js:4:15)
        at Timer.listOnTimeout [as ontimeout] (timers.js:110:15)
        
很明显因为在抛出异常的匿名函数运行时，函数f已经运行完成从堆栈中被退出了，因此函数f中的try…catch是捕捉不到这个异常的。

关于如何处理这种异常，可以参考这篇文章[Uncaught Exceptions in Node.js](http://shapeshed.com/uncaught-exceptions-in-node/)