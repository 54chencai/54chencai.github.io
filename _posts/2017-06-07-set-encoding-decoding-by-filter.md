---
layout: post
title:  "set encording and decoding by filter"
date:   2017-06-07 14:40:09 
categories: java
tags: filter encoding decoding
---

* content
{:toc}

由于编解码的不一致,常常回出现乱码,我们必须在从浏览器读取数据和返回数据时进行编解码,而通过在filter中对request对象进行包装增强从获得参数的方法:
getParameter
getParameterValues
getParameterMap
可以解决乱码的问题



## 通过对获得的数据进行编重新编解码来解决乱码
可以使用String的构造方法`new String(param.getBayes("iso8859-1"),"utf-8")`设置编解码方式
```java
/**
 * 解决get和post乱码问题: 使用包装进行增强
 */
public class GenericEncodingFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        // 向下转型
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        // 使用包装进行增强
        HttpServletRequest myRequest = new MyRequest(httpServletRequest);
        // 放行
        chain.doFilter(myRequest, response);
    }

    public void destroy() {
    }

    public void init(FilterConfig fConfig) throws ServletException {
    }

    class MyRequest extends HttpServletRequestWrapper {

        // 为了使用其他方法都是request对象,声明一个成员变量
        private HttpServletRequest httpServletRequest;

        // 定义变量: 保存是否已经编码的状态
        private boolean isEncoding = false;

        public MyRequest(HttpServletRequest httpServletRequest) {
            super(httpServletRequest); // 注意这句话必须有,否则报错
            this.httpServletRequest = httpServletRequest;
        }

        @Override
        public Map<String, String[]> getParameterMap() {
            // 1 获取方法类型
            String method = httpServletRequest.getMethod();

            if ("post".equalsIgnoreCase(method)) {
                // 2.1 如果是post方式,设置请求编码集
                try {
                    httpServletRequest.setCharacterEncoding("utf-8");
                    return httpServletRequest.getParameterMap();
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            } else if ("get".equalsIgnoreCase(method)) {
                // 2.2 如果是get方式,先编码,再解码
                Map<String, String[]> parameterMap = httpServletRequest.getParameterMap();
                if (!isEncoding) {
                    for (Entry<String, String[]> en : parameterMap.entrySet()) {
                        String[] valArr = en.getValue();
                        if (valArr != null) {
                            for (int i = 0; i < valArr.length; i++) {
                                try {
                                    valArr[i] = new String(valArr[i].getBytes("iso-8859-1"), "utf-8");
                                } catch (UnsupportedEncodingException e) {
                                    e.printStackTrace();
                                }
                            }
                        }
                    }

                    // 编码后 修改状态
                    isEncoding = true;
                }
                return parameterMap;
            }
            return super.getParameterMap();
        }

        @Override
        public String getParameter(String name) {
            // 获取map
            Map<String, String[]> parameterMap = this.getParameterMap();
            String[] valArr = parameterMap.get(name);
            if (valArr != null) {
                return valArr[0];
            } else {
                return null;
            }
        }

        @Override
        public String[] getParameterValues(String name) {
            return this.getParameterMap().get(name);
        }

    }

}
```

