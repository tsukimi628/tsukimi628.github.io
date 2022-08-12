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
1. 安装prettier(脚手架初始项目时也可以选择安装)
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
"scripts": {
    ...
    "prettier": "prettier --write ."
}
{% endcodeblock %}
然后执行
```bash
$ npm run prettier
```
#### 使用Eslint检测
我们用脚手架初始项目时，一般选择安装过了eslint，所以默认会生成一份eslint配置文件.eslintrc.js
VSCode编译器需要安装Eslint插件
如有elsint/prettier配置冲突的地方，则按如下解决：
安装插件(同上，如果脚手架初始化项目时选择过了安装prettier和eslint, 这里就不用单独安装了)：
```bash
$ npm i eslint-plugin-prettier eslint-config-prettier -D
```
往eslint配置文件追加prettier
```javascript
extends: {
    ...
    +"plugin: prettier/recommended"
}
```
有些情况我们希望按照本来的意愿而不是eslint提示的那样去编码，如：我们在vue.config.js里用cjs方式引入模块，eslint会提示相关警告"Require statement not part of import statement.eslint(@typescript-eslint/no-var-requires)", 这时我们可以手动配置让eslint关闭这个校验，如下：
```javascript
// .eslintrc.js
rules: {
    ...
    +"@typescript-eslint/no-var-requires": "off"
}
```

#### Git提交代码监测
    husky是一个git hook工具，为我们提供pre-commit、commit-msg、pre-push这些阶段钩子
键入husky自动化配置命令:
```bash
$ npx husky-init && npm i
```
将会安装husky并在根目录下创建.husky文件夹(存放husky配置),在package.json中添加prepare脚本
指定.husky/pre-commit配置:
{% codeblock %}
// /.husky/pre-commit
npm run lint
{% endcodeblock %}
然后我们再git commit时就可以看到husky帮我们完成 auto-fixed工作了
#### 规范化Git commit messgae
    Commitizen 是一个帮助我们编写规范 commit message 的工具, 规范的commit msg 形如"fix(scope):description
安装Commitizen:
```bash
$ npm i commitizen -D
# 安装cz-conventional-changelog作为commitizen的适配器
$ npx commitizen init cz-conventional-changelog --save-dev --save-exact
```
提交流程：
```bash
$ npx cz
```
将会看到让我们选择提交类型的交互界面
确定相关选项后 git add .  npx cz
顺便改造下cz命令，在package.json里:
```json
"scripts": {
    "commit": "cz"
}
```
为确保使用Commitizen提交而非一般Git提交流程
1. 我们需要安装以下工具:
```bash
$ npm i @commitlint/config-conventional @commitlint/cli -D
```
2. 在根目录添加commitlint.config.js文件，配置commitlint
```javascript
module.exports = {
    extends: ['@commitlint/config-conventional']
}
```
3. 使用husky生成commit meaasge文件，验证提交信息：
```bash
$ npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"
```
如：执行git commit -m"不合规msg",则会阻止提交，这时执行npm run commit就会进到提交信息交互界面，按需选择往下走即可
