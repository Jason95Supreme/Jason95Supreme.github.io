---
title: 仿造vue模板写一个自定义模板插入数据的小demo
date: 2019-11-09 21:35:21
tags:
---

### 为撒子突然想到写这个了
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>最近正在看vue源码实现机制的视频，里面讲到了如何实现模板替换的问题，听了课之后觉得光听不练还是菜，所以最后决定仿造视频的例子在自己用自己的理解写一遍，以此加深记忆。
</font>
<!-- more -->

### 开始，先写一个html
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>既然要实现模板替换，那么不可缺少的html先得拿出来，主体html代码如下：
</font>
```
<div id="root" class="root">
    <div>
        <div>
        <p>{{name}}--{{msg}}</p>
        </div>
    </div>
    <p>{{sex.sex1.male}}</p>
    <p>{{sex.sex1.female}}</p>
</div>
```

### 来点儿数据加点料
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>这里我们在html自定义了模板写法\{ \{ \} \}，里面包含了我们想要测试的一些字段，接下来就是数据了，这里我提供了如下数据：
</font>
```
data: {
    name: '我都名字',
    msg: '有条消息',
    sex: {
        sex1: {
        male: '男',
        female: '女',
        }
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>因为是自己写出来测试的缘故，就随便写死了些数据方便测试。我们都知道Vue的话引入了库之后在js里面new一个Vue的实例，参数为一个对象，包含要挂载到的节点属性名，数据，这里简单处理我们就只提供了节点和数据了。从上面的html代码里可以看到，我们准备挂载的节点属性名为root，参照vue挂载实例的方式我们可以写出下面的代码来：
</font>
```
new newVue({
    el: "#root",
    data: {
        name: '我都名字',
        msg: '有条消息',
        sex: {
            sex1: {
            male: '男',
            female: '女',
            }
        }
    }
})
```

### 正菜在这里
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>而要实现上面的这个newVue构造函数的话，我们需要对传入的参数进行处理。这里理一下思路：
**1. 先用cloneNode(true)方法拷贝一份原dom节点的数据。因为我们不能直接修改原dom，不然后面如果有其他的数据都没法继续渲染了。（模板是不变的，变得是数据）**
**2. 传入的节点进行递归遍历，查找到所有格式为\{ \{ \} \}的文本节点，并用data中的数据替换文本节点的nodevalue。**
**3. 将修改过后的新dom元素替换原本的dom节点。**
我使用了一个class语法糖的形式进行newVue构造函数的编写，render函数作为渲染函数，compiler函数处理内部递归替换逻辑，update函数使用新节点替换旧节点。相关实现代码如下：
</font>
```
class newVue {
  constructor(options) {
    this._el = document.querySelector(options.el)
    this.$el = this._el.cloneNode(true)
    this._parent = this._el.parentNode
    this._data = options.data
    // 替换原本的节点
    this.render()
  }
}

function compiler(template, data) {
  const reg = /\{\{(.*?)\}\}/g
  const childNodes = template.childNodes
  
  for(let i=0; i<childNodes.length; i++) {
    const type = childNodes[i].nodeType
    // 文本节点
    if(type === 3) {
      childNodes[i].nodeValue = childNodes[i].nodeValue.replace(reg, function(a, b) {
        const key = b.trim()
        const finalTxt = this.getMultiLevel(data)
        return finalTxt(key)
      })
    }
    // 元素节点
    else if(type === 1) {
      this.compiler(childNodes[i], data)
    }
  }
}
function getMultiLevel(data) {
  return function(str) {
    const strArr = str.split('.')
    let res = data
    let key = ''
    while(key = strArr.shift()) {
      res = res[key]
    }
    return res
  }
}

newVue.prototype.render = function() {
  this.compiler()
}
newVue.prototype.compiler = function() {
  compiler(this.$el, this._data)
  this.update()
}
newVue.prototype.update = function() {
  this._parent.replaceChild(this.$el, this._el)
}

```

### 落幕
&nbsp;&nbsp;&nbsp;&nbsp;<font size=3>上面代码中还有个我刚刚没提到的函数：**getMultiLevel**，这个函数采用了柯里化写法，简化了需要传给函数参数的个数，内部则是对多层级模板写法例如：\{ \{sex.sex1.male\} \}，进行递归遍历，以访问到更深层次的data数据。最后的实现效果我放两张图更直观。
</font>
实现之前：
{% asset_img 1.png %}
实现之后：
{% asset_img 2.png %}
完美！perfect！当然，上面的实现只是一个简化简化再简化的模板替换方案，还没有使用到虚拟dom。虚拟dom的话后面应该也会加上的~