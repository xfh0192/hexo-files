---
title: 关于将vue-cli改为多页面配置
date: 2018-05-13 16:50:56
tags: webpack
---


# 关于将vue-cli改为多页面配置

现在大家的前端项目，基本都用webpack了，不仅是css的autoprefixed补充前缀，就连js的新特性都可以用起来了，挺爽的。

使用vue的话，当然也用上咯。但是自己搭一个脚手架实在是比较麻烦，所以还是用现成的会比较好，默认当然是有大佬维护的vue-cli咯。

但是又有了一个问题，项目需要是多页面应用，但是vue-cli默认生成的配置却是单页面的。所以就只能自己去改配置了。

> 以我当前做的项目为例子。目前用的 webpack--3.6.x

## 需要达到的打包结果

vue-cli的话，直接打包的结果：

```
-dist
	├── index.html
	└── static
		├── css
		└── js
```

可以看到，打包出来是单页面的

但是因为当前项目，需要分为mobile和pcmall两个目录，因此我需要的结果：

```
-dist
	├──mobile
	│	└──index.html
	├──pcmall
	│	└──index.html
	└──static
		├── css
		└── js
			├── mobile-index.js
			└── pcmall-index.js
```

可以看到，需要分为不同目录供pc和mobile端使用。

js文件需要添加前缀区分对应的页面，否则名称一致将导致冲突。

## 文件目录

首先放一下当前项目主要的文件目录

```
-build
	|...
	├── utils.js
	├── webpack.base.conf.js
	├── webpack.config.js
	├── webpack.dev.conf.js
	└── webpack.prod.conf.js
-src
	|...
	├── mobile
	|	└── pages
	│       ├── index
	│       │   ├── index.html
	│       │   ├── index.js
	│       │   └── index.vue
	├── pcmall
	|	└── pages
	│       ├── index
	│       │   ├── index.html
	│       │   ├── index.js
	│       │   └── index.vue
	...
```

可以看到，相比cli，build目录中增加了一个webpack.config.js文件。这是将webpack.base.conf.js中的entry项，单独提取出来并加以修改之后存放的地方。

## 开始改配置吧！

首先我们看看cli的entry和output配置。

```
## webpack.base.conf.js

module.exports = {
  entry: {
    app: './src/main.js'
  },
  output: {
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production'
    ? config.build.assetsPublicPath
    : config.dev.assetsPublicPath
  },
  
  ...
 }
```

entry 只有一个文件，表示整个项目从 ./src/main.js 开始建立依赖树。

output path表示打包到哪里，filename代表打包出来的文件名字，publicPath代表引用文件的位置。

我们发现，output.filename 中使用了 [name]，表示将根据entry的key进行命名。

**那么思路就简单了，增加entry就好**

***

### webpack.config.js

1.

我们在 build文件夹中增加webpack.config.js，然后将webpack.base.conf.js的entry改一改

```
## webpack.base.conf.js

...
const webpackConfig = require('./webpack.config.js')

module.exports = {
	entry: webpackConfig.entry,
	...
}

```

2.

接下来，开始写entry配置

> 这里去匹配文件路径的处理，使用了一个npm包 -> [glob](https://www.npmjs.com/package/glob)

```
## webpack.config.js

const webpackConfig = {
    entry: {},
    plugins: []
}

module.exports = webpackConfig;

const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const webpack = require('webpack')
const glob = require('glob')
const utils = require('./utils')

const isProduction = process.env.NODE_ENV === 'production'

 
/*
	获取指定路径下的入口文件
	
	根据目录结构，参数为 src/**/pages/**/*.js
	
	@return entries[Object]
	@example entries = {
		mobile/index : 'src/mobile/pages/index/index.js',
		pcmall/index : 'src/pcmall/pages/index/index.js'
	}
	
*/
const getEntries = function (globPath) {
    let files = glob.sync(globPath)
    let entries = {}

    files.forEach(function (filePath) {
        let split = filePath.split('/')
        let directory = split[split.length - 4];
        let name = split[split.length - 2]

       entries[directory + '/' + name] = './' + filePath;
    })

    return entries;
}

let entries = getEntries('src/**/pages/**/*.js');

Object.keys(entries).forEach(function (name) {
	webpackConfig.entry[name.replace('/', '-')] = entries[name];
})
```

这样，就可以按照'src/\*\*/pages/\*\*/\*.js'这样的约束，将每个页面的entry自动打包了。

但是有个问题，打包一下，诶？我们会发现打包出来的html对不上，还是旧的

3.

这是当然的，cli中的打包，本来就只是打包js，需要打包html还得用插件[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)

我们可以看看webpack.prod.conf.js，看看这个插件怎么用的:

```
## webpack.prod.conf.js

...

 new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      chunks: ['first', 'vendor', 'manifest', 'common'],
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
	   },
      chunksSortMode: 'dependency'
}),

```

其实挺简单的，跟output的配置差不多，主要就是配好filename和template

那么我们首先将上面webpack.prod.conf.js的代码注释一下。

我们调用html-webpack-plugin的地方，是webpack.config.js

```
## webpack.config.js

const webpackConfig = {
    entry: {},
    plugins: []
}

module.exports = webpackConfig;

const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const webpack = require('webpack')
const glob = require('glob')
const utils = require('./utils')

const isProduction = process.env.NODE_ENV === 'production'

/**
	获取指定路径下的入口文件
	
	根据目录结构，参数为 src/**/pages/**/*.js
	
	@return entries[Object]
	@example entries = {
		mobile/index : 'src/mobile/pages/index/index.js',
		pcmall/index : 'src/pcmall/pages/index/index.js'
	}
	
*/
const getEntries = function (globPath) {
    let files = glob.sync(globPath)
    let entries = {}

    files.forEach(function (filePath) {
        let split = filePath.split('/')
        let directory = split[split.length - 4];
        let name = split[split.length - 2]

       entries[directory + '/' + name] = './' + filePath;
    })

    return entries;
}

let entries = getEntries('src/**/pages/**/*.js');

Object.keys(entries).forEach(function (name) {
    
    let fileName = name.split('/')[1];
    let srcPath = entries[name]
    let isPcmall = name.split('/')[0];

    let targetPath = process.env.NODE_ENV === 'production'
        ? utils.assetsHtmlPath(`/${isPcmall}/${fileName}.html`)
        : `${isPcmall}/${fileName}.html`
        
    let templatePath = path.join(__dirname, `../src/${isPcmall}/pages/${fileName}/${fileName}.html`)

    webpackConfig.entry[name.replace('/', '-')] = srcPath

    // 每个页面生成一个html
    let plugin = new HtmlWebpackPlugin({
        // 生成出来的文件名
        filename: targetPath,

        // 每个html的模板
        template: templatePath,

        // 自动将引用插入html
        inject: true,

        // 每个html引用的js模块，也可以在这里加上vendor等公共模块
        chunks: ['manifest', 'vendor', 'common', 'app', name.replace('/', '-')],
    })

    webpackConfig.plugins.push(plugin);
})

```

以上就是完整的webpack.config.js

当做完上述修改后，就达成了我们想要的打包效果了。

- [x] 多页面修改
- [ ] 代码优化

欢迎批评指正。