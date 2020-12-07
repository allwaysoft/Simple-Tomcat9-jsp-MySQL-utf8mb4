Tomcat JSP 使用 MySQL utf8mb4字符集避免中文乱码的必要配置

allway2 2020-12-07 11:22:59  12  收藏
编辑 版权
1.简介
 
 
UTF-8是Web应用程序中最常用的字符编码。它支持世界上目前使用的所有语言，包括中文，韩语和日语。

在本文中，我们演示了确保Tomcat中的UTF-8所需的所有配置。

 
2.连接器配置
 
连接器侦听特定端口上的连接。我们需要确保所有连接器都使用UTF-8编码请求。

让我们将参数URIEncoding =“ UTF-8”添加到TOMCAT_ROOT / conf / server.xml中的所有连接器：

<Connector 
  URIEncoding="UTF-8" 
  port="8080" 
  redirectPort="8443" 
  connectionTimeout="20000" 
  protocol="HTTP/1.1"/>
 
<Connector 
  URIEncoding="UTF-8" 
  port="8009" 
  redirectPort="8443" 
  protocol="AJP/1.3"/>
3.字符集过滤器
 
配置完连接器之后，是时候强制Web应用程序处理UTF-8中的所有请求和响应了。

让我们定义一个名为CharacterSetFilter的类：

public class CharacterSetFilter implements Filter {
 
    // ...
 
    public void doFilter(
      ServletRequest request, 
      ServletResponse response, 
      FilterChain next) throws IOException, ServletException {
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html; charset=UTF-8");
        response.setCharacterEncoding("UTF-8");
        next.doFilter(request, response);
    }
 
    // ...
}
我们需要将过滤器添加到应用程序的web.xml中，以便将其应用于所有请求和响应：

<filter>
    <filter-name>CharacterSetFilter</filter-name>
    <filter-class>com.baeldung.CharacterSetFilter</filter-class>
</filter>
 
<filter-mapping>
    <filter-name>CharacterSetFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
4.服务器页面编码
 
我们需要配置的Web应用程序的另一部分是Java服务器页面。

确保服务器页面中UTF-8的最佳方法是在每个JSP页面的顶部添加此标记：

<%@page pageEncoding="UTF-8" contentType="text/html; charset=UTF-8"%>
<%@ page language="java" pageEncoding="UTF-8"%>
<%@ page contentType="text/html;charset=UTF-8"%>
<%
request.setCharacterEncoding("UTF-8");
response.setCharacterEncoding("UTF-8");
response.setContentType("text/html; charset=UTF-8");
%>
5. HTML页面编码
 
服务器页面编码告诉JVM如何处理页面字符，而HTML页面编码告诉浏览器如何处理页面字符。

我们应该在所有HTML页面的头部添加此<meta>标记：

<meta http-equiv='Content-Type' content='text/html; charset=UTF-8' />
6. MySQL服务器配置
 
现在，我们的Tomcat已配置完毕，该配置数据库了。

 
 
 
我们假设使用了MySQL服务器。该配置文件在Windows上名为my.ini，在Linux上名为my.cnf。

我们需要找到配置文件，搜索这些参数，并进行相应的编辑：

[client]
default-character-set = utf8mb4
 
[mysql]
default-character-set = utf8mb4
 
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
我们需要重新启动MySQL服务器，以使更改生效。

7. MySQL数据库配置
 
MySQL服务器字符集配置仅适用于新数据库。我们需要手动迁移旧的。使用一些命令即可轻松实现。

对于每个数据库：

ALTER DATABASE database_name CHARACTER SET = utf8mb4 
    COLLATE = utf8mb4_unicode_ci;
对于每个表：

ALTER TABLE table_name CONVERT TO 
    CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
对于每个VARCHAR或TEXT列：

ALTER TABLE table_name CHANGE column_name column_name 
    VARCHAR(69) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
如果要在数据库查询中传递带有UTF-8字符的数据，则需要确保建立的任何数据库连接均符合UTF-8编码。

对于基于JDBC的连接，可以通过以下连接URL来实现：

jdbc:mysql://localhost:3306/?useUnicode=yes;characterEncoding=UTF-8
8.结论
 
在本文中，我们演示了如何确保Tomcat使用UTF-8编码。
