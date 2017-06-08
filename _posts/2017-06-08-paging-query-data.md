---
layout: post
title:  "分页显示查询到的数据"
date:   2017-06-08 11:02:00 
categories: java
tags: limit mysql
---

* content
{:toc}

实现分页查询,一般有两种方法,一种是基于数据库对sql语句支持的物理分页
一种是基于算法的逻辑分页.




## 物理分页
mysql提供了limit关键字,oracle提供了rownum关键字
mysql: `select * from tableName limit startIndex,length;`
  - startIndex: 数据表的记录,从0开始
  - length:从startIndex开始查询多少条记录
模拟分页显示,则查询长度应为每页显示的数据条数,当length=10时当前页码如下表

|当前页码|开始下标|查询长度|
|--|--|--|
|1	 |0	    |10	|
|2	 |10	|10	|
|3	 |20	|10	|
|4	 |30	|10	|
|pageNum|    |numPerPage|

则查询的开始下标:`startIndex=(pageNum-1)*numPerPage`
```java
pubic void demo() {
    //测试物理分页,每页显示10条数据,查询第5页
    QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource);
    //查询第五页,也就是[50~60)数据
    int numPerPage = 10;
    int pageNum = 5;
    int startIndex = (pageNum-1) * numPerPage;
    String sql = "select * from user limit ?,?"
    List<User> users = qr.query(sql,BeanListHandler<User>(User.class),startIndex,numPerPage);
    //显示查询数据
    for(User user : users) {
        system.out.println(user);
    }
}
```

## 逻辑分页
一次性查询所有数据,再截取需要的数据,通过subList(start,end)截取需要的数据

当前页码|开始下标|结束下标|长度
--|--|--|--
1   |0   |10 |10
2   |10  |20 |10
3   |20  |30 |10
4   |30  |40 |10
5   |40  |50 |10

```java
    @Test
    public void demo2() throws SQLException {
        // 测试逻辑分页,每页显示10条,查询第五页数据
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());
        String sql = "select * from user";

        List<User> users = qr.query(sql, new BeanListHandler<User>(User.class));

        int numPerPage = 10;
        int pageNum = 5;
        int start = (pageNum - 1) * numPerPage;
        int end = pageNum * numPerPage;
        users = users.subList(start, end);
        for (User user : users) {
            System.out.println(user);
        }
    }
```

## 分页应用场景
物理分页需要先获取所有的数据,因此无法单独应用在大规模数据中
在实际中这两种分页通常回结合使用

