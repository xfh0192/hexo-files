---
title: typescript - 了解一下配置
date: 2018-07-17 22:20:13
tags: 
- typescript
---

# typescript - 了解一下配置

Typescript 简单来说，就是JavaScript的超集。

百度一下，就知道上面是废话。。那么文档 -> [点这里](https://www.tslang.cn/)

* 假如是初识ts，了解语法的话，先看文档是正确的选择

那么如果已经看了几节，想实际写一下该怎么做呢？

简单介绍一下我摸索到的东西。

## IDE

typescript是微软家的，vscode是推荐ide，所以我是用vscode的 [下载](https://code.visualstudio.com/)

## 建个仓库

假如练手写的ts，想先在浏览器跑一下的话，该怎么办呢？

1. 安装node。因为typescript需要编译为JavaScript才能在浏览器跑，所以至少需要到node环境 [下载](https://nodejs.org/zh-cn/)
2. 利用npm安装typescript
> npm i -g typescript
3. 现在就可以找个位置建一个项目啦。
> cd ~/demo
4. 仓库初始化
> npm init
5. 初始化ts配置
> tsc --init

## hello.ts

上面ts初始化配置后，我们先写一个hello.ts试试编译

```
-- hello.ts --
class hello {
    public sayHello() {
        console.log('hello')
    }
}

let per = new hello()
per.sayHello()
```

保存然后在命令行运行 tsc，我们会看到出现了一个hello.js文件，这就是编译后的js

```
-- hello.js --
"use strict";
var hello = /** @class */ (function () {
    function hello() {
    }
    hello.prototype.sayHello = function () {
        console.log('hello');
    };
    return hello;
}());
var per = new hello();
per.sayHello();

```

可以在浏览器运行一下这个js试试

## 了解简单配置

那么现在，假如想开发项目的话，其实还有很多问题要考虑的，比如配置怎么写、怎么用开源包、怎么调试等。

那么就说说最简单的配置吧

我们可以看文档
[中文](https://www.tslang.cn/docs/handbook/tsconfig-json.html)
[英文](https://www.typescriptlang.org/docs/handbook/compiler-options.html)

```
-- tsconfig.json --
{
    "compilerOptions": {
        /* 指定编译文件的 ECMAScript 版本: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */
        "target": "es5",
        
        /* 指定模块生成方式：'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */
        "module": "commonjs",
        
        /* 是否允许出现js文件 */
        "allowJs": true,
        
        /* sourceMap */
        "sourceMap": false,
        
        /* 输出文件的目录，假如不设置，则编译后js文件会在源ts文件目录下 */
        "outDir": "./",
        
        /* 指定执行编译的入口目录 */
        "rootDir": './',
        
        /* 使用严格模式 */
        "strict": true
    }
}
```

一般来说，配置好以上内容就可以开始开发了。但是有时个别文件位置有点特别，就还需要指定

```
-- tsconfig.json --
{
    "compilerOptions": {...},
    
    /* 包含文件的列表 */
    "files": [
        "app.ts",
        ...
    ]
}
```

或者

```
-- tsconfig.json --
{
    "compilerOptions": {...},
    
    /* 假如需要glob匹配 */
    "include": [
        "src/**/*"
    ],
    "exclude": [
        "node_modules",
        "**/*.spec.ts"
    ]
}
```

支持的glob通配符：
* 匹配0或多个字符（不包括目录分隔符）
* ? 匹配一个任意字符（不包括目录分隔符）
* **/ 递归匹配任意子目录

如果一个glob模式里的某部分只包含*或.*，那么仅有支持的文件扩展名类型被包含在内（比如默认.ts，.tsx，和.d.ts， 如果 allowJs设置能true还包含.js和.jsx）

## 使用npm包

每一个npm包，都是一个JavaScript库，那么想在ts中使用，最好利用上ts的特点，帮助检查类型。

```
一般来讲，你组织声明文件的方式取决于库是如何被使用的。 在JavaScript中一个库有很多使用方式，这就需要你书写声明文件去匹配它们。
```

假如想要在npm上安装包，然后作为ts模块使用的话，就需要有声明文件

首先贴一下查找声明文件的地方 [点这里](https://github.com/Microsoft/TypeSearch)

如果查找到了某个库的声明文件，那么需要安装一下再用(假如安装一个node声明文件)
> npm i --save-d @types/node

默认所有可见的"@types"包会在编译过程中被包含进来。 node_modules/@types文件夹下以及它们子文件夹下的所有包都是可见的； 也就是说， ./node_modules/@types/，../node_modules/@types/和../../node_modules/@types/等等。

***

### 开始踩坑吧

差不多这样，就可以开始码了。