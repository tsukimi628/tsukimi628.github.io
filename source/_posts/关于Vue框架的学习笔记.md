---
title: 关于Vue框架的学习笔记
date: 2022-07-21 11:47:16
cover: 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.pianshen.com%2Fimages%2F745%2F1a42dd3f984fbe4e92695673fc7d1889.JPEG&refer=http%3A%2F%2Fwww.pianshen.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1661616666&t=8bc9f170f7b6f8ebcfc81d258fffe386'
tags: Vue
---

### v-model
    v-model 本质上不过是语法糖(帮我们完成:v-bind:value的数据绑定和@input/change的事件监听)
    v-model 指令在表单 <input>、<textarea> 及 <select> 元素上创建双向数据绑定

v-model 会忽略所有表单元素的 value、checked、selected attribute 的初始值。始终将当前活动实例的数据作为数据来源。应该通过在组件的 data 选项中声明初始值。

text 和 textarea 元素使用 value property 和 input 事件；
checkbox 和 radio 使用 checked property 和 change 事件；
select 字段将 value 作为 prop 并将 change 作为事件。

#### v-model修饰符-lazy
默认情况下，v-model双向绑定的是input事件，会在每次内容输入后将最新的值和绑定的属性同步，如果有lazy修饰符，那么会将绑定的事件切换为change事件，只在提交时(如回车)才触发同步
<p style="font-size:14px; color:#666;">后续更新v-model在非表单类元素中的使用</p>
### Vue打包后不同版本解析(运行时+编译器 VS 仅运行时)
    vue(.runtime).global(.prod).js
    通过浏览器script标签直接使用，比如通过CDN引入的版本
    会暴露一个全局的Vue

    vue(.runtime).esm-browser(.prod).js
    用于通过原生ES模块导入使用 浏览器<script type="module">引入

    vue(.runtime).esm-bundler.js
    用于webpack、rollup等构建工具，默认用vue.runtime.esm-bundler.js
    若需要解析模板template则需指定vue.esm-bundler.js

    vue.cjs(.prod).js
    服务端渲染使用
    通过require()在node.js中使用

### 双向数据绑定的原理
`Vue是采用数据劫持结合发布-订阅模式的方式，Vue2通过Object.defineProperty()来劫持各个属性的getter、setter(vue3的数据劫持放在下一段解读)，在数据变动时发布消息给订阅者，触发相应的监听回调。`
<b>主要分为以下几个步骤：</b>
1. 需要observe的数据对象进行递归遍历，包括子属性对象的属性，都加上getter、setter. getter中做依赖收集，再给某个属性赋值时就会触发setter, 就能监听到了数据变化
2. compile解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有更新，即收到通知更新视图
3. Watcher订阅者是Observer和Complie之间通信的桥梁，负责(1)在自身实例化时往属性订阅器(dep)里面添加自己；(2)自身必须有一个update方法；(3)待属性变动dep.notice()通知时，能调用自身的update，并触发Compile中绑定的回调
4. MVVM作为数据绑定的入口，整合Observer、Compile和Watcher三者，通过Observer来监听自己的model数据变化，通过Compile来解析编译模板指令，最终利用Watcher搭起Observer和Compile之间的通信桥梁，达到数据变化触发视图更新，视图交互变化触发数据model变更的双向绑定效果。
    
### 通过Object.defineProperty()来进行数据劫持的弊端
    Object.defineProperty不能拦截到给对象新增属性(因为Object.defineProperty拦截的是对象的属性，需要调用VueObj.$set处理这类场景)，以及通过数组下标方式修改数组数据，Vue内部通过重写函数的方式解决了数组响应式这个问题。Vue3通过Proxy劫持对象，实现数据劫持。proxy拦截的是整个对象。
### $nextTick原理
`nextTick是对Javascript执行原理-事件循环(EventLoop)的一种应用。其核心是利用了如Promise、MutationObserver、setImmediate、setTimeout的原生Javascript方法来模拟对应的微、宏任务的实现，利用Javascript的这些异步回调任务队列来实现Vue内部的异步回调队列。`
nextTick不仅是Vue内部的异步队列的调用方法，同时也提供$nextTick api供开发者在开发中使用，对Dom更新数据时机的后续逻辑处理。引用异步更新队列机制的原因：
1. 如果同步更新，则多次对一个或多个属性赋值，会频繁触发UI/DOM的渲染
2. 同时由于VirtualDom的引入，每一次状态发生变化后，状态变化的信号会通知到组件，组件内部使用VirtualDom进行计算得出需要更新的具体Dom节点，后对Dom进行更新操作，每次更新状态后渲染过程需要更多的计算，避免浪费性能，所以异步渲染。Vue采用数据驱动视图思想，但在一些情况下，仍需要直接操作dom。

<b>$nextTick的使用场景</b>
1. 在数据更新后，操作某个需要因数据变化而引发结构变化的Dom
2. 在create生命周期钩子函数中进行Dom操作

