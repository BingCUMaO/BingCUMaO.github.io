---
title: 解决github访问慢问题
date: 2020-05-30 14:13:40
categories:
- 技术博客
tags:
- Github
- 网络


---



#### 首先我们需要找到TTL跳数较小的服务器IP

```http
DNS查询网站:
http://tool.chinaz.com/dns
```
![在这里插入图片描述](%E8%A7%A3%E5%86%B3github%E8%AE%BF%E9%97%AE%E6%85%A2%E9%97%AE%E9%A2%98/20200327115419324.png)

### 需要修改hosts文件
windows系统的hosts文件的位置如下：C:\Windows\System32\drivers\etc\hosts
mac/linux系统的hosts文件的位置如下：/etc/hosts

##### 由于是系统文件，所以修改前记得获取权限
##### 之后将hosts用文本格式打开，在末尾追加我们查询到TTL跳值较小的IPv4地址
我个人使用以下IP，效果蛮好的
```
140.82.114.4	github.com
199.232.5.194	github.global.ssl.fastly.net
```
#### 如图：

![在这里插入图片描述](%E8%A7%A3%E5%86%B3github%E8%AE%BF%E9%97%AE%E6%85%A2%E9%97%AE%E9%A2%98/20200327114700595.png)

###### 保存之后再打开github.com，试试看访问速度是否已经像访问国内服务器一样了