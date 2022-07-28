---
title: 下一代前端开发与构建工具Vite
date: 2022-07-24 18:55:08
cover: 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.zyiz.net%2Fupload%2F202009%2F27%2F202009271403400475.png&refer=http%3A%2F%2Fwww.zyiz.net&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1661617736&t=4f0b86a7091fa9f5c9a7b64eeb9389c7'
tags: 
- 构建工具
- vite
---
Vite在开发模式下借助浏览器对 ESM 的支持，采用 nobundle 的方式进行构建，能提供极致的开发体验；生产模式下则基于 rollup 进行构建
### vite的基本使用
    使用npm init一个简单项目后，安装vite并使用vite启动项目
```bash
$ npm i vite -D
$ npx vite        // 调用项目内部安装的vite启动服务
```
### Vite对TypeScript的支持
    vite原生支持typescript,直接使用esbuild来完成编译
如果查看网站在vite编译后的浏览器中的静态资源请求，会发现请求的依然是后缀名为.ts文件，查看其response会发现内容其实是ts编译后的代码，这是因为vite中的服务器connect会对我们的请求做转发，返回给浏览器的可以直接解析的代码文件。

### Vite对Vue的支持
    vite对vue提供第一优先级支持,在不使用脚手架的情况下需要如下安装：
Vue3单文件组件支持：@vitejs/plugin-vue
Vue3 JSX支持：@vitejs/plugin-vue-jsx
Vue2支持：underfin/vite-plugin-vue2

    以vue3为例：
```bash
$ npm i @vitejs/plugin-vue -D
```
{% codeblock lang:javascript %}
// 根目录新建vite.config.js,在此配置vite插件
import vue from '@vitejs/plugin-vue';
module.exports = {
    plugins: [
        vue()
    ]
}

{% endcodeblock %}

### Vite打包项目
    首先可以简化一下command
{% codeblock lang:javascript %}
// package.json
"scripts":{
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
}
{% endcodeblock %}
    打包并起服务预览打包后的静态文件
```bash
$ npm run build
$ npm run preview
```

### ESBuild解析
- ESBuild特点：
    - 构建速度快，并且不需要缓存
    - 支持ES6/CommonJS模块化
    - 支持ES6的Tree Shaking
    - 支持GO、Javascript的API
    - 支持Typescript、JSX等语法编译
    - 支持SourceMap、代码压缩
    - 支持扩展其他插件
<br>
- ESBuild构建速度快的原因：
    - 因为使用GO编写的，可以直接转换成机器代码，而无需经过字节码
    - ESBuild可以充分利用CPU的多内核，尽可能让其饱和运行
    - ESBuild的所有内容是从零编写，各种性能考虑在内，从而避免使用三方带来的性能问题

<p style="color: #999;font-size:14px;">后续将持续添加vite工具相关学习笔记</p>