---
layout: post
title: "实现类似亚马逊的分类搜索功能"
date: 2017-06-06 14:33:24
categories: java
tags: JDBC servlet html 
---

* content
{:toc}

实现类似亚马逊网站的分类搜索



## 1.大体外观框架的实现
包括下拉菜单:可以通过html的select标签实现
搜索框:可以通过html input标签 type="text"实现
搜索按钮:可以通过html input标签的type="submit"实现
```html
<div>
<form style="text-align:center">
    <select name="select">
       <option value="id">id</option>
        <option value="name">姓名</option>
        <option value="gender">性别</option>
        <option value="birthday">出生日期</option>
        <option value="phone">手机号码</option>
    </select>
    <input type="text"  name="msg"  style='width:30%'>
    <input type="submit" value="搜索">
</form>
</div>
```

## 2.编写servlet

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // 1. 获得用户在搜索框选择的类别以及输入的关键字
        request.setCharacterEncoding("utf-8");
        String userSelect = request.getParameter("select");
        String userSearch = request.getParameter("search");
        // 2. 调用业务层,处理获得的数据
        UserService userService = new UserServiceImpl();
        List<User> someUserList = userService.queryUser(userSelect, userSearch);
        System.out.println(someUserList);
        // 3. 处理返回的数据
        // 3.1 将数据储存到request
        request.setAttribute("someUserList", someUserList);
        // 3.2 请求转发到结果页面
        request.getRequestDispatcher("/searchResult.jsp").forward(request, response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }
```

## 3.服务层接口

```java
public interface UserService {

    public List<User> queryUser(String userSelect, String userSearch);

}
```

## 4.实现接口

```java
public class UserServiceImpl implements UserService {

    @Override
    public List<User> queryUser(String userSelect, String userSearch) {
        UserDao userDao = new UserDaoImpl();
        List<User> list = userDao.getSelect(userSelect, userSearch);
        return list;
    }

}
```

## 5.dao层接口

```java
public interface UserDao {
     public List<User> getSelect(String userSelect, String userSearch);

}
```

## 6.实现dao层接口

```java
// 获得用户选择项目的所有数据集合
public List<User> getSelect(String userSelect, String userSearch) {
    // 1.创建QueryRunner对象
    QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());
    // 2.书写数据库语句
    String sql = "select * from user where " + userSelect + " like ? ";
    System.out.println(userSelect);
    System.out.println(userSearch);
    // 3.执行数据库语句
    try {
        List<User> list = qr.query(sql, new BeanListHandler<User>(User.class), "%" + userSearch + "%");
        System.out.println(list);
        // 4.查询成功
        return list;
    } catch (SQLException e) {
        e.printStackTrace();
        // 5.查询失败
        throw new RuntimeException("查询失败");
    }
}
```
