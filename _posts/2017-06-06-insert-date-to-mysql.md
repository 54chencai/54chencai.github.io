---
layout: post
title: "通过JDBC向Mysql数据库批量写入数据"
date: 2017-06-06 18:13:04
categories: java
tags: mysql JDBC 
---

* content
{:toc}

通过Java  JDBC协议向数据库中写入批量数据,用于后续增删改查的练习,期间会用到apache组织编写的dbutils可以简化和数据库的交互.



## 1.编写数据库连接的工具类,做到代码的可重复利用,方便开发
为了见名知意,工具名称为:JDBCUtils.实现了获得连接,获得连接池对象以及连接的释放
``` java
public class JDBCUtils {

    // 创建c3p0的连接池
    static DataSource cpds = new ComboPooledDataSource();

    // 获取连接池对象
    public static DataSource getDataSource() {
        return cpds;
    }

    // 获得连接
    public static Connection getConnection() throws Exception {
        // 获得连接
        return cpds.getConnection();
    }

    // 释放资源
    public static void release(Connection conn, Statement stmt, ResultSet rs) {
        try {
            if (rs != null) {
                rs.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        rs = null;

        release(conn, stmt);
    }

    // 释放资源
    public static void release(Connection conn, Statement stmt) {
        try {
            if (stmt != null) {
                stmt.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        stmt = null;

        try {
            if (conn != null) {
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        conn = null;
    }
}
```
## 2.利用dbutils向数据库中写入数据
``` java
public void insertDate() {
    //创建dbutils中操作数据库的QueryRunner对象
    QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource);
    //书写操作数据库的语句
    /*
     *数据库为user(id int primery key auto increament,name varchar(20),gender varchar(20),birthday varchar(20),phone varchar(20));
    */
    String sql = "insert into user values(null,?,?,?,?)";
    //获取语句中的参数信息
    User user = new User();
    for(int i = 0; i < 200; i++) {
        //设置传入姓名
        user.setName="name" + i;
        //设置传入性别
        int j = new Random().nextInt(2) + 1; //生成随机树1或者2
        user.setGender = (j % 2 == 0) ? 男 : 女
        //设置传入生日
        user.setBirthday = "1991-12-24";
        //设置手机号
        String num = "18710832";
        user.setPhone(num + (i+100)); 
        
        //执行sql语句
        try {
                qr.update(sql, user.getName(), user.getGender(), user.getBirthday(), user.getPhone());
            } catch (SQLException e) {
                e.printStackTrace();
                throw new RuntimeException("传入数据到数据库失败");
            }
    }
   
   
}
```

![2017-06-06-insert-date-to-mysql](http://github.com/54chencai/image/20170606/2017-06-06-insert-date-to-mysql.PNG "mySql")