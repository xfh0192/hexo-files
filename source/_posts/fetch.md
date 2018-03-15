---
title: fetch
date: 2018-01-15 22:24:56
tags: html
---

# fetch API

以往我们想要实现ajax请求，不论用的是哪种框架，基本上的实现是 **XMLHTTPRequest（XHR）** 。但是在以后，或许可以开始转向使用 **fetch** 了。

## 对比XHR

XMLHttpRequest 是一个设计粗糙的 API，不符合关注分离（Separation of Concerns）的原则，配置和调用方式非常混乱，而且基于事件的异步模型写起来也没有现代的 Promise，generator/yield，async/await 友好。

Fetch 的出现就是为了解决 XHR 的问题，拿例子说明：

使用 XHR 发送一个 json 请求一般是这样：

```
var xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.responseType = 'json';

xhr.onload = function() {
  console.log(xhr.response);
};

xhr.onerror = function() {
  console.log("Oops, error");
};

xhr.send();
```

如果用上promise，会简洁一点：

```
fetch(url)
.then(function(res) {
  return res.json();
})
.then(function(data) {
  console.log(data);
})
.catch(function(e) {
  console.log("Oops, error");
});
```

当然如果能使用上 async/await 就会更简洁了。

```
......

try {
  let response = await fetch(url);
  let data = await response.json();
  console.log(data);
} catch(e) {
  console.log("Oops, error", e);
}
```

## 基本使用

fetch(url [,options])

**url**: [string] 请求地址

**options**：[Object] 请求的配置，可包含请求头和Request对象的配置

### 请求头(Request Headers)

```
// 创建一个空的 Headers 对象,注意是Headers，不是Header
var headers = new Headers();

// 添加(append)请求头信息
headers.append('Content-Type', 'text/plain');
headers.append('X-My-Custom-Header', 'CustomValue');

// 判断(has), 获取(get), 以及修改(set)请求头的值
headers.has('Content-Type'); // true
headers.get('Content-Type'); // "text/plain"
headers.set('Content-Type', 'application/json');

// 删除某条请求头信息(a header)
headers.delete('X-My-Custom-Header');

// 创建对象时设置初始化信息
var headers = new Headers({
    'Content-Type': 'text/plain',
    'X-My-Custom-Header': 'CustomValue'
});
```

### Request对象

Request 对象表示一次 fetch 调用的请求信息

示例:

```
var request = new Request('/users.json', {
    method: 'POST', 
    mode: 'cors', 
    redirect: 'follow',
    headers: new Headers({
        'Content-Type': 'text/plain'
    })
});

// 使用!
fetch(request).then(function() { /* handle response */ });
```

配置对象说明：

key|value
---|---
method|GET, POST, PUT, DELETE, HEAD等
url|请求的url
headers|对应的 Headers 对象
referrer|请求的 referrer 信息
mode|cors(允许跨域请求)，same-origin(同源请求)，no-cors(禁止同源，只能使用get、post、head)
credentials|设置 cookies 是否随请求一起发送。可以设置: omit（从不发送cookie）, same-origin(同源时发送cookie), include(总是发送cookie)
redirect|如何处理重定向模式. follow, error, manual
integrity|请求内容的 subresource integrity 值
cache|设置 cache 模式 (default, reload, no-cache)

### Response 对象

(...属性部分未完，待整理)

提供的方法：

function|description
---|---
Response.clone()|创建一个Response对象的克隆
Response.error()|返回一个绑定了网络错误的新的Response对象
Response.redirect()|用另一个URL创建一个新的 response.
Response实现(implements)了Body,所以以下方法同样可用:|------
Body.arrayBuffer()|读取 Response对象并且将它设置为已读（因为Responses对象被设置为了 流【stream】 的方式，所以它们只能被读取一次）,并返回一个被解析为ArrayBuffer格式的promise对象
Body.blob()|....Blob格式....
Body.formData()|....FormData格式....
Body.json()|....JSON格式....(大概是用的最多的方法）
Body.text()|....USVString格式....


(implements,个人理解应该是类似class的类继承，所以子类可以调用父类的定义在类上面的方法【不是定义在父类的原型,参考es6的class部分

(stream 流。在node上面有提出，目前个人是理解为原始的二进制数据，即没有MINE类型，因此需要在Reponse对象中调用json()等函数进行格式转换

## 浏览器的原生支持率

不过注意一下，低版本的浏览器是需要polyfill的（当然也不建议低版本的浏览器用啦。。

![https://mdn.mozillademos.org/files/15083/fetch-api.png](https://mdn.mozillademos.org/files/15083/fetch-api.png)

原生支持率并不高，幸运的是，引入下面这些 polyfill 后可以完美支持 IE8+ ：

1. 由于 IE8 是 ES3，需要引入 ES5 的 polyfill: [es5-shim, es5-sham](https://github.com/es-shims/es5-shim)
2. 引入 Promise 的 polyfill: [es6-promise](https://github.com/jakearchibald/es6-promise)
3. 引入 fetch 探测库：[fetch-detector](https://github.com/camsong/fetch-detector)
4. 引入 fetch 的 polyfill: [fetch-ie8](https://github.com/camsong/fetch-ie8)


## 自己瞎写的一段

如题，参考了别人的demo，自己瞎写的一段。需要polyfill的。

```
export default async (url = '', method = 'get', data = {}) => {
    console.log(method + ":" + url)
    // 拼装基本路径
    url = baseUrl + url

    // get方法,query拼接
    if (method == 'get') {
        let dataStr = ''
        Object.keys(data).forEach(key => {
            dataStr += key + '=' + encodeURIComponent(data[key]) + '&';
        })

        if (dataStr) {
            dataStr = dataStr.substr(0, dataStr.lastIndexOf('&'))
            url = url + '?' + dataStr
        }
    }

    // 检查是否有fetch方法
    // https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch
    if (window.fetch) {
        console.log('fetch is working')
        let requestConfig = {
            // 带上cookie 
            credentials: 'include',

            method: method,
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            },
            mode: 'cors'
            // cache: 'force-cache'
        }

        if (method == 'post') {
            requestConfig.body = JSON.stringify(data)
        }

        // fetch 接口（有polyfill）
        // https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API
        try {
            const response = await fetch(url, requestConfig)
            console.log(response)
            let responseJson = await response.json();

            if (!responseJson.errors) {
                return Promise.resolve(responseJson);
            } else {
                return Promise.reject(responseJson.errors[0] || responseJson.errors)
            }

        } catch (e) {
            throw new Error(e);
        }
    } else {
        return new Promise( (resolve, reject) => {
            let requestObj;
            // 新版浏览器
            if (window.XMLHttpRequest) {
                requestObj = new XMLHttpRequest()
            }
            // ie 
            else {
                requestObj = new ActiveXObject();
            }

            /* 
             *   method：请求的类型；GET 或 POST
             *   url：文件在服务器上的位置
             *   async：true（异步）或 false（同步）
            */
            requestObj.open(method, url, true)
            requestObj.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded')

            requestObj.onreadystatechange = () => {
                if (requestObj.readystate == 4) {
                    if (requestObj.status == 200) {
                        let obj = requestObj.response
                        if (typeof obj !== 'object') {
                            obj = JSON.parse(obj)
                        }
                        resolve(obj)
                    } else {
                        reject(requestObj)
                    }
                }
            }
        })
    }

    // return axios[method](uri, data)
}
```



参考：

[Fetch API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API#%E6%A6%82%E5%BF%B5%E5%92%8C%E7%94%A8%E6%B3%95)

[fetch](http://blog.csdn.net/renfufei/article/details/51494396)

[传统 Ajax 已死，Fetch 永生](https://github.com/camsong/blog/issues/2)

[fetch 简介](http://blog.csdn.net/renfufei/article/details/51494396)

对于跨域：[http://blog.csdn.net/gdp12315_gu/article/details/66479524](http://blog.csdn.net/gdp12315_gu/article/details/66479524)