---
title: Vue webpack boilerplate 的一些捣腾
date: 2017-02-06 02:30
tags: 
- vue
---

最近在公司内部推 `Vuejs`，`vue-cli` webpack 生成的项目无法满足现有的需求，于是乎开始改造 `vue-cli` 初始化的项目, 主要解决项目多页面和外部引入样式文件没有 autoprefixer 的问题。

<!--more-->

## 多页面改造

vue webpack 模板默认是 SPA 项目，改成多页面则需要把单入口变为多入口，生成多个html文件。

### 项目结构

<pre>
|—— src
  |—— assets
  |—— components      // 多个页面间的公共组件
  |—— pages 
    |—— hello
      |—— hello.vue   // 页面主vue文件,和文件夹一致方便搜索
      |—— index.html  // html模板文件
      |—— main.js     // 入口文件
</pre>

**pages** 文件夹下每个页面都新建一个文件夹，固定三个文件 `index.html` 、 `main.js` 、 `page.vue` ， 后续会有代码来识别产生多入口

### 构建代码修改

构建逻辑部分需要自定识别 **pages** 下的文件夹，生成多个 `entry` 。

首先，增加一个 `page.js` 初始化所有的页面信息

```javascript
// page.js
var glob = require('glob');
var path = require('path');
var fs = require('fs');
var chalk = require('chalk');

var ENTRY = 'main.js';
var PAGE_FLODER = path.resolve(__dirname, '../src/pages/');
var PAGES = PAGE_FLODER + '/**/' + ENTRY;
var TEMPLATE = 'index.html';

function initPages() {
    var pages = {};
    glob.sync(PAGES).forEach(function (entry) {
        var pageName = getPageName(entry);
        var templatePath = getTemplatePath(entry);
        if (!fs.existsSync(templatePath)) {
            console.log(chalk.yellow(pageName + '找不到模板文件'));
            return;
        }
        
        pages[pageName] = {
            name: pageName,
            entry: entry,
            template: templatePath
        };
    });

    return pages;
}

function getPageName(filePath) {
    return filePath.substring(PAGE_FLODER.length + 1, filePath.indexOf(ENTRY) - 1);
}

function getTemplatePath(filePath) {
    var templatePath = filePath.substring(0, filePath.indexOf(ENTRY));
    templatePath += TEMPLATE;
    return templatePath;
}

module.exports = initPages();

```

然后 修改 `config/index.js`

```javascript
module.exports = {
    pages: require('./page'),   // 增加page的配置
    // ...
};

```

接下，`utils.js` 增加两个函数用来生成多个 `entry` 和 `HtmlWebpackPlugin`

```javascript
// utils.js
//...
exports.getEntry = function () {
    var entry = {};
    Object.keys(config.pages).forEach(function (name) {
        entry[name] = config.pages[name].entry;
    });
    return entry;
}

exports.getHtmlPlugin = function () {
    var assetsSubDirectory = process.env.NODE_ENV === 'production' ?
        config.build.assetsSubDirectory :
        config.dev.assetsSubDirectory;
    return Object.keys(config.pages).map(function (name) {
        return new HtmlWebpackPlugin ({
            template: config.pages[name].template,
            filename: name + '.html',
            inject: true,
            chunks: ['vendor', 'manifest', name],
            minify: env === 'production' ? {
              removeComments: true,
              collapseWhitespace: true,
              removeAttributeQuotes: true
              // more options:
              // https://github.com/kangax/html-minifier#options-quick-reference
            } : undefined,
            // necessary to consistently work with multiple chunks via CommonsChunkPlugin
            chunksSortMode: env === 'production' ? 'dependency' : undefined
        })
    })
}

```

最后改动 `webpack.base.conf.js` 、 `webpack.dev.conf.js` 和 `webpack.prod.conf.js`

```javascript
// webpack.base.conf.js
// ...
module.exports = {
    entry: utils.getEntry() // 多入口配置
    // ...
}
```

```javascript
// webpack.dev.conf.js
// ...
module.exports = merge(baseWebpackConfig, {
    // ...
    plugins: [
        new webpack.DefinePlugin({
            'process.env': config.dev.env
        }),
        // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
        new webpack.optimize.OccurrenceOrderPlugin(),
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoErrorsPlugin()
    ].concat(utils.getHtmlPlugin()),  // 加入生成的多页面 HtmlWebpackPlugin
});
```

```javascript
// webpack.prod.conf.js
//...
webpackConfig.plugins = webpackConfig.plugins.concat(utils.getHtmlPlugin()); // 加上这行代码
//...
```

ok，大功告成， 这样一来就变为多页面项目了（这样一来每个页面至少需要三个文件，自己写了一个atom插件，新建页面时自动生成相关文件和基础代码）。

## js内引入css没有 autoprefixer

`.vue` 文件写的样式都会经过 `postcss` 处理，但是项目中重构和js需要分开工作，所以需要将css文件独立出去，再在入口文件中引入，这是发现没有样式没有进过 `postcss` 处理.

这里需要在样式加载时使用 `postcss-loader` 处理

```
npm install postcss-loader --save-dev
```

```javascript
// utils.js
exports.cssLoaders = function (options) {
    // ...
    return {
        css: generateLoaders(['css', 'postcss']),
        postcss: generateLoaders(['css', 'postcss']),
        less: generateLoaders(['css', 'postcss', 'less']),
        sass: generateLoaders(['css', 'postcss', 'sass?indentedSyntax']),
        scss: generateLoaders(['css', 'postcss', 'sass']),
        stylus: generateLoaders(['css', 'postcss', 'stylus']),
        styl: generateLoaders(['css', 'postcss', 'stylus'])
    };
}
```

```javascript
// webpack.base.conf.js
var postcssConfig = [
    require('autoprefixer')({
        browsers: ['last 5 versions', 'android >= 4.2', 'ios >= 7']
    })
];

module.exports = {
    // ...
    postcss: postcssConfig, // 增加postcss配置
    vue: {
        loaders: utils.cssLoaders({
            sourceMap: useCssSourceMap
        }),
        postcss: postcssConfig
    }
}
```

## 最后

最近又发现 `vue webpack` 模板又升级到了 `webpack 2.2` 马上准备入一波坑了。

