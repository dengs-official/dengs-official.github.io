---
layout: mypost
title: 《Vue源码解析》-Vue&VNode对象
categories: [Vue.js]
---

[TOC]

## Vue对象

Vue对象是开发过程中经常接触到的，根节点是通过 `new Vue()` 生成的Vue对象实例，单文件组件是Vue子对象的配置，最终会注册为Vue子对象；那么Vue对象究竟是什么，其生成的实例又有什么，一起来看下

Vue对象是一个 **构造函数**，可以通过new的方式实例化，函数自身也是一个对象，Vue对象也不例外，并且在引入时已经扩展了很多

### config

Vue对象本身定义了一个config属性，用于全局配置Vue的一些行为，后续都会依照这些行为进行渲染更新等操作，不支持全替换，只支持 `Vue.config.xx` 的方式修改，在应用启动前配置，可用配置见官网 [全局配置]([https://cn.vuejs.org/v2/api/#全局配置](https://cn.vuejs.org/v2/api/#%E5%85%A8%E5%B1%80%E9%85%8D%E7%BD%AE))

### api

Vue对象本身扩展了一些全局api，用于提供一些方法实现系统内部提供的功能，这些api通过 `initGlobalAPI` 设置进去，可见官网[全局API]([https://cn.vuejs.org/v2/api/#全局-API](https://cn.vuejs.org/v2/api/#%E5%85%A8%E5%B1%80-API))

### prototype

Vue对象还在原型上扩展了一些属性和方法，使得实例化后的对象都可以直接调用这些属性和方法，用于操作实例实现系统内部通过的功能。这些是在 `xxMixin` 时设置进去的，常以 `$` 开头, 可见官网[实例xx]([https://cn.vuejs.org/v2/api/#实例-property](https://cn.vuejs.org/v2/api/#%E5%AE%9E%E4%BE%8B-property))

### options

Vue对象本身有一个全局的options对象，在 `initGlobalAPI` 时设置。Vue对象实例化时，支持传入一个options，通过合并全局配置生成Vue实例。这里分两种情况

一. 手动调用

手动调用即 `new Vue()` ,合并过程比较简单，通过 `mergeOptions` 方法，将全局options, 传入options, 继承/混入options 进行合并，生成到实例的 `$options`属性下，供后面使用，生成VNode对象实例，结果如下

```jsx
vm.$options = {
  components: { },
  created: [
    function created() {
      console.log('parent created')
    }
  ],
  directives: { },
  filters: { },
  _base: function Vue(options) {
    // ...
  },
  el: "#app",
  render: function (h) {
    //...
  }
}
```

二. 组件调用

组件调用即注册并使用组件，分为注册和使用两个阶段；

- 注册时，通过 `mergeOptions` 方法，将全局options，组件定义的options，继承/混入options合并，设置到子对象 `options` 属性中
- 使用时，通过 `initInternalComponent` 方法，将子对象options设置为实例的 `$options` 属性的原型；将父VNode对象实例，父Vue对象实例，设置到实例的 `$options` 中，供后面使用，生成VNode对象实例，结果如下

```jsx
vm.$options = {
  parent: Vue /*父Vue实例*/,
  propsData: undefined,
  _componentTag: undefined,
  _parentVnode: VNode /*父VNode实例*/,
  _renderChildren:undefined,
  __proto__: {
    components: { },
    directives: { },
    filters: { },
    _base: function Vue(options) {
        //...
    },
    _Ctor: {},
    created: [
      function created() {
      }
    ],
    mounted: [
      function mounted() {
      }
    ],
    data() {
       return {
         msg: 'Hello Vue'
       }
    },
    template: '<div>{{msg}}</div>'
  }
}
```

支持的options可见官网[选项xx]([https://cn.vuejs.org/v2/api/#选项-数据](https://cn.vuejs.org/v2/api/#%E9%80%89%E9%A1%B9-%E6%95%B0%E6%8D%AE))

## VNode对象

VNode对象在开发过程中，一般情况下很少直接接触，它主要是通过Vue对象，template/render生成，由Vue对象管理。但其是Vue的重要组成部分，通俗来讲，一般叫它 `Virtual DOM`，是用一个原生的JS数据对象去描述一个DOM节点的一种技术

### 生成

那么我们怎么能够访问到呢？

它作为一个附属属性被包含在 `Vue对象` 中，在生成Vue对象过程中通过 `render` 函数生成，如果是写的 `template`,则会被转换为render函数

在Vue对象生成过程中，会创建两个VNode类型属性，他们分别在不同的vue对象中生成的，具体过程如下

- `$vnode`：开始渲染，`new Vue()`会生成当前节点和子节点，如果子节点是组件，会生成tag为`vue-component-${id}-${name}`的vnode，然后会通过vnode生成组件实例，在组件实例中生成的vnode则会继承部分父vnode的属性，且设置$vnode为父vnode。
- `_vnode`：通过_render()生成的当前需要渲染的vnode，tag为模板的根节点

所以，理论上一个组件有两个vnode，通过分散在不同的vue实例组装为树结构

### 数据对象

VNode对象都是通过createElement函数生成的，其接受3个参数

- tag/component:  需要创建的节点，可以是一个html tag，一个组件选项对象
- data：节点的数据对象，包含了节点需要的各种属性样式事件等
- children：该节点下的子节点

在template中，数据对象通过各种指令的方式简化开发者使用，在不知觉的过程中传入了数据对象，而子节点则直接通过html嵌套的方式传入

在render函数中，则需要显式的传入，通过 `jsx` 的方式可以同时使用javascript的灵活和html的嵌套语法编写render函数，其中支持的数据对象可以通过官网[深入数据对象]([https://cn.vuejs.org/v2/guide/render-function.html#深入数据对象](https://cn.vuejs.org/v2/guide/render-function.html#%E6%B7%B1%E5%85%A5%E6%95%B0%E6%8D%AE%E5%AF%B9%E8%B1%A1))