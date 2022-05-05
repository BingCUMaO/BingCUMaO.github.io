---
title: Axios实现组件化文件下载
date: 2020-08-02 21:00:00
categories:
- 技术博客
tags:
- axios
- js
- 文件下载



---



# 1、第一步

- 首先为整个项目配置一些文件下载的公共部分：

```js
import axios from 'axios'

axios.defaults.headers['Content-Type'] = 'application/json;charset=utf-8'
// 创建axios实例
const active = axios.create({
  // axios中请求配置baseURL选项，请求URL中的公共部分
  baseURL: process.env.VUE_APP_BASE_API,
  timeout: 10000		//设置超时时限
})

export default request
```







# 2、第二步

- 封装具体的文件下载组件：


```js
//导入上面的request.js组件，对文件下载进行下一步的封装
import request from '@/utils/request'

//这里以下载pdf文件为例
//其中query中有文件下载请求的身份信息和一个uuid
export function downloadPdf(query) {
  return request({
    url: '/bingcu/downloader/pdf',
    method: 'post',
    params: query,
    responseType: 'blob'
  }).then(response =>{
     //这里直接单次调用了，如有需要，可以分离出去
    (function (response) {
      //判断response是否异常
      if (!response) {
        return
      }
      //将二进制数据保存到本地缓存
      let url = window.URL.createObjectURL(new Blob([response]))
      let link = document.createElement('a')
      //隐藏该链接
      link.style.display = 'none'
      link.href = url
      //这里的文件名就直接用query对象中的一个uuid身份信息来命名
      link.setAttribute('download', query.uuid+'.pdf')		

      //模拟点击该链接
      document.body.appendChild(link)
      link.click()
    })(response)
  }).catch(error =>{
    alert("失败，请重试")
  })
}
```

# 第三步

- 剩下第三步就是关于`query`具体有哪些参数了，但这已经是与后端的交互问题了，由于作者在其他项目的文件下载中需要对身份进行验证，所以就假设一下`query`对象的属性就几乎都是关于身份信息的相关数据了。

