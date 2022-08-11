---
title: Vue3+TS项目搭建规范
date: 2022-08-11 22:35:04
tags: 
- 项目搭建
- 规范
- Vue3
- Typescript
---

### 代码规范
#### 集成editorconfig配置 
    官网：https://editorconfig.org/
    EditorConfig有助于为不同的IDE编辑器上多人项目开发维护一致的编码风格
    EditorConfig文件使用INI格式。斜杠(/)做为路径分隔符，#或者;做为注释。
参考Vue官方配置：
{% codeblock %}
# https://editorconfig.org

root = true                       // 当用IDE打开一个文件时，EditorConfig插件会在打开文件的目录和其每一级父节点查找.editorconfig文件，直到找到一个配置了root = true的配置文件

[*]                               // 表示适用所有文件
charset = utf-8                   // 设置文件字符集为utf-8
indent_style = space              // 缩进使用space
indent_size = 2                   // 缩进为space时，缩进的字符数
end_of_line = lf                  // 换行符的类型 lf
insert_final_newline = true       // 使文件以一个空白行结尾
trim_trailing_whitespace = true   // 将行尾空格自动删除

[*.md]                            // 针对markdown文件的设置
insert_final_newline = false
trim_trailing_whitespace = false
{% endcodeblock %}
VSCode编译器需要安装 EditorConfig for VS Code来识别EditorConfig配置

#### 使用prettier代码格式化工具
1. 安装prettier
```bash
$ npm i prettier -D
```
2. 添加.prettierrc配置文件
{% codeblock %}
{
    useTabs: false,       // 使用空格
    tabWidth: 2,          // 缩进为2个空格
    singleQuote: false,   // 使用双引号
    printWidth: 100,      // 限制一行的长度
    trailingComma: 'none' // 省略末尾逗号
    semi: false           // 省略末尾分号
}
{% endcodeblock %}
设置prettier忽略文件.prettierignore
{% codeblock %}
/dist/*
/node_modules/**
**/*.svg
**/*.sh
/public/*
{% endcodeblock %}
安装vscode 插件 prettier code formatter, 尝试保存文件会发现自动格式化已生效, 那如何一键格式化项目所有文件(除忽略文件外)呢?
可以在package.json中添加相关脚本：
{% codeblock %}
"scripts":{
    ...
    "prettier": "prettier --write ."
}
{% endcodeblock %}
然后执行
```bash
$ npm run prettier
```
后续添加eslint
