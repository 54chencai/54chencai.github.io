---
layout: post
title: "四大域对象的创建和销毁"
date: 2017-06-09 20:15:54
categories: java
tags: ServletContext HttpSession ServletRequest PageContext
---

* content
{:toc}


四大域对象的创建和销毁



## 1. ServletContext
创建:加载有web应用的服务器开启时
消毁:加载有web应用的服务器关闭时

## 2. HttpSession
在服务器中,为浏览器创建独一无二的内存空间,保存会话相关的信息
创建:
  * 第一次调用request.getSession()时(服务器中没有该浏览器的session时)
  * 浏览器第一次访问jsp页面时
消毁: 
  * 一段时间内session没有被使用时(默认为30分钟)
  * 服务器非正常关闭时
  	- 服务器正常关闭,再启动,session会进行钝化和活化操作
  	- 如果服务器钝化时间在session默认销毁时间内,则活化后session还存在
  	- 如果javaBean数据在session钝化时,没有实现Serializable接口,则当session活化时,会消失

## 3. ServletRequest
创建:每次请求servlet时都会执行service方法从而由服务器创建request
消毁: 整个请求结束时(一般在执行response后) 
	  
## 4. PageContext
创建:对JSP页面发送请求时
消毁:页面响应结束时
可以获取其它域中的数据