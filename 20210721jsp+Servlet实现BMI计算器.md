计算BMI，并显示BMI的计算结果和建议。

BMI指数是体重公斤数除以身高

## Servlet

```java
package com.gaoxin.controller;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;

public class BMIServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //接收请求的参数
        String strName = request.getParameter("name");
        String strW = request.getParameter("w");
        String strH = request.getParameter("h");

        //接收到的是String
        //String转float
        System.out.println(strName);
        System.out.println(strH);
        float w = Float.valueOf(strW);
        float h = Float.valueOf(strH);

        //计算bmi 公式w /(h*h)

        float bmi =w/(h*h);
        System.out.println(bmi);

        String info="";
        if(bmi <18.5){
            info = "比较";
        }else if(bmi>=18.5&& bmi<24){
            info = "正常范围";
        }else{
            info = "比较胖";


        }

        //使用jsp

        //把数据保存到 request 作用域
        info = strName +"您的BMI是["+bmi+"],"+info;
        request.setAttribute("info",info);

        //需要使用视图，显示数据。
        request.getRequestDispatcher("/result.jsp").forward(request,response);

    }
}

```

## index.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: DELL
  Date: 2021/7/22
  Time: 9:32
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>全局刷新实现计算BMI</title>
  </head>
  <body>
    <div align = "center">

      <p>计算BMI</p>
      <!--action="/bmiServlet" 这个是两种方式-->
      <form action="s1" method="get">
        姓名：<input type="text" name="name"><br/>
        体重：<input type="text" name="w"><br/>
        身高：<input type="text" name="h"><br/>
        操作：<input type="submit" value="计算BMI"><br/>
      </form>
    </div>
  </body>
</html>

```



## result.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: DELL
  Date: 2021/7/22
  Time: 22:16
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>BMI</title>
</head>
<body>
    计算BMI的结果：${info}

</body>
</html>

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--计算BMI的Servlet -->
    <servlet>
        <servlet-name>s1</servlet-name>
        <servlet-class>com.gaoxin.controller.BMIServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>s1</servlet-name>
        <url-pattern>/s1</url-pattern>
    </servlet-mapping>
</web-app>
```

