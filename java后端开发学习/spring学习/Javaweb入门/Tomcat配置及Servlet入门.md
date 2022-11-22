# Tomcat配置及Servlet入门

> 此处学习内容援引自博客园”孤傲苍狼“[孤傲苍狼 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xdp-gacl/)

## 一、基本概念

Web资源分类：

- 静态web资源（如html页面）：指web页面中供人们浏览的数据是不变的。
- 动态web资源：指web页面中供人们浏览的数据由程序产生。

静态web资源开发技术：html

常用动态web资源开发技术：JSP/Servlet、ASP、PHP等。

静态web无法实现页面内容的动态更新，所有用户看见的内容相同，无法连接数据库，无法实现和用户的交互。

## 二、tomcat配置

### 2.1 虚拟目录映射

Web应用开发完成后，需要将web应用交给web服务器管理，这个过程称为虚拟目录映射。通过虚拟目录名，我们可以在浏览器中访问web资源。

- 在server.xml文件的host元素中配置（server.xml文件在tomcat目录的conf目录下）

```xml
<Host name="localhost"  appBase="webapps"
             unpackWARs="true" autoDeploy="true"
             xmlValidation="false" xmlNamespaceAware="false">

         <Context path="/JavaWebApp" docBase="F:\JavaWebDemoProject" />
 </Host>
```

以上代码中，`path`路径指的是web应用映射的虚拟目录，这个目录由Tomcat服务器管理，实际不存在；`docBase`指的是web应用在硬盘中的实际位置。

> 1. path必须以"/"开头。
> 2. Tomcat6后不建议直接在server.xml上修改，因为这需要Tomcat服务器重新启动后才能重新加载server.xml。

- 让tomcat服务器自动映射

可以将web应用直接复制到tomcat服务器的webapps目录中，tomcat服务器会自动将这个web应用映射为一个同名的虚拟目录。

- 在独立文件中配置context

在tomcat服务器的`\conf\Catalina\localhost`目录下添加一个以xml作为扩展名的文件，xml文件的名字可以任取，如aa.xml。虚拟目录的名称源于这个xml文件的名称。

在aa.xml中添加Context元素

```xml
<Context docBase="F:\JavaWebDemoProject" />
```

这个方法修改了配置文件后不需重启tomcat服务器。

### 2.2 配置虚拟主机

配置虚拟主机即配置一个网站，需要修改server.xml文件，可以看到tomcat服务器自带一个名称为localhost的虚拟主机。

```xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

    <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
    <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

    <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
           prefix="localhost_access_log" suffix=".txt"
           pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
```

将Javaweb应用放到webapps文件夹下，就可以通过`http://localhost:端口号/JavaWebAppName`的方式进行访问。这个虚拟主机管理webapps文件夹下的所有web应用。

可以通过如下方式配置新的虚拟主机

```xml
<Host name="www.gacl.cn" appBase="F:\JavaWebApps">
  
</Host>
```

虚拟主机的name是"www.gacl.cn"，管理着JavaWebApps下的所有web应用。这里的JavaWebApps文件夹代表的不是一个项目的根目录，而是一个存放了一个或者多个JavaWeb应用的文件夹。

配置的主机要想通过域名被外部访问，必须在DNS服务器或windows系统中注册访问的域名。

修改`C:\Windows\System32\drivers\etc`目录下的hosts文件，将新添加的域名与IP地址绑定起来。

![](.\1.png)

## 三、Servlet开发

Servlet是sun公司提供的一门用于开发动态web资源的技术。

### 3.1 Servlet运行过程

Servlet程序由Web服务器调用，web服务器收到客户端的Servlet访问请求后：

1. Web服务器首先检查检查是否已经装载并创建了该Servlet的实例对象。如果是，则直接执行第4步，否则，执行第2步。
2. 装载并创建该Servlet的一个实例对象。
3. 调用Servlet实例对象的init()方法。
4. 创建一个用于封装HTTP请求消息的HttpServletRequest对象和一个代表HTTP响应消息的HttpServletResponse对象，然后调用Servlet的service()方法并将请求和响应对象作为参数传递进去。
5. WEB应用程序被停止或重新启动之前，Servlet引擎将卸载Servlet，并在卸载之前调用Servlet的destroy()方法。



Servlet接口由SUN公司定义了两个默认实现类：GenericServlet、HttpServlet。

> HttpServlet指能够处理HTTP请求的servlet，它在原有Servlet接口上添加了一些与HTTP协议处理方法，它比Servlet接口的功能更为强大。因此开发人员在编写Servlet时，**通常应继承这个类**，而避免直接去实现Servlet接口。
> HttpServlet在实现Servlet接口时，覆写了service方法，该方法体内的代码会自动判断用户的请求方式，如为GET请求，则调用HttpServlet的doGet方法，如为Post请求，则调用doPost方法。因此，**开发人员在编写Servlet时，通常只需要覆写doGet或doPost方法，而不要去覆写service方法**。



在项目中的pom.xml文件引入`javax.servlet`依赖，从而来使用servlet。（**注意tomcat10.0后只支持jakarta.servlet**)

### 3.2 Servlet映射

Servlet程序要想被外界访问，首先需要映射到一个URL地址上，通过在web.xml文件上的<servlet>元素和servlet-mapping>元素完成。

```xml
<servlet>
	<servlet-name>HelloServlet</servlet-name>
	<servlet-class>org.example.HelloServlet</servlet-class>
</servlet>
    
<servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/demo/Hello</url-pattern>
</servlet-mapping>
```

<servlet-name>和<servlet-class>，分别用于设置Servlet的注册名称和Servlet的完整类名。<url-pattern>用于指定Servlet的对外访问路径。

**同一个Servlet可以被映射到多个URL上**，即多个<servlet-mapping>元素的<servlet-name>子元素的设置值可以是同一个Servlet的注册名。

**在Servlet映射到的URL中也可以使用\*通配符，但是只能有两种固定的格式：一种格式是"\*.扩展名"，另一种格式是以正斜杠（/）开头并以"/\*"结尾**

### 3.3 Servlet与普通Java类的区别

Servlet 是一个供其他 Java 程序（Servlet 引擎）调用的 Java 类，它不能独立运行，它的运行完全由 Servlet 引擎来控制和调度。
针对客户端的多次 Servlet 请求，通常情况下，**服务器只会创建一个 Servlet 实例对象**，也就是说 Servlet 实例对象一旦创建，它就会驻留在内存中，为后续的其它请求服务，直至 web 容器退出，servlet 实例对象才会销毁。
在 Servlet 的整个生命周期内，**Servlet 的 init 方法只被调用一次**。而对一个 Servlet 的每次访问请求都导致 Servlet 引擎调用一次 servlet 的 service 方法。对于每次访问请求，Servlet 引擎都会创建一个新的 HttpServletRequest 请求对象和一个新的 HttpServletResponse 响应对象，然后将这两个对象作为参数传递给它调用的 Servlet 的 service() 方法，service 方法再根据请求方式分别调用 doXXX 方法。

如果在 <servlet> 元素中配置了一个 `< load-on-startup >` 元素，那么 WEB 应用程序在启动时，就会装载并创建 Servlet 的实例对象、以及调用 Servlet 实例对象的 init() 方法。如

```xml
<servlet>
    <servlet-name>invoker</servlet-name>
    <servlet-class>
        org.apache.catalina.servlets.InvokerServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
```

用途：为 web 应用写一个 InitServlet，这个 servlet 配置为启动时装载，为整个 web 应用创建必要的数据库表和数据。

### 3.4 缺省Servlet

如果某个 Servlet 的映射路径仅仅为一个正斜杠（/），那么这个 Servlet 就成为当前 Web 应用程序的缺省 Servlet。
凡是在 web.xml 文件中找不到匹配的 <servlet-mapping> 元素的 URL，它们的访问请求都将交给缺省 Servlet 处理，也就是说，缺省 Servlet 用于处理所有其他 Servlet 都不处理的访问请求。 例如：

```xml
<servlet>
    <servlet-name>ServletDemo2</servlet-name>
    <servlet-class>gacl.servlet.study.ServletDemo2</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <!-- 将ServletDemo2配置成缺省Servlet -->
  <servlet-mapping>
    <servlet-name>ServletDemo2</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

### 3.5 ServletConfig

**1）配置初始化参数**

在web.xml中，可以使用一个或多个`<init-param>`标签为servlet配置一些初始参数。

```xml
<servlet>
    <servlet-name>ServletConfigDemo1</servlet-name>
    <servlet-class>gacl.servlet.study.ServletConfigDemo1</servlet-class>
    <!--配置ServletConfigDemo1的初始化参数 -->
    <init-param>
        <param-name>name</param-name>
        <param-value>gacl</param-value>
    </init-param>
     <init-param>
        <param-name>password</param-name>
        <param-value>123</param-value>
    </init-param>
    <init-param>
        <param-name>charset</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</servlet>
```

**2）通过ServletConfig获取Servlet的初始化参数**

当servlet配置了初始化参数后，web容器在创建servlet实例对象时，会自动将这些初始化参数封装到ServletConfig对象中，并在调用servlet的init方法时，将ServletConfig对象传递给servlet。

```java
package gacl.servlet.study;

import java.io.IOException;
import java.util.Enumeration;
import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ServletConfigDemo1 extends HttpServlet {

    /**
     * 定义ServletConfig对象来接收配置的初始化参数
     */
    private ServletConfig config;

    /**
     * 当servlet配置了初始化参数后，web容器在创建servlet实例对象时，
     * 会自动将这些初始化参数封装到ServletConfig对象中，并在调用servlet的init方法时，
     * 将ServletConfig对象传递给servlet。进而，程序员通过ServletConfig对象就可以
     * 得到当前servlet的初始化参数信息。
     */
    @Override
    public void init(ServletConfig config) throws ServletException {
        this.config = config;
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        //获取在web.xml中配置的初始化参数
        String paramVal = this.config.getInitParameter("name");//获取指定的初始化参数
        response.getWriter().print(paramVal);

        response.getWriter().print("<hr/>");
        //获取所有的初始化参数
        Enumeration<String> e = config.getInitParameterNames();
        while(e.hasMoreElements()){
            String name = e.nextElement();
            String value = config.getInitParameter(name);
            response.getWriter().print(name + "=" + value + "<br/>");
        }
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        this.doGet(request, response);
    }

}
```

### 3.6 ServletContext对象

WEB容器在启动时，它会**为每个WEB应用程序都创建一个对应的ServletContext对象，它代表当前web应用**。
ServletConfig对象中维护了ServletContext对象的引用，开发人员在编写servlet时，可以通过`ServletConfig.getServletContext`方法获得ServletContext对象。
由于**一个WEB应用中的所有Servlet共享同一个ServletContext对象**，因此Servlet对象之间可以通过ServletContext对象来实现通讯。ServletContext对象通常也被称之为context域对象。

范例：ServletContextDemo1和ServletContextDemo2通过ServletContext对象实现数据共享。

```java
package gacl.servlet.study;

import java.io.IOException;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ServletContextDemo1 extends HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String data = "xdp_gacl";
        /**
         * ServletConfig对象中维护了ServletContext对象的引用，开发人员在编写servlet时，
         * 可以通过ServletConfig.getServletContext方法获得ServletContext对象。
         */
        ServletContext context = this.getServletConfig().getServletContext();//获得ServletContext对象
        context.setAttribute("data", data);  //将data存储到ServletContext对象中
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }
}
```

```java
package gacl.servlet.study;

import java.io.IOException;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ServletContextDemo2 extends HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        String data = (String) context.getAttribute("data");//从ServletContext对象中取出数据
        response.getWriter().print("data="+data);
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }
}
```

**1）获取web应用初始化参数**

在web.xml文件中使用<context-param>标签配置WEB应用的初始化参数，如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
    http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
    <display-name></display-name>
    <!-- 配置WEB应用的初始化参数 -->
    <context-param>
        <param-name>url</param-name>
        <param-value>jdbc:mysql://localhost:3306/test</param-value>
    </context-param>

    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

获取web应用初始化参数，

```java
package gacl.servlet.study;

import java.io.IOException;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;


public class ServletContextDemo3 extends HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        ServletContext context = this.getServletContext();
        //获取整个web站点的初始化参数
        String contextInitParam = context.getInitParameter("url");
        response.getWriter().print(contextInitParam);
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }

}
```

