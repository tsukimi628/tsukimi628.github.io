---
title: Vue3带来的新变化
date: 2022-07-20 15:28:19
cover: 'https://picx.zhimg.com/v2-d9932822b90cd782a3dc85c8a288330e_1440w.jpg?source=172ae18b'
tags: vue3
---
## 源码层面
#### 源码通过monorepo的形式来管理源码
Mono:单个
主要是将许多项目的代码存储在同一个repository中
目的是保持多个包互相独立，但可以有自己的功能逻辑、单元测试等，同时又属同一个仓库下方便管理，模块划分更加清晰、可维护性、可扩展性更强

#### 源码使用typescript进行重写
vue2使用flow进行类型检查
vue3对ts支持更好了

## 性能层面
#### vue3使用proxy进行数据劫持
    vue2的defineProperty
#### 移除部分api
    实例上：$on $once $off $children
    特性：filter 内联模板
#### 编译优化
生成Block tree、slot编译优化、diff算法优化

## 新的api
#### composition api
#### hooks函数增加代码的复用性
    vue2中通过mixins实现共享组件逻辑，其缺陷是会存在命名冲突的问题，到了vue3，可以通过hooks函数实现抽取共享逻辑，并可以做到响应式

<p style="color: #999;font-size:14px;">持续更新中</p>
