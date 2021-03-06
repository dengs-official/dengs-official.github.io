---
layout: mypost
title: 《你不知道的JavaScript》--bind函数实现
categories: [JavaScript, 基础底层]
---

## 背景

在《你不知道的JavaScript》的“this和对象原型”这一部分中，在讲解this绑定规则优先级时，通过ES5内置的 **Function.prototype.bind(...)** 的实现介绍了new绑定的优先级比显示绑定优先级高，其中，在关于bind的实现代码中，关于this、闭包以及函数柯里化都有应用。阅读时对其有一定理解，故在此记录。

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