---
title: Babel
date: 2022-07-22 23:02:07
cover: 'https://cache.yisu.com/upload/information/20200720/151/49209.jpg'
tags: 
- babel
- webpack
---
### Babel 是一个 JavaScript 编译器
Babel是一个工具链，主要用于旧浏览器或者环境中将ECMAScript 2015+ 代码转换为向后兼容版本的Javascript;包括：语法转换、缺失特性的修补、源码转换等。
学习Babel对于理解代码从编写到线上的转变过程至关重要
Babel提供了插件化的功能，一切功能都可以以插件来实现，方便使用、启用

`Babel的核心库及相关插件如：`
@babel/babel-core
@babel/plugin-transform-arrow-function
@babel/plugin-transform-block-scoping
...
@babel/preset-env 是替代安装一个个插件的插件集合预设

<b>Babel的处理步骤：</b>
把我们编写的源代码转成浏览器识别的另一种源码，这可以把Babel看做是编译器<span style="color:#999; font-size: 14px;">(github一个js实现的编译器：https://github.com/jamiebuilds/the-super-tiny-compiler)</span>
Babel的工作流程：解析(parse)-转换(transform)-生成(generate)
1. 解析-接收代码并输出AST

- 词法分析
    - 把字符串形式的代码转换为令牌(tokens)流(可以看做是一个扁平的语法片段数组)
{% codeblock lang:javascript %}
    a * b
    // 转换成tokens后如下
    [
        {type:{...}, value:"a", start:0, end:1, loc:{...}},
        {type:{...}, value:"*", start:2, end:3, loc:{...}},
        {type:{...}, value:"b", start:4, end:5, loc:{...}},
        ...
    ]
{% endcodeblock %}
- 语法分析
    - 把令牌(tokens)流转换成AST的形式，此阶段会使用令牌中的信息把它们转换成一个抽象语法树AST的表述结构，这样更易于后续操作
<div style="color:#555;font-size:15px;font-weight:bold;">抽象语法树AST</div>    
    <p style="background:#efefef; color:#666; font-size: 14px;">一种描述用于静态分析的程序语法，https://astexplorer.net/查看AST</p>
字符串形式的type字段表示节点的类型<span style="color:#888; font-size: 14px;">(如："FunctionDeclaration"，"Identifier",或"BinaryExpression")</span>。每一种类型的节点定义了一些附加属性用来进一步描述该节点类型。
Babel还为每个节点额外生成了一些属性，用于描述该节点在原始代码中的位置，如start end
<br>
2. 转换-接收AST并对其遍历，在此过程中对节点进行添加、更新、移除等操作。这是Babel或是其他编译器中最复杂的环节，同时插件也将在该阶段介入工作
Babylon 是 Babel 的解析器；Babel Traverse（遍历）模块维护了整棵树的状态，并且负责替换、移除和添加节点；

<br>
3. 生成-代码生成步骤把最终(转换后)的AST转换成字符串形式的代码，同时还会创建源码映射(source maps)
Babel Generator模块是 Babel 的代码生成器，它读取AST并将其转换为代码和源码映射（sourcemaps）
### babel在webpack中的使用
```bash
$ npm i @babel/core babel-loader -D
$ npm i @babel/preset-env
```
{% codeblock lang:javascript %}
// webpack.config.js
{
    test: /\.js$/,
    use: [
        {
            loader: "babel-loader",
            <!-- options: {
                presets: [   // webpack根据预设来加载对应的插件列表传递给babel
                    ["@babel/preset-env"]
                ]
            } -->
        }
    ]
}

将babel配置单独抽取到一个配置文件
// babel.config.js
module.exports = {
    presets: ["@babel/preset-env"]
}
{% endcodeblock %}

### 简单编写一个babel插件
1. 一个插件就是一个函数
{% codeblock lang:javascript %}
export default function(babel){

}
// babel里我们主要用到types属性
export default function({types:t}){

}
Babel Types模块是一个用于 AST 节点的 Lodash 式工具库， 它包含了构造、验证以及变换 AST 节点的方法。该模块拥有每一个单一类型节点的定义，包括节点包含哪些属性，什么是合法值，如何构建节点、遍历节点，以及节点的别名等信息

// 单一节点类型的定义形式如下：
defineType("BinaryExpression",{
    builder: ["operator","left","right"],
    fields: {
        operator: {
            validate: assertValueType("string")
        },
        left: {
            validate: assertNodeType("Expression")
        },
        right: {
            validate: assertNodeType("Expression")
        }
    },
    visitor: ["left","right"],
    aliases: ["Binary","Expression"]
});

{% endcodeblock %}
2. 返回一个对象
{% codeblock lang:javascript %}
visitor属性是这个插件的主要访问者，visitor中的每个函数都接收两个参数state和path
export default function({types: t}){
    return {
        visitor: {

        }
    }
}
AST通常会有许多节点，那么节点之间如何相互关联呢？我们可以使用一个可操作和访问的可变对象表示节点之间的关联关系，或者也可以用Paths(路径)来简化这件事。
path是表示两个节点之间连接的对象
// 将子节点Identifier表示为一个路径(path)
{
    "parent": {
        "type": "FunctionDeclaration",
        "id": {...},
        ...
    },
    "node":{
        "type": "Identifier",
        "name": "square"
    }
}
{% endcodeblock %}
3. 创建plugin.js
```bash
$ npm i @babel/core
$ npm i babel-template
``` 
{% codeblock lang:javascript %}
babel-template 生成字符串形式且带有占位符的代码来代替手动编码
const template = require('babel-template');
const temp = template("var n = 1");
module.exports = function({ type: t}){
    // 插件内容
    return {
        visitor: {
            // 接收2个参数path state
            variableDeclaration(path, state){
                // 获取AST节点
                const node = path.node;
                // 判断节点类型是否是变量节点，申明方式是const
                if(t.isVariableDeclaration(node, {kind: "const"})){
                    // 将const声明编译为let
                    node.kind = "let"
                    // var n = 1的AST节点
                    const insertNode = temp();
                    // 插入一行代码 var n = 1
                    path.insertBefore(insertNode);
                }
            }
        }
    }
}
{% endcodeblock %}
4. 使用插件
{% codeblock lang:javascript %}
// babel.js
const definedPlugin = require('./plugin');
const babel = require('@babel/core');
const content = 'const name = tsukimi';
const { code } = babel.transform(content, {plugins: [definedPlugin]});
console.log(code)
{% endcodeblock %}