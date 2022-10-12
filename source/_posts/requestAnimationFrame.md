---
title: requestAnimationFrame
date: 2022-07-19 16:07:18
cover: 'https://pic4.zhimg.com/v2-de9936af42274b9b2495a64a6e18600f_r.jpg'
tags: h5
---
{% blockquote %}
requestAnimationFrame是浏览器用于定时循环操作的一个接口，类似于setTimeout,主要用途是按帧对网页进行重绘。
此外，使用这个API，一旦页面不处于浏览器的当前标签，就会自动停止刷新。这就节省了CPU、GPU和电力。不过requestAnimationFrame是在主线程上完成。这意味着，如果主线程非常繁忙，requestAnimationFrame的动画效果会大打折扣。

requestAnimationFrame使用一个回调函数作为参数。这个回调函数会在浏览器重绘之前调用。

requestAnimationFrame polyfill
<script src="https://gist.github.com/paulirish/1579671.js"></script>
{% endblockquote %}

{% codeblock lang:javascript %}
    // exp:按帧更新随机颜色
    changeColor(){
        this.randomColor = `rgb(${Math.floor(Math.random()*256)},${Math.floor(Math.random()*256)},${Math.floor(Math.random()*256)})`;
        this.randomAnimId = requestAnimationFrame(this.changeColor)
    }
    // 手动取消按帧刷新动画
    cancelAnimFrame(){
        cancelAnimationFrame(this.randomAnimId)
    }
{% endcodeblock %}