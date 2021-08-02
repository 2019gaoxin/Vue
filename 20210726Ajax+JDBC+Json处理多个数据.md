[TOC]

# 查询省份的所有信息

对比查询单个信息：

传送门：<https://blog.csdn.net/gao1213977085/article/details/119089627> 

​	

## 1.用JSON作为数据的格式

​	使用JDBC查询多个数据返回给前端，返回的数据会在一个字符串中，需要分割操作。所以这里在数据库中查询多个数据时使用JSON。

​	输出多个数据时使用JSON的格式更加简介和方便。JSON格式的数据输出到异步对象，异步对象接收数据之后，可以把JSON数据转换成JS中的JSON对象来用。通过对象来使用JSON数据时最方便的。

###1.1使用jackson

​	使用jackson工具库处理JSON结构的数据。

​	通过jackson可以把一个Java对象转换成一个JSON格式的字符串。

###1.2准备数据库

​	1.启动服务(用命令启动MySQL)

```bash
cd D:\AppDownload\mysql-8.0.25-winx64

net start mysql

##关闭
##net stop mysql

```



## 2.JSP

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
  <title>使用Ajax查询省份名称</title>
  <script type="text/javascript">
    function search() {

      //1.创建异步对象
      var xhr=new XMLHttpRequest();
      //2.绑定事件
      xhr.onreadystatechange = function () {
        if(xhr.readyState==4&&xhr.status==200){
          //获取数据
          var data = xhr.responseText;
          //data是字符串，必须转成对象才能识别。
          //alert("data=="+data);

          //把json格式的字符串 转为 json 对象。
          var obj = JSON.parse(data);
          //alert("data=="+obj+",id属性="+obj.id);
            
          //更新dom
          document.getElementById("proname").value=obj.name;
          document.getElementById("projiancheng").value=obj.jiancheng;
          document.getElementById("proshenghui").value=obj.shenghui;

        }

      }

      //3.初始请求参数
      //queryName?proid=1
      var proid = document.getElementById("proid").value;
      xhr.open("get","queryJson?proid="+proid,true);
      //4.发起请求
      xhr.send();

    }
  </script>
</head>
<body>
<div align="center">
  <p>根据id查询省份全部信息</p>
  <table>
    <tr>
      <td>省份id:</td>
      <td><input type="text" id="proid"></td>
    </tr>
    <tr>
      <td>省份名称</td>
      <td><input type="text" id="proname"></td>

    </tr>
    <tr>
      <td>省份简称:</td>
      <td><input type="text" id="projiancheng"></td>
    </tr>
    <tr>
      <td>省会名称:</td>
      <td><input type="text" id="proshenghui"></td>
    </tr>
    <tr>
      <td>操作</td>
      <td><input type="button" value="查询" onclick="search()"></td>
    </tr>
  </table>
</div>
</body>
</html>


```



## 3.实体类

​	用Java面向对象编程思想，将查询到的信息实例化。

```java
package com.gaoxin.entity;

public class Province {

    private Integer id;
    private String name;
    private String jiancheng;
    private String shenghui;

    public String getShenghui() {
        return shenghui;
    }

    public void setShenghui(String shenghui) {
        this.shenghui = shenghui;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getJiancheng() {
        return jiancheng;
    }

    public void setJiancheng(String jiancheng) {
        this.jiancheng = jiancheng;
    }
}

```



##4.DAO

​	与之前的不同点是返回一个对象。

```java
package com.gaoxin.dao;

import com.gaoxin.entity.Province;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class ProvinceDao {
    //定义方法：查询省份对象
    public Province queryProvinceName(Integer provienceId){
        String url = "jdbc:mysql://localhost:3306/springdb?useUnicode=true&characterEncoding=utf8";
        String usename = "root";
        String password = "123456";

        Connection conn = null;
        PreparedStatement pst=null;
        ResultSet rs=null;

        Province p=null;
        System.out.println("Dao执行");

        try{
            Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection(url,usename,password);

            String sql = "select * from province where id=?";
            pst =conn.prepareStatement(sql);
            pst.setInt(1,provienceId);

            rs = pst.executeQuery();

            while (rs.next()){//主键查询，使用if也可以。
                p = new Province();
                p.setId(rs.getInt("id"));
                p.setName(rs.getString("name"));
                p.setJiancheng(rs.getString("jiancheng"));
                p.setShenghui(rs.getString("shenghui"));
            }

        }catch (Exception e){
            e.printStackTrace();
        }finally {
            //关闭资源
            try{
                if(rs!=null){
                    rs.close();
                }
                if(pst!=null){
                    pst.close();
                }
                if(conn!=null){
                    conn.close();
                }
            }catch (Exception ex){
                ex.printStackTrace();
            }

        }

        return p;

    }


}

```



## 5.Servlet

​	在Servlet中把对象转为json，再通过HttpServletResponse输出。

```java

import com.fasterxml.jackson.databind.ObjectMapper;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class QueryProvinceServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String json ="";

        String strProid = request.getParameter("proid");

        //判断是否为空，并且不为空字符串
        if(strProid!=null&&!strProid.trim().equals("")){

            ProvinceDao dao = new ProvinceDao();
            Province p = dao.queryProvinceName(Integer.valueOf(strProid));

            //把对象转为json，才通过HttpServletResponse输出。

            if(p!=null){//说明查出了真实的对象。

                ObjectMapper om = new ObjectMapper();
                json = om.writeValueAsString(p);
                //得到一个JSON格式的字符串
            }
        }

        System.out.println("sevlet province转为json==="+json);

        //输出json给ajax请求。

        //application/json;给浏览器输出的是一个json格式的数据格式
        response.setContentType("application/json;charset=utf-8");

        PrintWriter out = response.getWriter();

        out.println(json);
        out.flush();
        out.close();

    }
}

```



## 6.web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>QueryProvinceServlet</servlet-name>
        <servlet-class>com.gaoxin.controller.QueryProvinceServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>QueryProvinceServlet</servlet-name>
        <url-pattern>/queryJson</url-pattern>
    </servlet-mapping>
</web-app>
```



​	

​	