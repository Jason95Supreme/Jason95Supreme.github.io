---
title: 你应该知道的setTimeout秘密
date: 2019-10-19 14:12:07
tags:
---
### 来来来，看下面这段代码：
```
    let start = new Date()
    setTimeout(function() {
        console.log(new Date() - start)
    }, 500)    
    while(new Date() - start < 1000) {}
```
<!-- more -->
<font size=3>
在上面的代码中，定义了一个setTimeout定时器，延时时间是500毫秒。你是不是觉得打印结果是：500。可事实却是出乎你的意料，打印结果是这样的（也许你打印出来会不一样，但肯定会大于1000毫秒）：
>**1004**

while 之前就设置了定时器，并添加到队列里面（假设时间点 a）， while 执行了1000ms 之后线程去队列里面取，发现 a + 1000ms > a + 500ms 说明 setTimeout 已经可以被执行了，不需要再等 500ms。

### setTimeout的应用：
```
    document.querySelector(#one_input).onkeydown = function() {
        document.querySelector(#one_span).innerHTML = this.value
    }
    document.querySelector(#second_input).onkeydown = function() {
        setTimeout(function() {
            document.querySelector(#second_span).innerHTML = document.querySelector(#second_input).value
        }, 0)  
    }
```
当你往两个表单输入内容时，你会发现未使用setTimeout函数的只会获取到输入前的内容，而使用setTimeout函数的则会获取到输入的内容。

这是为什么呢？

因为当按下按键的时候，JavaScript 引擎需要执行 keydown 的事件处理程序，然后更新文本框的 value 值，这两个任务也需要按顺序来，事件处理程序执行时，更新 value值（是在keypress后）的任务则进入队列等待，所以我们在 keydown 的事件处理程序里是无法得到更新后的value的，而利用 setTimeout(fn, 0)，我们把取 value 的操作放入队列，放在更新 value 值以后，这样便可获取出文本框的值。
setTimeout可以传入第三个参数、第四个参数....，它们表示神马呢？其实是用来表示第一个参数（回调函数）传入的参数。
</font>