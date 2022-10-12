---
title: 浏览器的观察者们—Observer API
date: 2022-08-03 23:44:05
cover: 'https://img-blog.csdnimg.cn/img_convert/a216eea9e8a7a513097d9d3f2024f72b.png'
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
`基础语法`
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
    rootMargin: "0px", // 使用类似于设置CSS边距的语法来指定根边距（根元素的观察影响范围）
    threshold: 0.3 // 阈值，可以为数组。[0.3]意味着，当目标元素在根元素指定的元素内可见30%时，调用处理函数。
})

// 定义要观察的目标对象
const target = document.querySelector(".target");
intersectionObserver.observe(target);
```
`实现图片懒加载`
```html
<div class="scrollWrap">
    <img src="loading.gif" data-src="img1.jpg">
    <img src="loading.gif" data-src="img2.jpg">
    <img src="loading.gif" data-src="img3.jpg">
</div> 


```
```javascript
<script>
    let observerImg = new IntersectionObserver((entries,observer) => {
        entries.forEach(entry => {
            // 进入可视区就替换为真实图片路径
            entry.target.src = entry.target.dataset.src
            // 假如我们这还有一个曝光埋点的需求，也可在此处理埋点逻辑
            if(entry.intersectionRatio === 1){
                // intersectionRatio === 1说明该观察元素完全暴露出来
                // 上报埋点逻辑
            }
            // 停止监听
            observer.unobserve(entry.target)
        })
    },{
        root: document.querySelector('.scrollWrap'),
        rootMargin: "0px 0px -200px 0px", // 在到达视口距离底部200px时加载图片
        threshold: 0.3 // 设置当一张图片的30%进入根元素才加载真实图片
    })
    // 定义要观察的目标对象
    document.querySelectorAll('img').forEach(img => observerImg.observe(img))
</script>
```
### ResizeObserver
    ResizeObserver 接口可以监听到 Element 的内容区域或 SVGElement的边界框改变。内容区域则需要减去内边距padding。

`监听body的size变化`
```javascript
const resizeObserver = new ResizeObserver(entries => {
  for (let entry of entries) {
    console.log(entry)
  }
});
resizeObserver.observe(document.querySelector('body'));
```

### PerformanceObserver
    PerformanceObserver是浏览器内部对Performance实现的观察者模式
这是一个浏览器和Node.js 里都存在的API，采用相同W3C的Performance Timeline规范
<p style="color: #999;font-size:14px;">持续更新中</p>