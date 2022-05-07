---
title: 使用jquery的ajax接收处理form提交后返回的数据
date: 2020-06-20 11:00:00
categories:
- 技术博客
tags:
- Jquery
- Ajax
- 文件上传





---



## 1、分析

原生的form表单是采用单向传输的方式传输数据，因此在submit事件发生后，就不受我们的控制了。

当然我们可以不采用form表单提交的方式，但是也有需要用到form表单与后台服务器进行交互的时刻。因此本文将介绍如何一个工具，方便我们在编写前后端交互的代码时，可以使用简洁的form提交，来进行业务操作。



## 2、引入

首先我们需要到JQuery的Form项目中找到官方的js文件，以便我们进行引入：

```http
https://github.com/jquery-form/form
```

![image-20200620105130360](%E4%BD%BF%E7%94%A8jquery%E7%9A%84ajax%E6%8E%A5%E6%94%B6%E5%A4%84%E7%90%86form%E6%8F%90%E4%BA%A4%E5%90%8E%E8%BF%94%E5%9B%9E%E7%9A%84%E6%95%B0%E6%8D%AE/image-20200620105130360.png)

可通过Download直接下载



## 3、官方API文档

```html
https://github.com/jquery-form/form#api
```

其中，如果并不是想像文档中一样实现较复杂的交互，可直接在ajaxSubmit中传入一个回调function，如

```javascript
//其中function是一个回调function
//msg则是后台服务器返回给前端的字符串
$('#form-id').ajaxSubmit(function (msg) {
    alert(msg)
    console.log(msg)
})
```



