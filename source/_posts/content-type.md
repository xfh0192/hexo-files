---
title: 关于http请求中的Content-Type
date: 2018-04-18 22:24:56
tags: http
---

# 关于http请求中的Content-Type

写在前面：之前和后端对接的时候，被各种跨域还有header的问题坑来坑去，所以记得啥就写些啥了。先从简单的写吧。

先mark一些问题：

1. 跨域，cors。加header（ Access-Control-Allow-Origin、Access-Control-Expose-Headers

2. header中的content-type

3. 和php后台对接的时候，data请先用qs模块进行stringify

4. get的参数用一下encodeURIComponent，post的数据需要转码

5. 顺便记一下以前碰到的问题：假如url中的参数经过base64等方式加密，有可能会出现空格（' '），而假如使用加密后的url进行跳转（如下），浏览器在url解析编码时，有时候会将空格编码为加号（'+'），有时则只是%20（ascii码表的十六进制，代表空格）。

http://www.baidu.com?key=value name

因此建议使用前，对参数调用encodeURIComponent，将空格都转成%20

***

HTTP大家都不陌生，但是HTTP的许多细节，还是很复杂的。

## 写一个AJAX

AJAX实现的核心就是利用XMLHttpRequest对象发送请求。

首先，怎么用原生js写一个GET请求呢？

```
let xhr = new XMLHttpRequest();
xhr.open("GET", "/api");
xhr.send();
```

xhr.open 第一个参数是请求方法，第二个参数是请求url，然后send出去就可以了

并且需要注意一下的是，参数需要转义：

```
let ajax =（url = '', data = {}) => {
	let args = [];
	for (let key in data) {
		// 参数需要转义
		args.push(`${encodeURIComponent(key)}=${encodeURIComponent(data[key]}`;
	}
	
	let search = args.join('&');
	// 判断当前url是否有参数
	url = url.indexOf('?') >= 0 ? `&${search}` : `?${search}`;
	
	let xhr = new XMLHttpRequest();
	xhr.open("GET", url);
	xhr.send();
}
```

get请求的话，参数是在url后面的。

然后是POST请求。

```
let xhr = new XMLHttpRequest();
xhr.open('POST', '/api');
xhr.send('id=5&name=yin');	// 请求的数据是放在send里面的
```
post请求的话，数据就会在请求体中了。

## 看看请求的结果

以上面的post请求为例。

我们会发现请求的mine类型为text/plain

```
Request Headers:
...
Content-Type: text/plain;charset=UTF-8
...

Request Payload:
page=5&pageSize=20
```

并且请求的参数，跟我们平常看到的不太一样。

```
// 平常看到的payload

src: web
uid: 58b322a22f301e006c0f3060
token: eyJhY2Nlc3NfdG9rZW4iOiJYVnJEMHFMRVZtWWdyOXJVIiwicmVmcmVzaF90b2tlbiI6InVSYmVwcnRURkJPY1dPOFciLCJ0b2tlbl90eXBlIjoibWFjIiwiZXhwaXJlX2luIjoyNTkyMDAwfQ==
device_id: 1523980025803
field: jobTitle
value: 这个是value，截图不好使，就copy出来了
```

可以看到，平常的请求检查时，会按字段显示

## 设置content-type

这是为什么呢？是因为我们没有设置它的Content-Type，如下：

```
let xhr = new XMLHttpRequest();
xhr.open("POST", "/api");
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhr.send('id=5&name=yin');
```

### x-www-form-urlencoded

如果设施了content-type为x-www-form-urlencoded的话，检查时，chrome也会按字段分行展示：

```
id: 5
name: yin
```

它是一种最常用的请求编码格式，支持GET/POST方法，特点是：所有的数据变成键值对的形式，即key1=value1&key2=value2，并且特殊字符需要转义。

另外，假如send的是一个对象，结果会显示[object Object]。说明会自动调用toString，因此在send数据时，需要转换为字符串


### application/json

假如开发的时候，和后端约定好，数据是json格式。这个时候要求格式要变成json，就需要指定Content-Type为application/json，然后send的数据要stringify一下

```
let xhr = new XMLHttpRequest();
xhr.open("POST", "/add");
xhr.setRequestHeader("Content-type", "application/json");
let data = {id:5, name: "yin"};
xhr.send(JSON.stringify(data));
```

结果：

```
{id: 5, name: "yin"}
```

我们可以比较json和urlencoded这两种形式的优缺点.

* json的缺点是parse解析的工作量要明显高于split("&")的工作量

* 但是json的优点又在于表达复杂结构的时候比较简洁，如二维数组(m * n)在urlencoded需要拆成m * n个字段，而json就不用了。

* 所以相对来说，如果请求数据结构比较简单应该是使用常用的urlencoded比较有利，而比较复杂时使用json比较有利。通常来说比较常用的还是urlencoded.

### multipart/form-data

还有这一种编码multipart/form-data。

```
let formData = new FormData();
formData.append("id", 5); // 数字5会被立即转换成字符串 "5"
formData.append("name", "#yin");
// formData.append("file", input.files[0]);

let xhr = new XMLHttpRequest();
xhr.open("POST", "/add");
xhr.send(formData);
```

它通常用于上传文件，上传文件必须要使用这种格式。上面代码发送的内容如下所示

```
...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryJ4wHSrmAWG3dNM98
...
payload:
----WebKitFormBoundaryJ4wHSrmAWG3dNM98	// 字段和字段之间用一个随机字符串隔开（上传文件需要用这种形式
Content-Disposition: form-data; name="id"

5
----WebKitFormBoundaryJ4wHSrmAWG3dNM98
Content-Disposition: form-data; name="name"

#yin 			// 内容就不需要转义了
----WebKitFormBoundaryJ4wHSrmAWG3dNM98
```
每个字段之间用一个随机字符串隔开，保证发送的内容不会出现这个字符串，这样发送的内容就不需要进行转义了，因为如果文件很大的话，转义需要花费相当的时间，体积也可能会成倍地增长。

另外,假如碰到需要下载/打开文件的时候，就需要服务端设置Content-Disposition这个header了，简单来说。。就是并不需要我们折腾


***

还有些其他的知识，有空补充嗯。

1. http状态码
2. 使用地址栏和使用ajax的get请求，会有啥不一样？