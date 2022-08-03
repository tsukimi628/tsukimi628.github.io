---
title: 浏览器观察者—IntersectionObserver
date: 2022-08-03 23:44:05
tags:
- 观察者
- 浏览器
- Observer
---
### Observer API
    Observer是浏览器自带的观察者,它主要提供了Intersection, Mutation, Resize, Performance这四类观察者
### IntersectionObserver
{% blockquote %}
IntersectionObserver接口(从属于Intersection Observer API)提供了一种异步观察目标元素与其根(root)(祖先元素或顶级文档视窗(viewport))交叉状态的方法。当一个IntersectionObserver对象被创建时，其被配置为监听根中一段给定比例的可见区域。一旦IntersectionObserver被创建，则无法更改其配置，所以一个给定的观察者对象只能用来监听可见区域的特定变化值；然而，我们可以在同一个观察者对象中配置监听多个目标元素。
{% endblockquote %}
我们可以利用此api异步监听目标元素在文档中的位置变化，并触发相应回调(自定义逻辑处理)。
例如：`长列表无限滚动`
```javascript
// 定义观察者及观察回调
const intersectionObserver = new IntersectionObserver((entries,observer) => {
    // entries:被观察对象列表,一旦被观察对象发生突变就会被移入该列表, 列表中每一项都保留有观察者的位置信息
    // observer: 观察者
    entries.forEach(entry => {
        console.log(entry)
        ...
    })
},{ // 配置监听属性
    root: document.querySelector("#app"),
    rootMargin: "0px", // 计算交叉时添加到根(root)边界盒bounding box的矩形偏移量， 可以有效的缩小或扩大根的判定范围从而满足计算需要
    threshold: 0.5 // 一个包含阈值的列表, 按升序排列, 列表中的每个阈值都是监听对象的交叉区域与边界区域的比率。可以细粒度控制位差引发的交互
})

// 定义要观察的目标对象
const target = document.querySelector(".target");
intersectionObserver.observe(target);
```