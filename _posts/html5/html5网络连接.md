---
title: html5网络连接
date: 2016-05-31 12:05:05
tags: HTML5
---
## XMLHttpRequest Level 2

XMLHttpRequest Level 2 对于 老版本的XMLHttpRequest对象 功能的一个补全，这里我们先看下老版本的XMLHttpRequest对象的缺点

> * 只支持文本数据的传送，无法用来读取和上传二进制文件。
> * 传送和接收数据时，没有进度信息，只能提示有没有完成。
> * 受到"同域限制"（Same Origin Policy），只能向同一域名的服务器请求数据

而新版的 XMLHttpRequest 增加了更多的功能

> * 可以设置HTTP请求的时限。
> * 可以使用FormData对象管理表单数据。
> * 可以上传文件。
> * 可以请求不同域名下的数据（跨域请求）。
> * 可以获取服务器端的二进制数据。
> * 可以获得数据传输的进度信息。

<!--more-->

### 设置请求时限

```javascript
　　xhr.timeout = 3000;
    xhr.ontimeout = function(event){
　　　　alert('请求超时！');
　　}
```

目前，Opera、Firefox和IE 10支持该属性，IE 8和IE 9的这个属性属于XDomainRequest对象，而Chrome和Safari还不支持。

### FormData 上传文件

```html
<input type="file" accept="video/*" onchange="fileUpload(this.files[0])"/>
<script type="text/javascript">
    function fileUpload(file){
        var xhr = new XMLHttpRequest();
        var formData = new FormData();
        formData.append("glbh", params.key);
        formData.append("fileName", file.name);
        //文件大小
        formData.append("fileSize", file.size);
        formData.append("file", file);
        xhr.open('post', '/uploadFile');
        xhr.send(formData);
    }
</script>
```

### 更多的事件监听

```javascript
xhr.addEventListener('progress', function(oEvent){
    //监听上传进度
} , false);
xhr.addEventListener('load', function(evt){
    //上传完成
}, false);

xhr.addEventListener('error', function(e){
    //上传出错
}, false);
```

## WebSocket

HTTP协议是一种无状态协议，服务器端本身不具有识别客户端的能力，必须借助外部机制，比如session和cookie，才能与特定客户端保持对话。这多多少少带来一些不便，尤其在服务器端与客户端需要持续交换数据的场合（比如网络聊天），更是如此。为了解决这个问题，HTML5提出了浏览器的WebSocket API。

WebSocket的主要作用是，允许服务器端与客户端进行全双工（full-duplex）的通信。举例来说，HTTP协议有点像发电子邮件，发出后必须等待对方回信；WebSocket则是像打电话，服务器端和客户端可以同时向对方发送数据，它们之间存着一条持续打开的数据通道。

### 建立连接

```javascript
var connection = new WebSocket('ws://localhost:1740'); //建立连接
connection.onopen = function(event){
    //连接建立成功后调用此方法
}
connection.onclose = function(){
    //连接被关闭调用此方法
}
```

WebSocket协议完全可以取代Ajax方法，用来向服务器端发送文本和二进制数据，而且还没有“同域限制”。

WebSocket不使用HTTP协议，而是使用自己的协议。

### 发送和接收数据

```javascript
connection.send(message); //发送文字信息

// 使用Blob发送文件
var file = document.querySelector('input[type="file"]').files[0];
connection.send(file);
```

客户端收到服务器发送的数据，会触发message事件。可以通过定义message事件的回调函数，来处理服务端返回的数据。

```javascript
connection.onmessage = function (event) {
	console.log(event.data);
}
```

上面代码的回调函数的参数为事件对象event，该对象的data属性包含了服务器返回的数据。

### Socket.io

Socket.io是目前最流行的WebSocket实现，包括服务器和客户端两个部分。它不仅简化了接口，使得操作更容易，而且对于那些不支持WebSocket的浏览器，会自动降为Ajax连接，最大限度地保证了兼容性。它的目标是统一通信机制，使得所有浏览器和移动设备都可以进行实时通信。

```javascript
var socket = io.connect('http://localhost');
//监听事件
socket.on('news', function (data){
   console.log(data);
});
//发送信息
socket.emit('anotherNews');

```

上面代码的io.sockets.on方法指定connection事件（WebSocket连接建立）的回调函数。在回调函数中，用emit方法向客户端发送数据，触发客户端的news事件。然后，再用on方法指定服务器端anotherNews事件的回调函数。

不管是服务器还是客户端，socket.io提供两个核心方法：emit方法用于发送消息，on方法用于监听对方发送的消息。

## Server-Sent Events

传统的网页都是浏览器向服务器“查询”数据，但是很多场合，最有效的方式是服务器向浏览器“发送”数据。比如，每当收到新的电子邮件，服务器就向浏览器发送一个“通知”，这要比浏览器按时向服务器查询（polling）更有效率。

服务器发送事件（Server-Sent Events，简称SSE）就是为了解决这个问题，而提出的一种新API，部署在EventSource对象上。目前，除了IE，其他主流浏览器都支持。

简单说，所谓SSE，就是浏览器向服务器发送一个HTTP请求，然后服务器不断单向地向浏览器推送“信息”（message）。这种信息在格式上很简单，就是“信息”加上前缀“data: ”，然后以“\n\n”结尾。

SSE与WebSocket有相似功能，都是用来建立浏览器与服务器之间的通信渠道。两者的区别在于：

> WebSocket是全双工通道，可以双向通信，功能更强；SSE是单向通道，只能服务器向浏览器端发送。
> WebSocket是一个新的协议，需要服务器端支持；SSE则是部署在HTTP协议之上的，现有的服务器软件都支持。
> SSE是一个轻量级协议，相对简单；WebSocket是一种较重的协议，相对复杂。
> SSE默认支持断线重连，WebSocket则需要额外部署。
> SSE支持自定义发送的数据类型。

```javascript
var source = new EventSource('/dates');

source.onmessage = function(e){
  console.log(e.data);
};

// 或者
source.addEventListener('message', function(e){})

source.onopen = function(event) {
  // handle open event
};

// 或者

source.addEventListener("open", function(event) {
  // handle open event
}, false);
```

SSE 有两个事件， open用代开连接后事件， message，当接收到信息后的事件， 其中message的event对象有以下属性

> data：服务器端传回的数据（文本格式）。
> origin： 服务器端URL的域名部分，即协议、域名和端口。
> lastEventId：数据的编号，由服务器端发送。如果没有编号，这个属性为空。

## fetch

JavaScript 通过XMLHttpRequest(XHR)来执行异步请求，这个方式已经存在了很长一段时间。虽说它很有用，但它不是最佳API。它在设计上不符合职责分离原则，将输入、输出和用事件来跟踪的状态混杂在一个对象里。而且，基于事件的模型与最近JavaScript流行的Promise以及基于生成器的异步编程模型不太搭.

fetch 是w3c新推出用于 http协议数据交互的api， 主要是问了解决上述中 XMLHttpRequest 对象的缺点。 fetch基于 promise 的异步设计，是http请求操作更为优雅。

```javascript
fetch("/data.json").then(function(res) {
  // res instanceof Response == true.
  if (res.ok) {
    res.json().then(function(data) {
      console.log(data.entries);
    });
  } else {
    console.log("Looks like the response wasn't perfect, got status", res.status);
  }
}, function(e) {
  console.log("Fetch failed!", e);
});
```

在请求的时候也可以修改一些参数, 例如我们发送一个post请求

```javascript
fetch("http://www.example.org/submit.php", {
  method: "POST",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded"
  },
  body: "firstName=Nikhil&favColor=blue&password=easytoguess"
}).then(function(res) {
  if (res.ok) {
    alert("Perfect! Your settings are saved.");
  } else if (res.status == 401) {
    alert("Oops! You are not authorized.");
  }
}, function(e) {
  alert("Error submitting form!");
});

```

无论Request还是Response都可能带着body。由于body可以是各种类型，比较复杂，所以前面我们故意先略过它，在这里单独拿出来讲解。

body可以是以下任何一种类型的实例：

> ArrayBuffer
> ArrayBufferView (Uint8Array and friends)
> Blob / File
> 字符串
> URLSearchParams
> FormData ——目前不被Gecko和Blink支持，Firefox预计在版本39和Fetch的其他部分一起推出。

此外，Request和Response都为他们的body提供了以下方法，这些方法都返回一个Promise对象。

> arrayBuffer()
> blob()
> json()
> text()
> formData()

在使用非文本的数据方面，Fetch API和XHR相比提供了极大的便利。


## Navigation Timing

在页面开发中我们需要对网页中一些操作的性能做监测，以便于后续的顶点优化，例如： 资源文件下载时间，dom渲染时间等等,HTML5 给我们提供了 Navigation Timing API 来处理这些问题， 该API通过window.performance对象的属性来报告页面被导航或被加载的时间及相关信息。

> navigation:用户是怎样导航到当前页面的。
> timing:页面被导航或被加载完毕时所耗费的时间。

在Chrome浏览器中，从任何页面上执行以下操作：

1. 从浏览器右上角的图标菜单中选择“工具”>“JavaScript控制台”。
2. 在JavaScript控制台底部的>提示符旁边输入“performance”命令后按回车键。
3. 点击“performance”查看memory对象、navigation对象及timing对象的各属性值。
4. 点击timing对象旁的箭头查看它的属性值。

你会看到这样一些信息:

```JavaScript
connectEnd: 1351215923674
connectStart: 1351215923667
domComplete: 1351215925257
domContentLoadedEventEnd: 1351215925095
domContentLoadedEventStart: 1351215925091
domInteractive: 1351215925091
domLoading: 1351215924954
domainLookupEnd: 1351215923667
domainLookupStart: 1351215923667
fetchStart: 1351215923667
loadEventEnd: 0
loadEventStart: 1351215925257
navigationStart: 1351215923667
redirectEnd: 0
redirectStart: 0
requestStart: 1351215923675
responseEnd: 1351215925024
responseStart: 1351215924948
secureConnectionStart: 0
unloadEventEnd: 1351215924949
unloadEventStart: 1351215924949
```

Navigation Timing 将页面各个操作的时间点都记录了下来，我们只需将这些数据传输到后台，进过后台的分析来判断哪些部分页面耗时过长需要优化。
