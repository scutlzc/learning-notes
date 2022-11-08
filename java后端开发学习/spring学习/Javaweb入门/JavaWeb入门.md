# JavaWeb入门

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

