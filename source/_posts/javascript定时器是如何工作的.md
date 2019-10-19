---
title: javascript定时器是如何工作的
date: 2019-10-19 14:29:42
tags:
---
### 这是一篇译文...原文地址是...搞忘记了
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>从最基本的层面上讲明白javascript定时器如何工作是非常重要的。很多时候它们的执行过程不直观的原因是因为他们是处于单线程中执行的。下面用三个我们能够构造和操作定时器的函数来分析。
</font>
<!-- more -->
<font size=3>
```
var id = setTimeout(fn, delay); --启用一个定时器将会在延时时间后调用指定的函数。函数会返回一个唯一的ID用以在一段时间后取消定时器。
var id = setInterval(fn, delay); --与setTimeout相似但是会持续调用函数（每隔一段时间）直到定时器被取消。
clearInterval(id);,clearTimeout(id); --接受一个定时器ID（由上述两函数中某一个返回）并且取消定时器的调用。
```
</font>
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>
为了弄明白定时器内部如何工作的，就需要对一个很重要的概念做进一步探讨：时间延迟并没有确定性。所有在浏览器上的javascript代码都会在一个单线程异步事件上执行（譬如鼠标点击和定时器）而这个线程只有当执行环境开放时才会运行。下面的图解很好的演示了它是如何工作的：
</font>

{% asset_img timer1.png %}

&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>乍一看这张图好像有许多信息要我们消化理解，但是完完全全的弄明白这张图会让你更好的认识到异步javascript代码的是如何工作的。这个图是单向执行的：在垂直方向是以毫秒为单位的时间。蓝色的方框则代表了javascript代码正被执行的部分。比方说第一个方框表示javascript代码大概执行了18ms左右，而鼠标点击块则是大约执行了11ms左右等等。
</font>

### 进一步剖析：
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>由于javascript一次只能执行一段代码（因为它的单线程性质）这些代码块中的每一个都会”阻塞“其他异步事件的进程。这就意味着当一个异步事件开始时（像一个鼠标点击事件，一个定时器启动，或者是一个 XMLHttpRequest 完成时）它会先排队后执行（关于排队机制实际上是怎样一个过程对于不同的浏览器来说并不一样，所以这里认成是一个简化的过程）
</font>

&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>首先，在第一个javascript块中，两个定时器启动：一个10ms的setTimeout定时器和一个10ms的setInterval定时器。由于timer所处的位置和时间上的原因它实际上在第一个代码块完成之前就启动了。但是请注意这并不意味着它会立即开始执行（因为单线程的缘故它是不能够立即执行的）。取而代之的是延时函数会排队等待一直到下一次可执行时间段才会执行。
</font>

&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>除此之外，在第一个javascript块中还存在一个鼠标点击事件，与此相关的javascript回调异步事件（我们永远不知道用户什么时候会执行一个操作，因此就认为它是异步的）也不会立即执行，像最初的timer一样会排队等待可执行的时间点。
</font>

&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>在最初的javascript块执行完成之后浏览器会立即询问：有什么正在等待的操作需要被执行？而这时鼠标点击处理程序和一个timer回调都是正在等待被执行的。随后浏览器会选择一个（鼠标点击回调）并立即执行。timer则会等待直到下一次可执行的时间点才会执行。
</font>

&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>注意当鼠标点击处理程序在执行时，第一个interval回调也执行了。与timer一样它的处理程序会排队稍后执行。然而请注意当interval再次触发时（当timer处理程序正在执行时）这段时间处理程序会执行删除操作。假设你有一大段代码要执行时则所有的interval回调都会挂起等待，结果就是这些interval执行时是没有延时的。而浏览器的表现则是在下一次挂起等待之前简短的等待直到没有interval处理程序排队了（因为浏览器每次会询问是否有interval处于等待中）。
</font>

&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>事实上我们看到当第三个interval回调触发时interval它本身也在执行。这种情况向我们展示了一个重要的事实：interval并不关心当前是谁正在执行，它们将会陆续的排到队列中，即使这意味着回调之间的时间并不再准确。
</font>

&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>最后，在第二个interval回调完成执行时，我们看到没有什么东西要留给javascript引擎执行的了。这就表示不会再等待新的异步事件的产生。在图中可以看到50ms处又有一个interval触发，但是这次并没有任何事件会阻塞它的执行，因此它会立即执行。
</font>

### 看看区别：
<font size=3>
下面让我们看一个例子来更好的说明setTimeout和setInterval的区别：
</font>

{% asset_img timer2.png %}

&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>粗略瞟一眼这两段代码功能似乎是相同的，但实际上并非如此。尤其是setTimeout的代码总是会在前一个回调执行之后存在至少10ms的延迟（为什么是至少呢，因为可能会有某些原因导致延时加长，但延时绝不会比10ms更短）。然而setInterval将会试图每10ms执行一次回调不管上一次回调是什么时候执行的。
</font>

<font size=3>

**定时器有很多我们需要学习的东西，在这里只是总结几条比较重要的：**
&nbsp;&nbsp;&nbsp;&nbsp;**1. javascript引擎只有一个线程，迫使异步事件需要排队执行。**
&nbsp;&nbsp;&nbsp;&nbsp;**2. setTimeout和setInterval在如何执行异步代码上有根本性的不同。**
&nbsp;&nbsp;&nbsp;&nbsp;**3. 如果一个timer无法立即执行那么它将被延迟到下一个可能的执行点（这会导致比期望的延迟时间更长）。**
&nbsp;&nbsp;&nbsp;&nbsp;**4. 如果有段代码执行了足够长的时间那么interval则可能会连续执行相互之间没有没有延迟（这个时间要比指定的延迟时间更长）。**

所有以上的这些都是非常重要的知识点需要我们去记住。在不时有产生大量异步事件的时候，知道javascript引擎是如何工作的可以让我们在构建一个先进的应用程序时拥有一个极为强力的基础。
</font>