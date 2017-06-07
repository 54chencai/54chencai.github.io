---
layout: post
title: "查询并打印数据库中的数据到网页"
date: 2017-06-06 13:43:54
categories: java
tags: JDBC servlet html
---

* content
{:toc}

通过点击浏览器的"查询所有用户信息"连接,返回数据库中的用户信息



## 1.通过jsp(html)编写"查询所有用户信息"网页.
连接`href='${pageContext.request.contextPath }/queryAll'`
其中${pageContext.request.contextPath }为同过EL表达式获得当前项目路径
*queryAll*当点击链接十调用queryAllServlet
```html 
<body>
    <h1>查询页面</h1>
    <h3><a href="${pageContext.request.contextPath }/queryAll">查询所有用户信息</a></h3>
</body>
```

## 2.编写QueryAllServlet获得浏览器请求,并做出响应
```java
public class QueryAllServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // 浏览起请求为获得所有用户数据
        // 1.调用业务层方法获得所有用户数据
        UserService userService = new UserserviceImpl();
        List<User> userList = userService.queryAllUser();
        // 2.获得的所有用户信息
        // 2.1将用户信息封装到request容器中
        request.setAttribute("userList", userList);
        // 2.2请求转发到显示页面
        request.getRequestDispatcher("list.jsp").forward(request, response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }

}
```

## 3. 编写业务层,业务层从数据库中取得数据返回给Servlet

### 3.1 首先定义业务层的接口
接口中定义查询所有用户的方法queryAllUser()
```java
public interface UserService {
    //查询所有用户
    public List<User> queryAllUser();

}
```

### 3.2 实现接口,从数据哭获取数据(用了apache的dbutils工具)
```java
public class UserDaoImpl implements UserDao {

    @Override
    // 重写UserDao接口中的查询所有用户信息的方法
    public List<User> getAllUser() {
        // 用apache组织的dbutils工具实现和数据库的交互
        // 1.创建QueryRunner对象,
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());
        // 2.书写操作数据库的语句
        String sql = "select * from user";
        // 3.执行语句获得数据
        try {
            List<User> userList = qr.query(sql, new BeanListHandler<User>(User.class));
            //获取所有用户成功,返回用户数据
            return userList;
        } catch (SQLException e) {
            e.printStackTrace();
            //获取所用用户失败,抛出异常
            throw new RuntimeException("获取所有用户失败!");
        }
    }

}
```

## 4.编写网页接收并显示数据

### 4.1建立显示用户信息页面,通过EL表达式${userList}测试能否返回数据
```html
<body>
${userList }
</body>
```

### 4.2编写用户信息的显示格式
设置页面显示格式,有用到table,标准标签库c,css并结合EL表达式完成.
需要注意的是:标准标签库中的<c:foreach>标签的属性
varStatus:
*指定将代表当前迭代状态信息的对象保存到page这个Web域中的属性名称*
count：当前已循环迭代的次数
index：当前迭代的索引号

```html
<body>
<%-- ${userList } --%>

<%--用户数据为空,打印数据库中无用户信息 --%>
<c:if test="${empty userList }">
<h2>数据库中无用户信息</h2>
</c:if>

<%--用户数据不为空,打印用户数据到浏览器 --%>
<c:if test="${not empty userList }">
    <h1 class="center">用户信息表</h1>
    <table border="1" align="center" cellspacing="0" cellpadding="4">
        <tr>
            <th>编号</th>
            <th>ID</th>
            <th>姓名</th>
            <th>性别</th>
            <th>生日</th>
            <th>电话</th>
        </tr>
        <c:forEach items="${userList }" var="user" varStatus="i">
        <tr class="center">
            <td>${i.count }</td>
            <td>${user.id }</td>
            <td>${user.name }</td>
            <td>${user.gender }</td>
            <td>${user.birthday }</td>
            <td>${user.phone }</td>
        </tr>
        </c:forEach>
    </table>
</c:if>
</body>
```

![2017-06-06-queryAll-user](http://github.com/54chencai/image/20170606/2017-06-06-queryAll-user.PNG "quaryAll")