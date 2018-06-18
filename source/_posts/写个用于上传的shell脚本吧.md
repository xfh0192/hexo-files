---
title: 写个用于上传的shell脚本吧
date: 2018-06-16 20:20:13
tags: 
- node
- shell
---

# 写个用于上传的shell脚本吧

最近困扰于通过ftp上传代码到测服时，总是需要手动操作。于是摸索了一下，可以借助node写一点脚本，帮助我快速实现上传的操作。

具体来说，用到了node的包：[ssh2](https://www.npmjs.com/package/ssh2)

## 需求效果

大致上是将dist文件夹上传到远程服务器上面。文件夹目录如下：

```
dist
├── mobile
│   ├── article-list.html
│   ├── index.html
├── pcmall
│   ├── forTest.html
│   └── index.html
└── static
    ├── css
    │   ├── mobile-article-list.css
    │   ├── mobile-index.css
    ├── img
    │   ├── appreciate-people.859a832.png
    │   ├── banner1.0f75289.png
    │   ├── banner2.4a37379.png
    └── js
        ├── babel-polyfill.c168bb080d720539793a.js
        ├── manifest.f8d9173cdd53ce0c820b.js
        └── vendor.ad87910d1acc0e175314.js
```

我们通过使用ssh的sftp实现。

利用 [fastPut](https://github.com/mscdex/ssh2-streams/blob/master/SFTPStream.md) 方法

## 具体步骤

1. 首先可以写一些简单的方法

```
# ssh-base-class.js

// https://github.com/mscdex/ssh2#user-content-client-methods

let Client = require('ssh2').Client;

class sshBaseClass {
	
    constructor() {
    	// 实例化一个客户端对象
        this.client = new Client();
    }

	// 创建一个ssh连接
    connect(config) {
        config = config || {};

        return new Promise((resolve, reject) => {
            this.client.on('ready', () => {
                console.log('connect ready');
                resolve();
            })
            .on('error', (err) => {
                console.log('connect error');
            })
            .on('end', (err) => {
                console.log('connect end');
            })
            .on('close', (err) => {
                console.log('connect close');
            })
            .connect(config);
        })
    }

	// 断开连接
    end() {
        return new Promise((resolve, reject) => {
            resolve(this.client.end());
        })
    }

}

module.exports = sshBaseClass;

```

2、然后封装一些简单的方法

utils.js

```
# utils.js

let glob = require('glob');

module.exports = {
    getFiles: (pattern) => {
        if (!pattern) return [];
        
        let files = glob.sync(pattern, {
            nodir: true,
        });
        return files;
    }
}
```

sftp.js

```
let fs = require('fs');

let utils = require('./utils');
let sshBaseClass = require('./ssh-base-class');

module.exports = class SftpClient extends sshBaseClass{

    constructor() {
        super();
        
        // client.sftp() 会返回一个sftp会话，将该会话保存在此处
        this.sftp = null;
    }
    
    // 连接sftp (override了)
    connect(config) {
        config = config || {};

        return new Promise((resolve, reject) => {
            this.client.on('ready', () => {
                console.log('sftp connection ready');
                
                this.client.sftp((err, sftp) => {
                    if (err) {
                        reject(err);
                    }
                    this.sftp = sftp;
                    resolve(sftp);
                });
            })
            .on('error', (err) => {
                consolel.log('sftp connection error event');
                reject(err);
            })
            .connect(config);
        })
    }
    
    // 获取配置项
    getOptions(customOptions) {
        let options = Object.assign({}, {encoding: 'utf8'}, customOptions);
        return options;
    }
    
    // 上传指定路径的指定文件。单个
    put(localPath, remotePath, customOptions) {
        let options = this.getOptions(customOptions || {});

        return new Promise((resolve, reject) => {
            let sftp = this.sftp;

            if (sftp) {

                sftp.fastPut(localPath, remotePath, options, (err) => {
                    if (err) {
                        reject(err);
                        return false;
                    }
                    resolve();
                    return false;
                });

            }
        })
    }
    
    // 在远程服务器，创建文件夹
    // @params {string} path: 远程中，需要创建的文件夹路径
    // @params {boolean} recursive: 是否需要遍历目录深层
    mkdir(path, recursive) {
        
        return new Promise((resolve, reject) => {
            let sftp = this.sftp;
            
            if (sftp) {

                if (!recursive) {
                    sftp.mkdir(path, (err) => {
                        if (err) {
                            // console.log(err);
                            reject(err);
                            return false;
                        }
                        resolve();
                    })
                    return false;
                }

                let tokens = path.split(/\//g);
                let p = '';

                let mkdir = () => {
                    let token = tokens.shift();

                    if (!token && !tokens.length) {
                        resolve();
                        return false;
                    }
                    token += '/';
                    p = p + token;
                    sftp.mkdir(p, (err) => {
                        if (err && err.code !== 4) {
                            reject(err);
                        }
                        mkdir();
                    })
                }
                return mkdir();
            } else {
                reject(Error('sftp mkdir error'));
            }
        })
    }
    
    // 上传文件整个文件夹。遍历并创建中间目录
    // item {string} item: 目标文件夹中的文件路径，如：mobile/index.html
    // localDir {string} localDir: 本地目标文件夹，如： path.resolve('dist') + '/'
    // remoteDir {string} remoteDir: 
    putDir(item, localDir, remoteDir) {
        return new Promise((resolve, reject) => {
            let sftp = this.sftp;
            let tokens = item.split('/');
            tokens.pop();
            let p = tokens.join('/');

            if (sftp) {
                return this.mkdir(remoteDir + p, true)
                        .then(() => {
                            resolve(this.put(localDir + item, remoteDir + item));
                        })
            }
        })
    }
}


```

3、然后考虑到需要遍历文件夹内的上传文件，所以要保证执行顺序：

promise-executor.js

```
// https://www.jianshu.com/p/f4eb7432feb2

// 利用promise队列，使一连串promise的执行lazy化，有效控制顺序
class PromiseExecutor {
    constructor(queue) {
        // lazy promise 队列
        this._queue = [];

        // 一个变量锁。当前有promise执行，就等待
        this._isBusy = false;

        // 回调函数
        this._callback = null;
        
    }

    each(callback) {
        this._callback = callback;
    }

    add(lazyPromise) {
        if (!lazyPromise) {
            // lazyPromise = Promise.resolve();
            return false;
        }
        this._queue.push(lazyPromise);

        if (this._isBusy) {
            return;
        }
        this._isBusy = true;

        return this.execute();
    }

    async execute() {

        try {
            // 按队列任务顺序依次执行
            while(this._queue.length > 0) {
                const head = this._queue.shift();
                const value = await head;
                this._callback && this._callback(value);
            }

            // 执行完之后，解锁
            this._isBusy = false;
            return Promise.resolve();
        } catch(e) {
            console.log(e);
            return Promise.reject(e);
        }
    }
}

module.exports = PromiseExecutor;

```

4、 好的，最后就是实际调用了

将build之后的 **dist/static** 文件夹上传。


test.js

```
let path = require('path');
let fs = require('fs');

let FstpClient = require('./utils/sftp');
let utils = require('./utils/utils');
let PromiseExecutor = require('./utils/promise-executor');

let client = new FstpClient();
let promiseExec = new PromiseExecutor();

client.connect({
    host: 'xxxxxxx',
    port: 22,
    username: 'xxxxxxx',
    password: 'xxxxxxxxxx'
})
.then(() => {
    let localDir = path.resolve('dist') + '/';
    let files = utils.getFiles(localDir + '**');
    let targetDir = '/media/test/';

    for (let i = 0, len = files.length; i < len; i++) {
        let promise = null;

        let item = files[i];
        let fileName = item.split(localDir)[1];
        promise = new Promise((resolve, reject) => {
            resolve(client.putDir(fileName, localDir, targetDir));
        })

        promiseExec.add(promise);
    }
    return promiseExec.execute();
})
.then((data) => {
    console.log('finish');
    client.end();
})
.catch(err => {
    console.log('err')
    client.end();
})
```