# node.js - 百度百科

Node.js发布于2009年5月，由Ryan Dahl开发，是一个基于Chrome V8引擎的[JavaScript](https://baike.baidu.com/item/JavaScript/321142)运行环境，使用了一个[事件驱动](https://baike.baidu.com/item/事件驱动/9597519)、非阻塞式I/O模型， [1] 让JavaScript 运行在[服务端](https://baike.baidu.com/item/服务端/6492316)的开发平台，它让JavaScript成为与[PHP](https://baike.baidu.com/item/PHP/9337)、[Python](https://baike.baidu.com/item/Python/407313)、[Perl](https://baike.baidu.com/item/Perl/851577)、[Ruby](https://baike.baidu.com/item/Ruby/11419)等服务端语言平起平坐的[脚本语言](https://baike.baidu.com/item/脚本语言/1379708)。 [2] 

Node.js对一些特殊用例进行优化，提供替代的[API](https://baike.baidu.com/item/API/10154)，使得V8在非浏览器环境下运行得更好，V8引擎执行Javascript的速度非常快，性能非常好，基于Chrome JavaScript运行时建立的平台， 用于方便地搭建响应速度快、易于扩展的[网络应用](https://baike.baidu.com/item/网络应用/2196523)。

## 发展历程

- 2009年2月，Ryan Dahl在[博客](https://baike.baidu.com/item/博客/124)上宣布准备基于V8创建一个轻量级的[Web服务器](https://baike.baidu.com/item/Web服务器)并提供一套库。

- 2009年5月，Ryan Dahl在GitHub上发布了最初版本的部分Node包，随后几个月里，有人开始使用Node开发应用。

- 2009年11月和2010年4月，两届JSConf大会都安排了Node.js的讲座。

- 2010年年底，Node获得[云计算](https://baike.baidu.com/item/云计算)服务商Joyent资助，创始人Ryan Dahl加入Joyent全职负责Node的发展。

- 2011年7月，Node在微软的支持下发布Windows版本。

## 主要功能

- V8引擎本身使用了一些最新的编译技术。这使得用Javascript这类[脚本语言](https://baike.baidu.com/item/脚本语言)编写出来的代码运行速度获得了极大提升，又节省了开发成本。对性能的苛求是Node的一个关键因素。 Javascript是一个[事件驱动](https://baike.baidu.com/item/事件驱动)语言，Node利用了这个优点，编写出可扩展性高的服务器。Node采用了一个称为“事件循环(event loop）”的架构，使得编写可扩展性高的服务器变得既容易又安全。提高服务器性能的技巧有多种多样。Node选择了一种既能提高性能，又能减低开发复杂度的架构。这是一个非常重要的特性。并发编程通常很复杂且布满地雷。Node绕过了这些，但仍提供很好的性能。

- Node采用一系列“非阻塞”库来支持事件循环的方式。本质上就是为文件系统、数据库之类的资源提供接口。向文件系统发送一个请求时，无需等待硬盘（[寻址](https://baike.baidu.com/item/寻址)并检索文件），硬盘准备好的时候非阻塞接口会通知Node。该模型以可扩展的方式简化了对慢资源的访问， 直观，易懂。尤其是对于熟悉[onmouseover](https://baike.baidu.com/item/onmouseover)、onclick等[DOM](https://baike.baidu.com/item/DOM/50288)事件的用户，更有一种似曾相识的感觉。

- 虽然让Javascript运行于服务器端不是Node的独特之处，但却是其一强大功能。不得不承认，浏览器环境限制了我们选择编程语言的自由。任何服务器与日益复杂的浏览器客户端应用程序间共享代码的愿望只能通过Javascript来实现。虽然还存在其他一些支持Javascript在服务器端 运行的平台，但因为上述特性，Node发展迅猛，成为事实上的平台。

- 在Node启动的很短时间内，社区就已经贡献了大量的扩展库（模块）。其中很多是连接数据库或是其他软件的驱动，但还有很多是凭他们的实力制作出来的非常有用的软件。

- 最后，不得不提到的是Node社区。虽然Node项目还非常年轻，但很少看到对一个项目如此狂热的社区。不管是新手，还是专家，大家都围绕着项目，使用并贡献自己的能力，致力于打造一个探索、支持、分享、听取建议的乐土。

## 运行环境

Node作为一个新兴的前端框架，后台语言，有很多吸引人的地方：RESTful API，单线程

Node可以在不新增额外线程的情况下，依然可以对任务进行并发处理 —— Node.js是单线程的。它通过事件循环（event loop）来实现并发操作，对此，我们应该要充分利用这一点 —— 尽可能的避免阻塞操作，取而代之，多使用非阻塞操作。

- 非阻塞IO

- V8虚拟机

- 事件驱动

## 功能模块

Node使用Module模块去划分不同的功能，以简化应用的开发。Modules模块有点像C++语言中的类库。每一个Node的类库都包含了十分丰富的各类函数，比如http模块就包含了和http功能相关的很多函数，可以帮助开发者很容易地对比如http,tcp/udp等进行操作，还可以很容易的创建http和tcp/udp的服务器。

要在程序中使用模块是十分方便的，只需要如下：

在这里，引入了http类库，并且对http类库的引用存放在http变量中了。这个时候，Node会在我们应用中搜索是否存在node_modules的目录，并且搜索这个目录中是否存在http的模块。如果Node.js找不到这个目录，则会到全局模块缓存中去寻找，用户可以通过相对或者绝对路径，指定模块的位置，比如：

```js
var myModule = require('./myModule.js');
```

模块中包含了很多功能代码片断，在模块中的代码大部分都是私有的，意思是在模块中定义的函数方法和变量，都只能在同一个模块中被调用。当然，可以将某些方法和变量暴露到模块外，这个时候可以使用exports对象去实现。

## 下载安装

***Linux下安装Node\***

下面介绍下Node的安装，首先在nodejs的网站上根据操作系统下载相关的安装包，对于Ubuntu(linux)下的安装，可以如下进行：

```shell
sudo apt-get update
sudo apt-get install node
```

或者：

```shell
sudo apt update
sudo apt install node
```

***Windows下安装Node\***

官网现已提供安装包（最新的长期支持版本: **10.16.0**）、编译器和相应的API 文档（English）。 [1] 

## 应用方向

- 具备书写JavaScript的IDE，普通的记事本也可以进行开发。在几年的时间里，Node.JS逐渐发展成一个成熟的开发平台，吸引了许多开发者。有许多大型高流量网站都采用Node.JS进行开发，此外，开发人员还可以使用它来开发一些快速移动[Web](https://baike.baidu.com/item/Web/150564)框架。

- 除了Web应用外，NodeJS也被应用在许多方面，本文盘点了NodeJS在其它方面所开发的十大令人神奇的项目，这些项目涉及到应用程序监控、媒体流、远程控制、桌面和移动应用等等。

## 示例程序

任何一套标准都由一个著名的程序开始：Hello World ！在Node中，Http是首要的。Node为创建http服务器做了优化，所以你在网上看到的大部分示例和库都是集中在web上(http框架、模板库等）。以下做了一个nodejs的Hello World 演示：

```js
var http = require('http');
server = http.createServer(function (req, res) {
res.writeHeader(200, {"Content-Type": "text/plain"});
res.end("Hello World\n");
});
server.listen(8000);
console.log("httpd start @8000");
```

事务处理示例，本示例意图向读者传递 Node.js 关于 HTTP 处理过程的详实概念。在不考虑编程语言和环境的情况下，我们假设您已经知晓通常情况下 HTTP 请求是如何工作的，并且对 Node.js 的 EventEmitters 和 Streams 也已知晓。如果您对他们并不熟悉，通过 API 文档可以快速查阅。

创建服务Node 的网络应用都需要先创建一个网络服务对象，这里我们通过 createServer 来实现。

```js
var http = require('http');
var server = http.createServer(function(request, response) {   
// handle your request
});
```

传入 createServer 的 function 在每次 HTTP 请求时都将被调用执行，因此这个 function 也被称为请求的处理者。事实上通过 createServer 返回的 Server 对象是一个 EventEmitter，我们需要做的仅仅是在这里保存这个 server 对象，并在之后对其添加监听器。

```js
var http = require('http');
var server = http.createServer(); 
server.on('request', function(request, response) {
// handle your request
});
```

当 HTTP 请求这个服务时，node 调用请求处理者 function 并传入一些用于处理事务相关的对象：request 和 response。我们可以非常方便的获得这两个对象。

```js
var http = require('http');
var server = http.createServer(); 
server.on('request', function(request, response) {
// handle your request
}).listen(8080); 
```

为了对实际的请求提供服务，在 server 对象上需要调用 listen 方法。绝大多数情况你需要传入 listen 你想要服务监听的端口号，这里也存在很多其他的可选方案，参见 API reference。