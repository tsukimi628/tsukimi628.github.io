---
title: Vue3带来的新变化
date: 2022-07-20 15:28:19
cover: 'https://picx.zhimg.com/v2-d9932822b90cd782a3dc85c8a288330e_1440w.jpg?source=172ae18b'
tags: vue3
---
### 源码层面
#### 源码通过monorepo的形式来管理源码
Mono:单个
主要是将许多项目的代码存储在同一个repository中
目的是保持多个包互相独立，但可以有自己的功能逻辑、单元测试等，同时又属同一个仓库下方便管理，模块划分更加清晰、可维护性、可扩展性更强

#### 源码使用typescript进行重写
vue2使用flow进行类型检查
vue3对ts支持更好了

### 性能层面
#### vue3使用proxy进行数据劫持
    vue2的defineProperty
#### 移除部分api
    实例上：$on $once $off $children
    特性：filter 内联模板
#### 编译优化
生成Block tree、slot编译优化、diff算法优化

### 新的api
#### composition api
#### hooks函数增加代码的复用性
    vue2中通过mixins实现共享组件逻辑，其缺陷是会存在命名冲突的问题，到了vue3，可以通过hooks函数实现抽取共享逻辑，并可以做到响应式
利用 hooks 结合 customRef api实现对用户输入项响应的节流操作，如下
{% blockquote %}
<b>customRef</b>
创建一个自定义的 ref，并对其依赖项跟踪和更新触发进行显式控制。它需要一个工厂函数，该函数接收 track 和 trigger 函数作为参数，并且应该返回一个带有 get 和 set 的对象。
{% endblockquote %}
{% codeblock lang:javascript %}
//  @/hook/customDebounce.js
import { costomRef } from 'vue'
// 自定义ref
export default function(value, delay = 300){ // value：依赖项
    let timer = null
    return customRef((track, trigger) => {
        return {
            get(){
                // (收集)跟踪依赖
                track();
                return value
            },
            set(newVal){
                clearTimeout(timer)
                timer = setTimeout(()=>{
                    value = newVal
                    // 触发更新
                    trigger()
                }, delay)
            }
        }
    })
}

// sfc.vue
<template>
    <input v-model="msg">
    <p v-text="msg"></p>
</template>
<script setup>
    import customDebounce from '@/hook/customDebounce'
    export default {
        // 使用customDebounce hook显示控制msg的依赖追踪和更新触发
        const msg = customDebounce("hello world")
    }
</script>
{% endcodeblock %}
#### watchEffect清除副作用
{% blockquote %}
有时副作用函数会执行一些异步的副作用，这些响应需要在其失效时清除 (即完成之前状态已改变了) 。所以侦听副作用传入的函数可以接收一个 onInvalidate 函数作入参，用来注册清理失效时的回调。
- 当以下情况发生时，这个失效回调会被触发：
    - 副作用即将重新执行时
    - 侦听器被停止 (如果在 setup() 或生命周期钩子函数中使用了 watchEffect，则在组件卸载时)
{% endblockquote %}

<p style="color: #999;font-size:14px;">持续更新中</p>
