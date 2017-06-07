---
layout: post
title:  "remember user's name & password for auto log in"
date:   2017-06-07 15:40:00 
categories: java
tags: session cookie filter
---

* content
{:toc}


使用cookie,session和filter完成用户自动登陆



## Servlet
1. 根据用户输入的用户名和密码到数据库查询是否存在
2. 如果不存在
  * 将需返回的信息封装到request中
  * 请求转发到login页面
3. 如果存在
  * 将用户信息封装到session中
  * 用户有勾选自动登录
    - 将用户名和密码储存到cookie中
    - 重定向到主页
  * 用户没有勾选自动登录
    - 将cookie中储存的用户名和密码删除
    - 重定向到主页
```java
//获得用户名和密码
String name = request.getParameter("name");
String password = request.getParameter("password");
//连接数据库验证
QuaryRunner qr = new QuaryRunner(JDBCUtils.getDataSouce);
String sql = "select * from user where name=? and password=?";
User exitUser = qr.quary(sql,new BeanHandler<User>(User.class),name,password);
//判断exitUser是否为null
if(exitUser != null) {
    //将用户信息封装到session中
    request.getSession().setAttribute("exitUser",exitUser);
    String autoLogin = request.getParameter("autologin");
    if(autoLogin != null) {
        Cookie cookie = new Cookie("userMsg",name + "chencai" + password);
        cookie.setMaxAge(60*60*24*30);
        cookie.setPath("/");
        response.addCookie(cookie);
        request.getRequestDispatcher("index.jsp").forword(request,response);
        return;
    }else {
        Cookie cookie = new Cookie("userMsg","");
        cookie.setMaxAge(0);
        cookie.setPath("/");
        response.addCookie(cookie);
        request.getRequestDispatcher("index.jsp").forword(request,response);
        return;
    }
} else {
    request.setAttribute("error","用户名或密码错误");
    request.getRequestDispatcher("login.jsp").forword(request,response);
    return;
}
```

## Filter
过滤器作用页面为主页,利用filter拦截主页的打开
* 用户已经登录,放行
* 用户未登录,获取存储用户信息的cookie
* 保存用户的cookie不存在,放行
* 保存用户的cookie存在
  1. 将cookie解码为用户名和用户密码
  2. 连接数据库获得用户信息
  3. 将获得的用户信息保存到session中
  4. 放行.
```java
//判断ession中是否存有用户信息,需要通过HttpServeletRequest获得,filter中的request需要向下强转
HttpServeletRequest httpServeletRequest = (HttpServeletRequest) request;
//获取并判断session中是否存储有用户信息
User exitUser = httpServeletRequest.getSession().getAttribute("exitUser");
if(exitUser != null) {
    //存在直接放行
    chain.doFilter(HttpServeletRequest,response);
}else {
    //不存在
    Cookie[] cookies = request.getCookies();
    String name;
    Stirng password;
    if(cookies != null) {
        for(Cookie cookie : cookies) {
            if(cookie.equels("userMsg")) {
                String[] userArr = cookie.getValue().split(chencai);
                name = userArr[0];
                password = userArr[1];
            }
        }
    }
    QueryRunner qr = new QueryRunner(JDBCUtils.getDataSouce);
    String sql = "select * from user where name=? and password=?";
    User exitUser = qr.quary(sql,new BeanHandler<User>(User.class);
    if(exitUser != null) {
        httpServeletRequest.getSession(),setAttribute("exitUser",exitUser);
    }
}
//放行
chain.doFilter(httpServeletRequest,response);
```
  
