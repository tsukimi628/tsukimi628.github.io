---
title: Webpack高级 — CodeSplit & Network Cache
date: 2022-09-10 17:03:54
cover: 'https://img0.baidu.com/it/u=2807815437,1978693260&fm=253&fmt=auto&app=138&f=PNG?w=516&h=500'
tags:
- webpack高级
- Codesplit
- NetworkCache
---
### Code Split
在代码分割前，打包代码时会将所有js文件打包到一个文件中，体积大。所以我们需要将打包生成的文件进行代码分割，生成多个js文件，渲染时仅加载用到的js文件，速度就更快。
- 代码分割(Code Split)：
  - 1. 分割文件：将打包生成的文件进行分割，生成多个js文件
  - 2. 按需加载：用到哪个文件即加载哪个文件

### 实现Code Split
#### 以单入口为例
开发单页面应用(SPA)，单入口的配置：
```js
// webpack.prod.js
module.exports = {
  ...
  output: {
    path: path.resolve(__dirname, "../dist"), //绝对路径
    // 入口文件打包输出文件名
    filename: "static/js/[name].js",
    // 给打包输出的其他文件(非入口文件)命名
    chunkFilename: "static/js/[name].chunk.js"
  },
  optimization: {
    // 压缩
    minimizer: [...
    ],
    // 代码分割配置
    splitChunks: {
      // 1.将node_modules代码分割到一个单独的chunk; 
      // 2.将动态导入的模块单独打包为一个个独立chunk,进而实现按需导入
      chunks: "all"
    }
  }
}
```
使用动态都语法导入模块
```js
// ./js/math.js
export function sum(a,b){
  return a + b
}

// main.js
document.querySelector('.btn').onclick = () => {
  // 使用import动态导入脚本，将其从main.js的chunk分离出来，单独打包到一个bundle中
  // /* webpackChunkName: "math" */ webpack魔法命名，给动态导出的chunk命名为 math,方便见名知意
  import(/* webpackChunkName: "math" */ './js/math.js').then(({ sum }) => {
    ...
    sum(x, y)
  })
}

// 如eslint不能识别动态导入语法，需要额外追加eslint配置
// .eslintrc.js
{
  ...
  plugins: ["import"]
}
```
综上，打包后的效果将是：
```js
/dist
  /static
    /js
      main.js
      math.chunk.js
```

### Network Cache
由 `contenthash`(根据文件内容生成文件hash值)实现缓存最大化后，还有一个值得关注的问题：math.js文件变更后，打包发现依赖math.js的main.js模块生成的打包文件的hash值也变了，可以推测出main.js也发生了改变，但并没有主动修改main.js文件，那是为什么main.js的hash也变了呢？是因为main.js文件引入的math.js文件的hash变化连带引起的，这样main的hash也会变化，缓存将失效。解决该问题的思路：将main.js引入的math.js的hash值单独存到一个文件(runtime)

配置如下：
```js
// webpack.prod.js
{
  ...
  opitimization: {
    // 代码分割
    splitChunks: {
      chunks: "all"
    },
    // 把文件之间依赖的hash值提取生成runtime文件
    runtimeChunk: {
      name: (entrypoint) => `runtime~${entrypoint.name}.js`
    }
  }
}
```
再次修改依赖文件，执行 `npm run build`查看打包后效果，只有math.js和runtime文件会发生变化，保证了其他文件缓存能正常使用
```js
/dist
  /static
    /js
      main.72777.js
      math.chunk.87868.js
      runtime~main.js.56866.js
```

