---
title: 微信小程序 -- 引入promise
date: 2018-08-14 23:30:42
tags: 
- 微信
---

# 微信sdk接口 -- 引入promise

最近进行了一波wx小程序开发，发现不论是h5中还是小程序中，微信sdk提供的接口都是用回调函数的，这就导致了多层回调嵌套的问题。

尤其是在微信支付的流程中，需要异步调用多个接口。那个代码，属于不好维护的那种【。

举例：
```
- pay.js -

// 获取code
wx.login({
    success: function(res) {
        // 将code给服务器获取openid
        wx.request({
            url: xxxxx,
            method: 'POST',
            data: {code: res.code},
            success: function(res) {
                // 调其他的接口或者发请求
                wx.request({
                    url: xxxxx1,
                    data: {},
                    headers: {
                        'content-type': 'application/json''
                    },
                    success: function(res) {
                        ....
                    }
                })
            }
        })
    }
})
```

我觉得吧，还是看看有没有办法引入promise

## 过程

大致步骤都是能查到的。

1. 在项目中引入 [es6-promise](https://github.com/stefanpenner/es6-promise)

初始化一个微信小程序项目。结构如下（已经删掉了/pages/logs/）
```
.
├── app.js
├── app.json
├── app.wxss
├── pages
│   └── index
│       ├── index.js
│       ├── index.json
│       ├── index.wxml
│       └── index.wxss
├── project.config.json
└── utils
    ├── es6-promise.js
    ├── util.js
    └── wx-method.js
```

在/utils中引入es6-promise.js文件，然后wx-method.js则是封装方法存放的地方

在页面中使用的时候当然就是
```
const wxMethod = require('../../utils/wx-method')
```

2. 对wx的api进行promise封装

小程序内容不多，因此只有几个api封装了，如下

```
- wx-method.js -

var Promise = require('./es6-promise')

module.exports = {

    // 获取code【用户登录凭证
    login(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'login',
            options: options
        }))
    },

    // 设置当前页面标题
    setNavigationBarTitle(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'setNavigationBarTitle',
            options: options
        }))
    },

    // 跳转到其他页面
    navigateTo(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'navigateTo',
            options: options
        }))
    },
    // 关闭当前页面，跳转到应用内的某个页面
    redirectTo(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'redirectTo',
            options: options
        }))
    },

    // toast
    showToast(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'showToast',
            options: options
        }))
    },

    // loading
    showLoading(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'showLoading',
            options: options
        }))
    },
    hideLoading(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'hideLoading',
            options: options
        }))
    },

    // 发起请求
    request(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'request',
            options: options
        }))
    },

    // 跳到另一个小程序
    navigateToMiniProgram(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'navigateToMiniProgram',
            options: options
        }))
    },

    // 调起支付
    requestPayment(options = {}) {
        return Promise.resolve(extendWxFn({
            wxMethodName: 'requestPayment',
            options: options
        }))
    }

}


/**
 * 对微信方法调用的统一封装（除部分参数结构不一样的方法，需要在外部做处理
 * @param {string} wxMethodName - 调用的wx包的方法
 * @param {Object} options - 对应不同方法中，包含的参数
 * @param {boolean} options.miniProgram - true:小程序的方法 false: wx包公众号网页的方法
 * @param {function} options.success - 成功回调
 * @param {function} options.fail - 失败回调
 * ps：没有提供complete回调，因为对应逻辑可以在promise链末端实现
 */
function extendWxFn({wxMethodName, options = {}}) {
    return new Promise((resolve, reject) => {
        let successFn = (res) => {
            let success = options.success;
            typeof success === 'function' && success(res);
            return resolve(res);
        }
        let failFn = (res) => {
            let fail = options.fail;
            typeof fail === 'function' && fail(res);
            return reject(res);
        }

        // wx[wxMethodName](opts);
        if (options && options.miniProgram) {
            let opts = Object.assign({}, options, {success: successFn, fail: failFn});
            wx.miniProgram[wxMethodName](opts);
        } else {
            let opts = Object.assign({}, options, {success: successFn, fail: failFn});            
            wx[wxMethodName](opts);
        }
    })
}
```

用法就比较简单啦，查微信小程序的API，用哪个写哪个。要按照文档增加参数，直接在options对象里面加就ok

注：
1. 尽量避免使用success回调，更好的做法是在下一环的promise中调用
2. 对调函数中最好使用箭头函数，以免引起this指向丢失问题

```
- /pages/index/index.js -

const wxMethod = require('../../utils/wx-method')
const app = getApp()

Page({
    ...
    onLoad: function() {
        wxMethod.showLoading()
        .then(res => {
            return wxMethod.request({
                url: xxxxx,
                method: 'POST',
                data: {name: 'john'}
            })
        })
        .then(res => {
            let data = res.data;
            this.setData({
                title: data.title
            })
        })
        .catch(e => {
            console.log(e);
        })
    }
})
```

看起来舒服多了。

## 一些其他

因为用到的每个api都要类似的写一遍，可能有点繁琐，但是api封装为这个样子，其实还有考虑到：这样写可以比较简单扩展为一个在h5中也能用的类。

比如在web-view嵌的h5中，要根据环境判断使用不同的逻辑（wx小程序、wx公众号、普通浏览器）：

```
...

// 获取小程序环境
export const wxGetSystemInfoSync = async (options = {}) => {
    // 根据ua判断是否在wx浏览器中
    if (equipment.isWx) {
        // 此处options.miniProgram设计为需要手动指定，让其他人阅读业务逻辑时，更清晰了解此处是在什么环境中执行
        // 但是如果用在小程序中，就不用指定了。小程序中的api直接就是wx.login这样的
        if (options.miniProgram) {
            return new Promise((resolve, reject) => {
                wx.miniProgram.getEnv(res => {resolve(res)})
            })
        } else {
            return Promise.resolve(extendWxFn({
                wxMethodName: 'getSystemInfoSync', 
                options: options || {}
            }));
        }
    } else {
        // 假如是web环境
        return Promise.resolve({
            miniprogram: false
        })
    }
}

/**
 * 对微信方法调用的统一封装（除部分参数结构不一样的方法，需要在外部做处理
 * @param {string} wxMethodName - 调用的wx包的方法
 * @param {Object} options - 对应不同方法中，包含的参数
 * @param {boolean} options.miniProgram - true:小程序的方法 false: wx包公众号网页的方法
 * @param {function} options.success - 成功回调
 * @param {function} options.fail - 失败回调
 * ps：没有提供complete回调，因为对应逻辑可以在promise链末端实现
 */
function extendWxFn({wxMethodName, options = {}}) {
    debug && console.log('extendWxFn', wxMethodName, options)    
    return new Promise((resolve, reject) => {
        let successFn = (res) => {
            debug && console.log('extend promise', options)
            let success = options.success;
            if (success && typeof success === 'function') {
                success(res);
            };
            return resolve(res);
        }
        let failFn = (res) => {
            let fail = options.fail;
            if (fail && typeof fail === 'function') {
                fail(res);
            }
            return reject(res);
        }

        if (options && options.miniProgram) {
            let opts = Object.assign({}, options, {success: successFn, fail: failFn});
            wx.miniProgram[wxMethodName](opts);
        } else {
            let opts = Object.assign({}, options, {success: successFn, fail: failFn});            
            wx[wxMethodName](opts);
        }
    })
}
```

***

后续有需求，会补充更多的。