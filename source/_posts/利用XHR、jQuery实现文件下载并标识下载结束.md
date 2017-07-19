---
title: 利用XHR、jQuery实现文件下载并标识下载结束
date: 2017-05-09 19:40:48
tags: javascript
---

## 问题描述

实现文件下载，点击下载按钮出现遮罩/提示层，等待后台文件准备完毕弹出另存为窗口，遮罩层消失。

## 问题所在

传统的文件下载可以实现点击以后出现遮罩层，但是由于无法判断文件何时传输完毕，导致弹出另存为窗口时无法找到事件钩子或回调函数以实现遮罩层消失。

## 背景

### XHR-XMLHttpRequest

XHR其实我们平时很经常的在用到，它是Ajax实现的基础，所有现代浏览器均支持XHR。它可以与服务器异步通信，所以可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

主要用到的函数与变量主要有：

1. open(*method*,*url*,*async*)

规定请求的类型、URL 以及是否异步处理请求。

- *method*：请求的类型；GET 或 POST
- *url*：文件在服务器上的位置
- *async*：true（异步）或 false（同步）

2. send(*string*)

发送请求，当POST请求时，用到string参数。

3. setRequestHeader(*header*,*value*)

设置HTTP头的函数。

4. responseText & responseXML & response

XHR对象属性，用于获取服务器响应。

##### 下面的是解决本问题所用到的额外的几个参数

5. responseType

XHR对象属性，设置想要获得的response类型，常用的有下面几个：

| Value           | Data type of `response` property         |
| --------------- | ---------------------------------------- |
| `""`            | [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) (默认值) |
| `"arraybuffer"` | [`ArrayBuffer`](https://developer.mozilla.org/en-US/docs/Web/API/ArrayBuffer) |
| `"blob"`        | [`Blob`](https://developer.mozilla.org/en-US/docs/Web/API/Blob) (用于文件下载等大量数据) |
| `"document"`    | [`Document`](https://developer.mozilla.org/en-US/docs/Web/API/Document) |
| `"json"`        | 由服务端返回的JSON字符串所构造的JavaScript 对象          |

6. onload事件

当请求成功完成时触发，此时`xhr.readystate=4`，利用onload回调，可以在此实现文件下载完毕时关闭遮罩层。

### Blob简介

BLOB (binary large object)，二进制大对象，是一个可以存储二进制文件的容器。
在计算机中，BLOB常常是数据库中用来存储二进制文件的字段类型。
BLOB是一个大文件，典型的BLOB是一张图片或一个声音文件，由于它们的尺寸，必须使用特殊的方式来处理（例如：上传、下载或者存放到一个数据库）。

利用Blob，我们的问题得以解决。说到Blob，就不得不说起利用***window.URL*** or ***window.webkitURL***实现下载Blob的问题啦

***URL.createObjectURL()*** 方法会根据传入的参数创建一个指向该参数对象的URL。这个URL的生命仅存在于它被创建的这个文档里。 新的对象URL指向执行的File对象或者是Blob对象。

***URL.revokeObjectURL()* **方法会释放一个通过URL.createObjectURL()创建的对象URL，当你要已经用过了这个对象URL，然后要让浏览器知道这个URL已经不再需要指向对应的文件的时候，就需要调用这个方法。

具体的意思就是说，一个对象URL，使用这个url是可以访问到指定的文件的，但是我可能只需要访问一次，一旦已经访问到了，这个对象URL就不再需要了，就被释放掉。被释放掉以后，这个对象URL就不再指向指定的文件了。

### 实现思路

利用XHR实现把后台传输的文件以Blob格式存储，在点击下载按钮时，发送XMLHttpRequest请求，在onload回调中创建虚拟的一个a标签并将其绑定到Blob对象，然后模拟点击URL下载的操作实现下载。弹出另存为窗口后，也即在onload事件的最后隐藏遮罩层。搞定~

## 代码

```javascript
$(document).ready(function() {
  $(".submit").click(function() {
    $('.wrd-loading').show();
    var oReq = new XMLHttpRequest();
    oReq.open("GET", "{{ action('System\LogController@getDownloadLogFile')  }}", true);
    oReq.responseType = "blob";

    oReq.onload = function(oEvent) {
      var blob = oReq.response;
      let a = document.createElement('a');
      let myURL = window.URL
      if(myURL.createObjectURL == undefined)
        myURL = window.webkitURL
      let url = myURL.createObjectURL(blob);
      let filename = 'api_push.zip';
      a.href = url;
      a.download = filename;
      a.click();
      myURL.revokeObjectURL(url);
      $('.wrd-loading').hide();
    };
    oReq.send();
  });
});
```
后台只需按正常方式发送即可，如php可能就是echo file_get_contents();

## Reference

1. [你真的会使用XMLHttpRequest吗？]: https://segmentfault.com/a/1190000004322487

2. [XMLHttpRequest.responseType]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseType

3. [URL.createObjectURL和URL.revokeObjectUR]: http://www.mamicode.com/info-detail-456059.html
