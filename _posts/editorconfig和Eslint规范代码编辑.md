---
title: editorconfig和Eslint规范代码编辑
date: 2016-04-05 19:00:00
tags:
- 前端工具
- eslint
- editorconfig
---
代码规范是一个老生常谈的问题了，在很多人维护一个项目的时候，如何保证每个人的代码风格基本一致呢，每个团队的都会给程序员制定一套开发规范，但是如何去再编写的时候能检测代码时候按照规范编写的呢，这里就需要借助前端工具 **editorconfig** 和 **eslint** 了。
<!--more-->
## [editorconfig](http://editorconfig.org/)
**EditorConfig** 可以帮助开发者在不同的编辑器和IDE之间定义和维护一致的代码风格。**EditorConfig** 包含一个用于定义代码格式的文件和一批编辑器插件，这些插件可以让编辑器读取配置文件并依此格式化代码。 **EditorConfig** 的配置文件十分易读，并且可以很好的在VCS（Version Control System）下工作。
```ini
root = true

[*]
charset = utf-8
end_of_line = lf
indent_size = 4
indent_style = space
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.py]
indent_size = 4

```

> * **root** 表明是最顶层的配置文件，发现设为true时，才会停止查找.editorconfig文件
> * **[]**   配置文件格式
> * **charset**  字符编码
> * **end_of_line** 定义换行符，支持lf、cr和crlf(统计不同系统下的换行符)
> * **indent_size** 代码缩进size
> * **indent_style** 代码缩进方式 **space** / **tab**
> * **insert_final_newline** 在文件最后一行插入空行
> * **trim_trailing_whitespace** 删除多余的空格

使用 **editorcofig**
1. 首先你要安装一个你的编辑的 [插件](http://editorconfig.org/#download) (如果是sublime直接在 **package control** 里搜索 **editorcofig** 就可以了）
2. 然后在项目文件夹下面新建一个 **.editorcofig** 的文件，把上面的配置copy进去（可以根据自己的需要改改）；

**editorcofig** 主要是做文件方面的规范，保证多个系统和ide上的一致的代码风格

## [eslint](http://eslint.org/)
JavaScript是一门「年轻的语言」，因此也就存在很多糟粕的地方，这些糟粕使得程序员在编写JavaScript代码的时候，容易出错，而不易被编辑器或程序员本身发现。于是有个大牛开发了 **jshint** 的工具来检查js代码是否规范，但是，javascript发展太快了，各种各样的写法铺面而来，另一位大牛意识到要对不同的写法开发一套自己的linter规则（插件式开发）,于是就有的现在的 **eslint**

ESLint 主要有以下特点：
默认规则包含所有 JSLint、JSHint 中存在的规则，易迁移；
规则可配置性高：可设置「警告」、「错误」两个 error 等级，或者直接禁用；
包含代码风格检测的规则（可以丢掉 JSCS 了）；
支持插件扩展、自定义规则。

### 安装方法
1.安装node(现在所有的工具都是基于node，不会node都不好意思说自己是前端)

2.全局安装 **eslint** 和 **babel-eslint**
```shell
npm install -g eslint
npm install -g babel-eslint
```

3.属于你编辑器的eslint插件--- 我用的sublime, 只需要安装 **SublimeLinter** 和 **SublimeLinter-csslint** 就好啦

### 配置
​* ESLint主要有两种配置方法

1）Configuration Comment： 通过在所有的JS文件的JavaScript 注释中写ESLint配置信息。

2）Configuration Files: 通过一份配置文件，来对ESLint进行基本配置，配置文件可以是JavaScript、JSON、YAML中任意一种格式。如果是用过.eslintrc 或者在package.son 文件中的eslintConfig属性内进行配置，那么ESLint就能够自动的检测到这两个文件中的配置信息，如果是其他文件或文件名，就需要在命令行中指定特定的配置文件。

* ESLint配置的主要内容
1）Environments： 定义我们脚本的运行环境，配置这项的一个主要原因就是不同的运行环境都会有预先定义的不同的global variables。

2）Globals：配置脚本运行过程中附件的全局变量（global variables）；

3）Rules： 在我们的项目中，需要配置的代码规则，并且指定每一条规则的警告级别（error level）。比如说no-var: 2 这条规则，ESLint在进行代码检测时，如果我们JavaScript代码中使用了var申明变量，那么就会报错，其中2 是一个警告等级，在后面会介绍到。

这里是我的配置，有问题的可以查看这份 [中文文档](https://github.com/Jocs/ESLint_docs)
```JSON

{
  "env": {
    "browser": true,
    "node": true,
    "commonjs": true
  },
  "ecmaFeatures": {
    // lambda表达式
    "arrowFunctions": true,
    // 解构赋值
    "destructuring": true,
    // class
    "classes": true,
    // http://es6.ruanyifeng.com/#docs/function#函数参数的默认值
    "defaultParams": true,
    // 块级作用域，允许使用let const
    "blockBindings": true,
    // 允许使用模块，模块内默认严格模式
    "modules": true,
    // 允许字面量定义对象时，用表达式做属性名
    // http://es6.ruanyifeng.com/#docs/object#属性名表达式
    "objectLiteralComputedProperties": true,
    // 允许对象字面量方法名简写
    /*var o = {
        method() {
          return "Hello!";
        }
     };

     等同于

     var o = {
       method: function() {
         return "Hello!";
       }
     };
    */
    "objectLiteralShorthandMethods": true,
    /*
      对象字面量属性名简写
      var foo = 'bar';
      var baz = {foo};
      baz // {foo: "bar"}

      // 等同于
      var baz = {foo: foo};
    */
    "objectLiteralShorthandProperties": true,
    // http://es6.ruanyifeng.com/#docs/function#rest参数
    "restParams": true,
    // http://es6.ruanyifeng.com/#docs/function#扩展运算符
    "spread": true,
    // http://es6.ruanyifeng.com/#docs/iterator#for---of循环
    "forOf": true,
    // http://es6.ruanyifeng.com/#docs/generator
    "generators": true,
    // http://es6.ruanyifeng.com/#docs/string#模板字符串
    "templateStrings": true,
    "superInFunctions": true,
    // http://es6.ruanyifeng.com/#docs/object#对象的扩展运算符
    "experimentalObjectRestSpread": true
  },

  "rules": {
    // 定义对象的set存取器属性时，强制定义get
    "accessor-pairs": 2,
    // 指定数组的元素之间要以空格隔开(,后面)， never参数：[ 之前和 ] 之后不能带空格，always参数：[ 之前和 ] 之后必须带空格
    "array-bracket-spacing": [2, "never"],
    // 在块级作用域外访问块内定义的变量是否报错提示
    "block-scoped-var": 0,
    // if while function 后面的{必须与if在同一行，java风格。
    "brace-style": [2, "1tbs", { "allowSingleLine": true }],
    // 双峰驼命名格式
    "camelcase": 2,
    // 数组和对象键值对最后一个逗号， never参数：不能带末尾的逗号, always参数：必须带末尾的逗号，
    // always-multiline：多行模式必须带逗号，单行模式不能带逗号
    "comma-dangle": [2, "never"],
    // 控制逗号前后的空格
    "comma-spacing": [2, { "before": false, "after": true }],
    // 控制逗号在行尾出现还是在行首出现
    // http://eslint.org/docs/rules/comma-style
    "comma-style": [2, "last"],
    // 圈复杂度
    "complexity": [2, 9],
    // 以方括号取对象属性时，[ 后面和 ] 前面是否需要空格, 可选参数 never, always
    "computed-property-spacing": [2,"never"],
    // 强制方法必须返回值，TypeScript强类型，不配置
    "consistent-return": 0,
    // 用于指统一在回调函数中指向this的变量名，箭头函数中的this已经可以指向外层调用者，应该没卵用了
    // e.g [0,"that"] 指定只能 var that = this. that不能指向其他任何值，this也不能赋值给that以外的其他值
    "consistent-this": 0,
    // 强制在子类构造函数中用super()调用父类构造函数，TypeScrip的编译器也会提示
    "constructor-super": 0,
    // if else while for do后面的代码块是否需要{ }包围，参数：
    //    multi  只有块中有多行语句时才需要{ }包围
    //    multi-line  只有块中有多行语句时才需要{ }包围, 但是块中的执行语句只有一行时，
    //                   块中的语句只能跟和if语句在同一行。if (foo) foo++; else doSomething();
    //    multi-or-nest 只有块中有多行语句时才需要{ }包围, 如果块中的执行语句只有一行，执行语句可以零另起一行也可以跟在if语句后面
    //    [2, "multi", "consistent"] 保持前后语句的{ }一致
    //    default: [2, "all"] 全都需要{ }包围
    "curly": [2, "all"],
    // switch语句强制default分支，也可添加 // no default 注释取消此次警告
    "default-case": 2,
    // 强制object.key 中 . 的位置，参数:
    //      property，'.'号应与属性在同一行
    //      object, '.' 号应与对象名在同一行
    "dot-location": [2, "property"],
    // 强制使用.号取属性
    //    参数： allowKeywords：true 使用保留字做属性名时，只能使用.方式取属性
    //                          false 使用保留字做属性名时, 只能使用[]方式取属性 e.g [2, {"allowKeywords": false}]
    //           allowPattern:  当属性名匹配提供的正则表达式时，允许使用[]方式取值,否则只能用.号取值 e.g [2, {"allowPattern": "^[a-z]+(_[a-z]+)+$"}]
    "dot-notation": [2, {"allowKeywords": true}],
    // 文件末尾强制换行
    "eol-last": 2,
    // 使用 === 替代 ==
    "eqeqeq": [2, "allow-null"],
    // 方法表达式是否需要命名
    "func-names": 0,
    // 方法定义风格，参数：
    //    declaration: 强制使用方法声明的方式，function f(){} e.g [2, "declaration"]
    //    expression：强制使用方法表达式的方式，var f = function() {}  e.g [2, "expression"]
    //    allowArrowFunctions: declaration风格中允许箭头函数。 e.g [2, "declaration", { "allowArrowFunctions": true }]
    "func-style": 0,
    "generator-star-spacing": [2, { "before": true, "after": true }],
    "guard-for-in": 0,
    "handle-callback-err": [2, "^(err|error)$" ],
    "indent": [2, 2, { "SwitchCase": 1 }],
    "key-spacing": [2, { "beforeColon": false, "afterColon": true }],
    "linebreak-style": 0,
    "lines-around-comment": 0,
    "max-nested-callbacks": 0,
    "new-cap": [2, { "newIsCap": true, "capIsNew": false }],
    "new-parens": 2,
    "newline-after-var": 0,
    "no-alert": 0,
    "no-array-constructor": 2,
    "no-caller": 2,
    "no-catch-shadow": 0,
    "no-cond-assign": 2,
    "no-console": 0,
    "no-constant-condition": 0,
    "no-continue": 0,
    "no-control-regex": 2,
    "no-debugger": 2,
    "no-delete-var": 2,
    "no-div-regex": 0,
    "no-dupe-args": 2,
    "no-dupe-keys": 2,
    "no-duplicate-case": 2,
    "no-else-return": 0,
    "no-empty": 0,
    "no-empty-character-class": 2,
    "no-empty-label": 2,
    "no-eq-null": 0,
    "no-eval": 2,
    "no-ex-assign": 2,
    "no-extend-native": 2,
    "no-extra-bind": 2,
    "no-extra-boolean-cast": 2,
    "no-extra-parens": 0,
    "no-extra-semi": 0,
    "no-fallthrough": 2,
    "no-floating-decimal": 2,
    "no-func-assign": 2,
    "no-implied-eval": 2,
    "no-inline-comments": 0,
    "no-inner-declarations": [2, "functions"],
    "no-invalid-regexp": 2,
    "no-irregular-whitespace": 2,
    "no-iterator": 2,
    "no-label-var": 2,
    "no-labels": 2,
    "no-lone-blocks": 2,
    "no-lonely-if": 0,
    "no-loop-func": 0,
    "no-mixed-requires": 0,
    "no-mixed-spaces-and-tabs": 2,
    "no-multi-spaces": 2,
    "no-multi-str": 2,
    "no-multiple-empty-lines": [2, { "max": 1 }],
    "no-native-reassign": 2,
    "no-negated-in-lhs": 2,
    "no-nested-ternary": 0,
    "no-new": 2,
    "no-new-func": 0,
    "no-new-object": 2,
    "no-new-require": 2,
    "no-new-wrappers": 2,
    "no-obj-calls": 2,
    "no-octal": 2,
    "no-octal-escape": 2,
    "no-param-reassign": 0,
    "no-path-concat": 0,
    "no-process-env": 0,
    "no-process-exit": 0,
    "no-proto": 0,
    "no-redeclare": 2,
    "no-regex-spaces": 2,
    "no-restricted-modules": 0,
    "no-return-assign": 2,
    "no-script-url": 0,
    "no-self-compare": 2,
    "no-sequences": 2,
    "no-shadow": 0,
    "no-shadow-restricted-names": 2,
    "no-spaced-func": 2,
    "no-sparse-arrays": 2,
    "no-sync": 0,
    "no-ternary": 0,
    "no-this-before-super": 2,
    "no-throw-literal": 2,
    "no-trailing-spaces": 2,
    "no-undef": 2,
    "no-undef-init": 2,
    "no-undefined": 0,
    "no-underscore-dangle": 0,
    "no-unexpected-multiline": 2,
    "no-unneeded-ternary": 2,
    "no-unreachable": 2,
    "no-unused-expressions": 0,
    "no-unused-vars": [2, { "vars": "all", "args": "none" }],
    "no-use-before-define": 0,
    "no-var": 0,
    "no-void": 0,
    "no-warning-comments": 0,
    "no-with": 2,
    "object-curly-spacing": 0,
    "object-shorthand": 0,
    "one-var": [2, { "initialized": "never" }],
    "operator-assignment": 0,
    "operator-linebreak": [2, "after", { "overrides": { "?": "before", ":": "before" } }],
    "padded-blocks": 0,
    "prefer-const": 0,
    "quote-props": 0,
    "quotes": [2, "single", "avoid-escape"],
    "radix": 2,
    "semi": [2, "never"],
    "semi-spacing": 0,
    "sort-vars": 0,
    "space-after-keywords": [2, "always"],
    "space-before-blocks": [2, "always"],
    "space-before-function-paren": [2, "always"],
    "space-in-parens": [2, "never"],
    "space-infix-ops": 2,
    "space-return-throw-case": 2,
    "space-unary-ops": [2, { "words": true, "nonwords": false }],
    "spaced-comment": [2, "always", { "markers": ["global", "globals", "eslint", "eslint-disable", "*package", "!"] }],
    "strict": 0,
    "use-isnan": 2,
    "valid-jsdoc": 0,
    "valid-typeof": 2,
    "vars-on-top": 0,
    "wrap-iife": [2, "any"],
    "wrap-regex": 0,
    "yoda": [2, "never"]
  }
}

```
