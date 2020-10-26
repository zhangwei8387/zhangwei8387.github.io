---
title: batchprint
date: 2020-10-26 15:46:59
categories: 
- 开发技术
tags:
- 软件开发
---
# 项目背景
为了解决目前EES中批量打印对html页面的支持问题，以及提高速度和降低依赖，设计了这个项目。
# 相关技术
## [Node.js](https://nodejs.org/zh-cn/)
[Node.js](https://nodejs.org/zh-cn/)是一个Javascript运行环境(runtime)，发布于2009年5月，由Ryan Dahl开发，实质是对Chrome V8引擎进行了封装。
Node.js对一些特殊用例进行优化，提供替代的API，使得V8在非浏览器环境下运行得更好。V8引擎执行Javascript的速度非常快，性能非常好。
Node.js是一个基于Chrome JavaScript运行时建立的平台， 用于方便地搭建响应速度快、易于扩展的网络应用。
Node.js 使用事件驱动，非阻塞I/O 模型而得以轻量和高效，非常适合在分布式设备上运行数据密集型的实时应用。
## [Restify](http://restify.com/)
Restify 是一个 Node.JS 模块，可以让你创建正确的 REST web services。它借鉴了很多 express 的设计，因为它是 node.js web 应用事实上的标准 API。
## [headless-chrome](https://developers.google.com/web/updates/2017/04/headless-chrome)
Headless Chrome 是 Chrome 浏览器的无界面形态，可以在不打开浏览器的前提下，使用所有 Chrome 支持的特性运行你的程序。相比于现代浏览器，Headless Chrome 更加方便测试 web 应用，获得网站的截图，做爬虫抓取信息等。相比于出道较早的 PhantomJS，SlimerJS 等，Headless Chrome 则更加贴近浏览器环境。
## [puppeteer](https://github.com/GoogleChrome/puppeteer)
Puppeteer 是一个控制 headless Chrome 的 Node.js API 。它是一个 Node.js 库，通过 DevTools 协议提供了一个高级的 API 来控制 headless Chrome。它还可以配置为使用完整的（非 headless）Chrome。
在浏览器中手动完成的大多数事情都可以通过使用 Puppeteer 完成，下面是一些入门的例子：

 - 生成屏幕截图和 PDF 页面
 - 检索 SPA 并生成预渲染内容（即“SSR”）
 - 从网站上爬取内容
 - 自动提交表单，UI测试，键盘输入等
 - 创建一个最新的自动测试环境。使用最新的 JavaScript 和浏览器功能，在最新版本的 Chrome 中直接运行测试
 - 捕获网站的时间线跟踪，以帮助诊断性能问题
 
# 项目主要功能
 - 提供一个Web API,接收用户的批量打印请求。请求中至少携带一个页面的URL地址。
 - 处理批量打印请求，使用Chrome 的 headless方式访问请求中指定的页面。
 - 使用Chrome的API将网页另存为PDF文档。
 - 提供PDF文档的URL供下载或者浏览。
 - 需要考虑批量打印的PDF整合问题。
 
# 项目准备
以Windows开发环境为例。

## 1. 安装Node.js
到[Node.js](https://nodejs.org/zh-cn/)官网下载最新版的node.js,版本要求>8.0 。
## 2. 创建项目
在你认为合适的地方创建一个STAr-BatchPrint(可自定义)文件夹，使用CMD打开文件夹，执行如下命令:

    npm init

npm 会引导你创建package.json的文件。关于NPM其他相关命令请参考[NPM命令说明](https://docs.npmjs.com/cli/init)

## 3. 安装扩展包
在命令行 执行

    npm install restify puppeteer --save
    
安装 Restify和Pupperteer两个包，记得加上--save以便更新至packag.json中。
pupeteer会下载一个特别的Chrome浏览器到你的项目中，大概120MB，请耐心等待...

至此，项目开发环境就全部搭建完成。

# 功能实现
## 创建index.js并引入扩展包
首先我们得创建一个index.js，此文件是项目的启动文件。

    ```javascript
     // 引入puppeteer和restify
    const puppeteer = require('puppeteer');
    const restify = require('restify');
    ```
   

## 创建Restify API Server
使用Restify提供的方法创建一个Server，并指定IP和端口。

    ```javascript
    //指定Server的IP地址和端口
    const ip_addr = '127.0.0.1'; 
    const port = '8080';

    //启动Server
    var server = restify.createServer({
    name: 'Batch-Print-Server'
    });

    //启动服务端监听
    server.listen(port, ip_addr, function() {
    console.log('%s listening at %s ', server.name, server.url);
    });
    ```

## 配置API的Route
我们使用/print 来处理关于打印的请求。
其中postNewPrintJob为该请求的Post回调函数。下面我们来实现它。

    ```javascript
    //启用restify的插件
    server.use(restify.plugins.queryParser());
    server.use(restify.plugins.bodyParser({ requestBodyOnGet: true }));

    //指定根路由返回内容
    server.get('/', function(req, res, next) {
    res.send('Hello BatchPrint!');
    return next();
    });
    // testing the service  
    server.get('/test', function (req, res, next) {  
        res.send("testing...");  
        next();  
    });
    //指定Route
    PATH = '/print';
    //指定相应Route的方法
    server.post({ path: PATH, version: '0.0.1' }, postNewPrintJob);
    ```

## 实现postNewPrintJob
该回调函数主要处理请求的数据，以及完成生成PDF的操作。

    ```javascript
    function postNewPrintJob(req, res, next) {
    var job = {};
    //    job.title = req.params.title;
    //    job.description = req.params.description;
    job.location = req.params.location;
    job.filename = req.params.filename;
    job.postedOn = new Date().toLocaleString();

    console.log(job);

    //设置跨域响应
    res.setHeader('Access-Control-Allow-Origin', '*');

    (async () => {
        //启动Chrome
        const browser = await puppeteer.launch();
        //新建Page
        const page = await browser.newPage();
        //打开页面 load方式指页面所有事务执行完毕
        await page.goto(job.location, { waitUntil: 'load' });
        //指定响应Media为Print
        await page.emulateMedia('print');
        //生成PDF文件，format设为A4,margin为默认页边距。
        const pdffile = await page.pdf({
        path: pdffiledirectory+'/'+job.filename,
        format: 'A4',
        margin: {
            top: '10mm',
            bottom: '10mm',
            left: '10mm',
            right: '10mm'
        }
        });
        //关闭浏览器。
        browser.close();

        if (pdffile) {
        //返回结果。
        res.send(200, pdffile);
        return next();
        }
        return next(err);
    })();
    ```

上面的方法可以看到其中使用了Pupperteer对Chrome进行操作。具体API可以查询[Pupperteer](https://github.com/GoogleChrome/puppeteer)的GitHub.

具体代码请查看[index.js](https://github.com/zhangwei8387/STAr-BatchPrint/blob/master/index.js)

## 启动Server
在CMD命令行中执行node index.js

    PS D:\GitCode\STAr-BatchPrint> node index.js
  
会返回如下信息:

    Batch-Print-Server listening at http://127.0.0.1:8080

表示服务已经开启。

## 测试服务
在浏览器中打开http://127.0.0.1:8080/。我们可以看到如下信息.

    "Hello BatchPrint!"

访问下级路由，http://127.0.0.1:8080/test。我们可以看到如下信息.

    "testing..."

但是我们如果访问 http://127.0.0.1:8080/print, 会得到一个错误信息。

    {"code":"MethodNotAllowed","message":"GET is not allowed"}

说明这个路由并没有指定Get方法。因为我们使用Post来向服务器推送数据。

推荐使用[PostMan](https://www.getpostman.com/)API测试工具来向Server Post我们模拟的数据。

点击Send后，在我们Server的目录中会生成一个PDF文档。打开后就可以看到指定网页的内容。