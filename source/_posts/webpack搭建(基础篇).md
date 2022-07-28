---
title: webpack搭建(基础篇)
date: 2022-07-21 19:04:54
cover: 'https://static.ddhigh.com/blog/2020-03-18-032856.png'
tags: 
- webpack
---

#### 从零起步(无三方脚手架)
1 初始化项目，创建package.json用于管理项目信息、依赖库等
```bash
$ npm init
```
2 安装局部的webpack
```bash
$ npm i webpack webpack-cli -D
```
3 使用局部webpack构建
```bash
$ npx webpack
```
4 在package.json中添加scripts脚本，便于脚本执行
{% codeblock lang:javascript %}
"scripts":{
    "build":"webpack"
}
{% endcodeblock %}
```bash
$ npm run build
```
5 在根目录新建webpack.config.js作为webpack配置文件
{% codeblock lang:javascript %}
// webpack.config.js
const path = require('path');
module.exports = {
    entry: "./src/index.js",
    output: {
        filename: "bundle.js", //指定打包文件名
        path: path.resolve(__dirname,"./dist") //借助node中的path模块拼接出打包目标目录的绝对路径
    }
}

如果配置文件名称修改为其他如 xx.config.js,那么还需在脚本处指定修改后的文件名称

// package.json
"scripts":{
    "build":"webpack --config xx.config.js"
}
// 后续执行npm run build即可打包
{% endcodeblock %}

#### 非js类型文件的处理-Loaders
1 处理样式文件 exp: sass-loader(将s[ac]ss转成css)、css-loader(先解析css文件)、style-loader(再创建style标签将样式插入html文档中)
```bash
$ npm i css-loader style-loader -D
```
{% codeblock lang:javascript %}
// webpack.config.js
const path = require('path');
module.exports = {
    entry: "./src/index.js",
    output: {
        filename: "bundle.js", //指定打包文件名
        path: path.resolve(__dirname,"./dist") //借助node中的path模块拼接出打包目标目录的绝对路径
    },
    module:{
        rules:[
            //{rule},
            {
                test:/\.(sass|scss|css)$/, //匹配依赖包中已.css结尾的文件
                // loader语法糖写法
                // loader: "css-loader"
                // 完整写法
                //use:[loader1,loader2] loader->{loader:xxx,[options:str||obj]}
                use: ["style-loader","css-loader","sass-loader"] // 注意书写顺序：loader的执行顺序是从右向左或从后往前
            }
        ]
    }
}
{% endcodeblock %}

css适配工具 <b>PostCSS</b> 通过javascript来转换样式的工具，使用时需要借助于postcss对应的插件
功能：作css的转换和适配，如自动添加浏览器前缀，样式重置
起初：
postcss及配套插件autoprefixer

postcss单独作为工具库使用时：
```bash
$ npm i postcss postcss-cli
$ npm i autoprefixer
$ npx postcss --use autoprefixer -o output.css input.css
```
postcss在webpack中使用配置：
```bash
$ npm i postcss-loader -D
```
{% codeblock lang:javascript %}
// webpack.config.js
use:[
    "style-loader",
    "css-loader",
    {
        loader:"postcss-loader",
        options:{
            postcssOptions: {
                plugins:[
                    require("autoprefixer")()
                ]
            }
        }
    }
]
{% endcodeblock %}
抽离postcss配置到一个单独文件(根目录新建postcss.config.js)：
{% codeblock lang:javascript %}
// postcss.config.js
module.exports = {
    plugins: [
        require("autoprefixer")
    ]
}
// webpack.config.js
use: [
    "style-loader",
    "css-loader",
    "postcss-loader"
]
{% endcodeblock %}

改进：
postcss配合postcss-preset-env插件(能将一些现代css特性转成大多数浏览器认识的css,并且会根据目标浏览器或运行时环境添加所需的polyfill,内置了autoprefixer插件)
```bash
$ npm i postcss-preset-env -D
```
{% codeblock lang:javascript %}
// postcss.config.js
module.exports = {
    plugins:[
        require("postcss-preset-env")
    ]
}
{% endcodeblock %}

2 处理图片、字体文件 <span style="font-size:13px; color:#999">其中图片模块形式为：img的src引入(按模块方式引入如：import/require)；background-image:url()引入</span>
webpack5前处理方式：
    file-loader
```bash
$ npm i file-loader -D
```
{% codeblock lang:javascript %}
// webpack.config.js
module.exports = {
    module:{
        rules:[
            {
                test: /\.(jpe?g|png|gif|svg)$/i,
                use: [
                    {
                        loader: "file-loader"
                    }
                ]
            },
            {
                test: /\.(eot|ttf|woff2?)$/,
                use: [
                    {
                        loader: "file-loader"
                    }
                ]
            }
        ]
    }
}
{% endcodeblock %}

文件命名规则(常见如保留原文件名、扩展名，同时为了防止重名，包含hash值等) loaders/file-loader/placeholders(webpack提供的placeholder)
{% codeblock lang:javascript %}
// file-loader配置改进
{
    test: /\.(jpe?g|png|gif|svg)$/i,
    use: [
        {
            loader: "file-loader",
            options: {
                // outputPath: "image", 打包后的图片资源外包裹一层名为image的文件夹
                name: "image/[name]_[hash:6].[ext]" //运用placeholder将打包后的图片存放在image文件夹中，其中图片资源保留了原文件名及扩展名，同时命名中添加了6位hash值
            }
        }
    ]
}
{% endcodeblock %}

图片资源优化：借助url-loader将小图(因为大图转base64后资源也会很大，所以一般只转小图)转为base64资源(将不会单独打包为一个图片资源而是随js打包到bundle.js中，这意味着减少一次网络请求)
```bash
$ npm i url-loader -D
```
改进：
{% codeblock lang:javascript %}
// webpack.config.js
module.exports = {
    module:{
        rules:[
            {
                test: /.(jpe?g|png|gif|svg)$/i,
                use: { //只有一个loader时可简写为对象形式
                    loader: "url-loader", //替换file-loader
                    options: {
                        name: "image/[name]_[hash:6].[ext]",
                        limit: 60 * 1024 //小于60kb的转为base64编码
                    }
                }
            }
        ]
    }
}
{% endcodeblock %}

webpack5的处理方式：直接使用资源模块类型asset module type(无需安装file-loader、url-loader)
{% codeblock lang:javascript %}
// webpack.config.js
module.exports = {
    module:{
        output: {
            path: path.resolve(__dirname, "./dist"),
            filename: "bundle.js",
            // assetModuleFilename: "image/[name]_[hash:6][ext]"
        }
        rules:[
            {
                test: /.(jpe?g|png|gif|svg)$/i,
                type: "asset",
                generator: {
                    filename: "image/[name]_[hash:6][ext]"  // 命名规则也可在output中assetModuleFilename指定；扩展名ext在webpack5版本中已包含.
                },
                parser: {
                    dataUrlCondition: {
                        maxSize: 60 * 1024
                    }
                }
            },
            {
                test: /\.(eot|ttf|woff2?)$/,
                type: "asset/resource",
                generator: {
                    filename: "font/[name]_[hash:6][ext]"
                }
            }
        ]
    }
}
{% endcodeblock %}

#### webpack功能扩展(打包优化、资源管理、环境变量注入)-Plugin
CleanWebpackPlugin: 自动清除打包目录
HtmlWebpackPlugin: 生成部署所需的入口文件index.html, 可在插件实例中传入自定义模板生成入口文件
```bash
$ npm i clean-webpack-plugin -D
$ npm i html-webpack-plugin -D
```
{% codeblock lang:javascript %}
// webpack.config.js
const path = require('path');
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "./dist"),
        filename: "js/bundle.js"
    },
    module:{},
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin()
    ]
}
{% endcodeblock %}
此处已经实现了npm run build 之后，dist目录自动替换内容，且生成了webpack默认模板入口文件的index.html

进阶：
自定义入口文件index.html模板内容，如下操作：
{% codeblock lang:javascript %}
// webpack.config.js
plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
        template: "./public/index.html" // 此处tempalte即指定public文件夹下index.html为自定义入口文件模板
    })
]

// /public/.index.html
注意事项：html-webpack-plugin插件生成的入口文件模板或引用的入口文件模板采用ejs语法，如有<%= variable %>这类表达式，在配置HtmlWebpackPlugin时，需要填充ejs语法的变量值，这里需要借助webpack内置插件DefinePlugin(因为是内置所以无需安装)

// webpack.config.js
const path = require('path');
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { DefinePlugin } = require("webpack");
module.exports = {
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "./dist"),
        filename: "js/bundle.js"
    },
    module:{},
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            template: "./plugin/index.html",
            variableX: valX, //这里也可以作为index.html入口文件的参数传递，<%= htmlWebpackPlugin.options.variableX %>
        }),
        new DefinePlugin({
            variable: val // 传入入口文件模板中的变量值
        })
    ]
}
{% endcodeblock %}

文件拷贝：
```bash
$ npm i copy-webpack-plugin -D
```
{% codeblock lang:javascript %}
// webpack.config.js
const path = require('path');
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { DefinePlugin } = require("webpack");
const CopyWebpackPlugin = require("copy-webpack-plugin");
module.exports = {
    mode: "development", // 开启开发阶段模式，打包上线前设置为production模式
    devtool: "source-map", // 开启代码映射，便于开发阶段代码调试
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "./dist"),
        filename: "js/bundle.js"
    },
    module:{},
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            template: "./plugin/index.html",
            variableX: valX, //这里也可以作为index.html入口文件的参数传递，<%= htmlWebpackPlugin.options.variableX %>
        }),
        new DefinePlugin({
            variable: val // 传入入口文件模板中的变量值
        }),
        new CopyWebpackPlugin({
            patterns: [
                {
                    from: "public",
                    globOptions: {
                        ignore: [
                            "**/index.html", // 拷贝文件时忽略index.html
                        ]
                    }
                }
            ]
        })
    ]
}
{% endcodeblock %}

#### webpack搭建本地服务-webpack-dev-server
```bash
$ npm i webpack-dev-server -D
```
{% codeblock lang:javascript %}
// package.json
scripts: {
    "build": "webpack",
    "dev": "webpack serve --open" // 此open相当于devServer设置的开启open 
}
// webpack.config.js
module.exports = {
    entry: {},
    output: {},
    devServer:{
        contentBase: './public' // 告知dev server从什么位置查找文件
        hot: true, // 开启HMR webpack-dev-server会另开启一个socket服务，负责主动向客户端发送更新内容
        host: "0.0.0.0", // 和设置localhost的区别后续讲解
        port: 6789,
        open: true,
        compress: true // 开启压缩
        proxy: { // webpack-dev-server帮助转发请求解决本地开发阶段的跨域问题
            '/api': {
                target: "http://xxx:8080",
                pathRewrite: {
                    "^/api": ""
                },
                secure: false, // 无证书也支持转发https请求
                changeOrigin: true // 更新代理后请求的headers中host地址
            }
        }
    }
}
{% endcodeblock %}
devServer运行本地服务时是将文件打包到内存中，开发阶段浏览器向devServer搭建的服务器请求资源时，来源不仅可以是依赖图打包后的资源，也可以是contentBase指定目录的资源；到了生产环境，就需要用copyWebpackPlugin将public目录资源拷贝到目标目录中

#### webpack查找文件路径配置-resolve
{% codeblock lang:javascript %}
// webpack.config.js
module.exports = {
    resolve: {
        extensions: [".js",".json",".mjs",".wasm"] // 这里是默认值，webpack解析到文件时自动添加扩展名
        alias: {  
            "@": path.resolve(__dirname, "./src"), // 配置路径别名

        }
    }
}
{% endcodeblock %}

#### webpack配置(develop、production)模式分离
{% codeblock lang:javascript %}
1.根目录新建config目录专门存放配置文件
2.分别创建webpack.comm.config.js、webpack.dev.config.js、webpack.prod.config.js添加对应配置
// package.json
scripts: {
    "build": "webpack --config ./config/webpack.prod.config.js",
    "dev": "webpack serve --config ./config/webpack.dev.config.js"
}
// webpack.comm.config.js comm是无论开发还是生产都需要的配置,dev、prod配置需要与其合并
module.exports = {
    target: "web",
    ...
}
```bash
$ npm i webpack-merge -D
```
// webpack.dev.config.js
const { merge } from require("webpack-merge");
const commConfig = require("./webpack.comm.config")

module.exports = merge(commConfig, {
    mode: "development",
    devtool: "source-map",
    devServer: {
        ...
    }
})

// webpack.prod.config.js
const { merge } from require("webpack-merge");
const commConfig = require("./webpack.comm.config")

{% endcodeblock %}

真实开发环境中，每次这样从零配置搭建项目显然效率不高，所以通常会使用脚手架帮助我们搭建项目，后续将写到关于脚手架的内容。

#### webpack分包机制
默认情况下，在构建整个组件树的过程中，webpack将依赖图中的组件树打包到一个文件中如app.js, 这样随着项目不断扩大，app.js文件会随之变大，就会造成首屏的渲染速度变慢，这就用到webpack的分包机制：
    对于一些不需要立即使用的模块，我们可以单独拆分为一些小chunk,需要时从服务器加载。项目中使用import()函数导入的模块，函数返回一个promise,从中获取加载到的模块。webpack就对将其从app.bundle.js中提取单独分离出一个打包文件
{% codeblock lang:javascript %}
// import modu from "@/a/modu"   
import("@/a/modu").then(modu => {
    modu()
    ...
})
{% endcodeblock %}
#### webpack的构建流程
webpack的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：
1. 初始化参数
    从配置文件和shell语句中读取，合并得出最终参数
2. 开始编译
    用第一步得到的参数初始化Compiler对象，加载所有配置的插件，执行对象的run方法开始执行编译
3. 确定入口
    根据配置中的entry找出所有的入口文件
4. 编译模块
    从入口文件出发，分析整个项目内各个源文件之间的依赖关系，构建一个模块依赖图 - module graph，然后将 module graph 分离为多个 bundle(entry 所在的 initial bundle、lazy load 需要的 async bundle 和自定义分离规则的 custome bundle)。在构建 module graph 的过程中，会使用 loader 处理源文件，将它们转化为浏览器可识别的 js、css、image、音视频等。调用所有配置的loader对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
5. 完成模块编译
    在经过上一步使用loader翻译所有模块完成后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系
6. 输出资源
    根据入口和模块之间的依赖关系，组装成一个个包含多个模块的chunk,再把每个chunk转换成一个单独的文件加入到输出列表(这一步是可以修改输出内容的最后机会)
7. 输出完成
    在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统
在以上过程中，webpack会在特定时间广播特定事件，插件在侦听到自己该工作的事件后便执行特定逻辑，且插件可以调用webpack提供的api改变webpack的运行结果