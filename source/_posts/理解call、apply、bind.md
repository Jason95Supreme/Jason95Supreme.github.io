---
title: 理解call、apply、bind
date: 2019-11-06 21:10:59
tags:
---
### 好像真是一件难办的事儿？
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>以前看到各种call、apply、bind的调用简直是一脸懵逼，也不说完全不知道啥意思，但要想说很明白他们是怎么实现怎么调用的那也不现实。总感觉一半靠运气，一半靠实力，哈哈。。。不过在不懈的努力下，经过多次别人文章阐述的洗礼，终于能用自己的方式理解了这些个调用的意义。
</font>
<!-- more -->

### 如何用自己的方式理解call
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>话不多说，直接用例子说明一切，基于这个再来理解会容易许多。
</font>
```
let person = {
  name: 'Abiel'
}
function sayHi(age,sex) {
  console.log(this.name, age, sex);
}
sayHi.call (person, 25, '男') // 输出Abiel 25 男
```
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>从简单分析起，调用了call之后，第一个参数是当前方法的this，即作用域，call后面的参数都是作为函数参数传进去使用的。那么如何用自己的方式来理解呢？可以这样来想，每当看到call的时候，第一个参数这里是person对象，那么即sayHi的this指向了person对象，也就是！person对象里多了一个sayHi的函数，长这样。
</font>
```
let person = {
  name: 'Abiel',
  sayHi: function(age,sex) {
    console.log(this.name, age, sex);
  }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>一下就变得很熟悉了，这个时候sayHi被调用执行了，this.name即是指的person对象中的name: 'Abiel'，再把后面的参数作为函数参数传进去执行后，就是那个结果了。来看看原生实现：
</font>
```
Function.prototype.newCall = function(context, ...parameter) {
 if (typeof context === 'object' || typeof context === 'function') {
    context = context || window
} else {
    context = Object.create(null)
}
  context['fn'] = this  
  const res =context['fn'](...parameter)
  delete context.fn;
  return res
}
```
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>在function的原型上自定义一个自己的call方法，第一个参数指代当前的上下文，第二个参数指代即将传给函数执行的参数。如果第一个参数是对象或者函数的话，直接就将上下文指向对象或者函数，其他类型就创建一个新的对象类型。这里可以简单的将context理解为{}，毕竟函数也算是对象了，可以拥有自己的属性和方法。这个时候我们可以在{}新对象内部新创建一个属性名叫fn。因为call是将第一个参数作为被调用函数的this作用域的，**意思就是我们可以将其理解成函数将要在第一个参数的内部进行调用**。而这里sayHi对newCall进行了调用，那么newCall自身的this指向的就是sayHi这个函数本身，我们将sayHi函数本身赋值给了context内部的属性fn。下面紧接着在对象context内部进行了函数的调用，并传入参数，最后删除这个属性。一个call函数的执行流程就算完了。

### apply的原生实现
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>apply这里跟call有点区别就是apply参数是数组。来看看原生实现（内部基本与call的实现一致）：
</font>
```
Function.prototype.newApply = function(context, parameter) {
  if (typeof context === 'object' || typeof context === 'function') {
    context = context || window
  } else {
    context = Object.create(null)
  }
  let fn = Symbol()
  context[fn] = this
  return res=context[fn](..parameter);
  delete context[fn]
  return res
}
sayHi.newApply (person,[ 25, '男']) //Abiel 25 男
```

### bind的原生实现
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>bind的区别更大些，因为它返回的是一个新的函数。来看看原生实现：
</font>
```
Function.prototype.bind = function (context,...innerArgs) {
  var me = this
  return function (...finnalyArgs) {
    return me.call(context,...innerArgs,...finnalyArgs)
  }
}
let person = {
  name: 'Abiel'
}
function sayHi(age,sex) {
  console.log(this.name, age, sex);
}
let personSayHi = sayHi.bind(person, 25)
personSayHi('男')
```