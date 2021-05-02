---
layout: mypost
title: 《你不知道的JavaScript》--bind函数实现
categories: [JavaScript]
---

## 背景

在《你不知道的JavaScript》的“this和对象原型”这一部分中，讲解this绑定规则优先级时，通过polyfill的 **Function.prototype.bind(...)** 的实现介绍了new绑定的优先级比显示绑定优先级高，在实现中，有关于this、原型链、闭包以及函数柯里化的应用。阅读时对其有一定理解，故在此记录。

## bind(...)实现
```javascript
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      throw new TypeError(
        'Function.prototype.bind - what is typing' +
        'to be bound is not callable'
      );
    }

    var aArgs = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP = function() {},
        fBound = function() {
          return fToBind.apply(
            (
              this instanceof fNOP && oThis ? this : oThis
            ),
            aArgs.concat(
              Array.prototype.slice.call(arguments)
            );
          )
        }
    ;
    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
  }
}
```

## 解析

该函数通过传入需要绑定的this对象，最后返回一个函数，实现了this指向的改变。

1. 在bind函数中，通过`slice`获取到除第一个需要绑定的this对象之外的参数，供给返回函数使用，这种技术称为**柯里化**

2. 在bind函数中定义了fNOP空函数，将其原型设置为原函数原型，用于后续判断new操作，并将返回函数的原型设置为fNOP函数的实例，实现原函数的**原型链继承**

3. 在返回的函数中，通过`apply`调用原函数，先通过 `instanceof`判断调用返回函数时传入的this对象是否是原函数的实例，即`new `调用，如果是，就取消绑定效果；然后将bind函数中获取到的参数和返回函数调用时传入的参数合并调用，这里返回函数访问bind函数的变量现象成为**闭包**


