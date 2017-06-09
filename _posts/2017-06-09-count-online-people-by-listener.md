---
layout: post
title: "利用监听器实现统计在线人数"
date: 2017-06-09 19:22:54
categories: java
tags: listener
---

* content
{:toc}

利用监听器实现统计在现人数
ServletContextListener在项目启动创建,项目关闭销毁,可以用来计录总在线人数
新用户打开jsp网页都会新建seeion对象,触发HttpSessionListenser,可以用来统计人数



## 1. 利用ServletContextListener初始化并计录总在线人数
```java
public void contextIntialized(ServletContexEvent sce) {
    //1.获取ServletContext对象
    ServletContext sc = sce.getServletContext();
    //2.计录并初始化在线人数为0
    sc.setAttribute("onlineCount",0);
}
```

## 2. 利用HttpSessionListener的创建与消毁改变总在线人数
```java
public void  sessionCreated(HttpSessionEvent hse) {
    //获取HttpSession对象
    HttpSession hs = hse.getSession();
        //加锁
       synchronized(this){
        //获取ServletContext对象
        ServletContext sc = hs.getServletContext();
        int onlineCount = (int) sc.getAttribute("onlineCount");
        //实现创建session总人数+1
        sc.setAttribute("onlineCount",onlineCount+1);
    }   
}

public void sessionDestroyed(HttpSessionEvent hse) {
      //获取HttpSession对象
    HttpSession hs = hse.getSession();
        //加锁
       synchronized(this){
        //获取ServletContext对象
        ServletContext sc = hs.getServletContext();
        int onlineCount = (int) sc.getAttribute("onlineCount");
        //实现消毁session总人数-1
        sc.setAttribute("onlineCount",onlineCount-1);
    }
}
```

