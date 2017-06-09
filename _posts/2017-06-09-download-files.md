---
layout: post
title: "遍历本地文件并下载
date: 2017-06-09 18:35:26
categories: java
tags: iostream jsp
---

* content
{:toc}

利用递归遍历本地文件目录,利用servelet和IO流完成下载



## 1. 在jsp网页中遍历本地文件目录

```js
//获得本地文件根目录
    File root = new File("F:\\周杰伦");
    //将根目录存储在linkedList中（模拟队列，先进先出）
    LinkedList<File> list = new LinkedList<File>();
    list.addLast(root);
    //遍历list
    while(!list.isEmpty()) {
        //从队列中读取目录
        File path = list.removeFirst();
        //便利目录
        File[] files = path.listFiles();
        if(files != null) {
            for(File file : files) {
                if(file.isFile()) {
                    out.println("<a href='/day24/downloadList?filename=" + file.getCanonicalFile()  + "' >" + file.getName()+ "</a>");
                    out.println("<br/>");
                }else {
                    list.addLast(file);
                }
            }
        }
    }
```

## 2. servlet中设置java下载

```java
// 获得文件路径
String filepath = request.getParameter("filepath");
// 设置编解码
filepath = new String(filepath.getBytes("iso8859-1"), "uft-8");
// 获得文件名称
int index = filepath.lastIndexOf("\\");
String filename = filepath.substring(index - 1);
// 设置contextType
response.setContentType(this.getServletContext().getMimeType(filename));
// 设置setHeader
response.setHeader("content-disposition", "attachment;filename = " + filename);
// IO流
InputStream in = new BufferedInputStream(new FileInputStream(filepath));
OutputStream out = new BufferedOutputStream(response.getOutputStream());

int b;
while ((b = in.read()) != -1) {
    out.write(b);
}
// 关流
in.close();
```

## 3. 解决下载文件名乱码

```java
String userAgent = request.getHeader("user-agent");
if (userAgent.contains("Firefox") || userAgent.contains("Chorme")) {
    BASE64Encoder base64Encoder = new BASE64Encoder();
    filename = "=?utf-8?b?" + base64Encoder.encode(filename.getBytes("utf-8")) + "?=";
} else {
    filename = URLEncoder.encode(filename, "utf-8");
    filename = filename.replace("+", " ");
}
```

