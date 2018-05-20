---
title: Web服务器及Tomcat总结
tags: [Interview,java web,Tomcat]
categories: Tomcat
copyright: true
---
#### 一、Web服务
##### 1.web服务器分类
Unix和Linux平台下使用最广泛的免费HTTP服务器是Apache服务器，而Windows平台的服务器通常使用IIS作为Web服务器。选择Web服务器应考虑的因素有：性能、安全性、日志和统计、虚拟主机、代理服务器、缓冲服务和集成应用程序等。<!--more-->下面是对常见服务器的简介： 
- IIS：Microsoft的Web服务器产品，全称是Internet Information Services。IIS是允许在公共Intranet或Internet上发布信息的Web服务器。IIS是目前最流行的Web服务器产品之一，很多著名的网站都是建立在IIS的平台上。IIS提供了一个图形界面的管理工具，称为Internet服务管理器，可用于监视配置和控制Internet服务。IIS是一种Web服务组件，其中包括Web服务器、FTP服务器、NNTP服务器和SMTP服务器，分别用于网页浏览、文件传输、新闻服务和邮件发送等方面，它使得在网络（包括互联网和局域网）上发布信息成了一件很容易的事。它提供ISAPI(Intranet Server API）作为扩展Web服务器功能的编程接口；同时，它还提供一个Internet数据库连接器，可以实现对数据库的查询和更新。 

- Kangle：Kangle Web服务器是一款跨平台、功能强大、安全稳定、易操作的高性能Web服务器和反向代理服务器软件。此外，Kangle也是一款专为做虚拟主机研发的Web服务器。实现虚拟主机独立进程、独立身份运行。用户之间安全隔离，一个用户出问题不影响其他用户。支持PHP、ASP、ASP.NET、Java、Ruby等多种动态开发语言。 
- WebSphere：WebSphere Application Server是功能完善、开放的Web应用程序服务器，是IBM电子商务计划的核心部分，它是基于Java的应用环境，用于建立、部署和管理Internet和Intranet Web应用程序，适应各种Web应用程序服务器的需要。 
- WebLogic：WebLogic Server是一款多功能、基于标准的Web应用服务器，为企业构建企业应用提供了坚实的基础。针对各种应用开发、关键性任务的部署，各种系统和数据库的集成、跨Internet协作等Weblogic都提供了相应的支持。由于它具有全面的功能、对开放标准的遵从性、多层架构、支持基于组件的开发等优势，很多公司的企业级应用都选择它来作为开发和部署的环境。WebLogic Server在使应用服务器成为企业应用架构的基础方面一直处于领先地位，为构建集成化的企业级应用提供了稳固的基础。 
- Apache：目前Apache仍然是世界上用得最多的Web服务器，其市场占有率很长时间都保持在60%以上（目前的市场份额约40%左右）。世界上很多著名的网站都是Apache的产物，它的成功之处主要在于它的源代码开放、有一支强大的开发团队、支持跨平台的应用（可以运行在几乎所有的Unix、Windows、Linux系统平台上）以及它的可移植性等方面。 
- Tomcat：Tomcat是一个开放源代码、运行Servlet和JSP的容器。Tomcat实现了Servlet和JSP规范。此外，Tomcat还实现了Apache-Jakarta规范而且比绝大多数商业应用软件服务器要好，因此目前也有不少的Web服务器都选择了Tomcat。 
- Nginx：读作"engine x"，是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP代理服务器。 Nginx是由Igor Sysoev为俄罗斯访问量第二的[Rambler](http://www.rambler.ru/)站点开发的，第一个公开版本0.1.0发布于2004年10月4日。其将源代码以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名。在2014年下半年，Nginx的市场份额达到了14%。

##### 2、web.xml文件中可以配置哪些内容？ 
答：web.xml用于配置Web应用的相关信息，如：监听器（listener）、过滤器（filter）、 Servlet、相关参数、会话超时时间、安全验证方式、错误页面等，下面是一些开发中常见的配置：
① 配置Spring上下文加载监听器加载Spring配置文件并创建IoC容器：
```xml
  <context-param>
     <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>

  <listener>
     <listener-class>
       org.springframework.web.context.ContextLoaderListener
     </listener-class>
  </listener>
```
② 配置Spring的`OpenSessionInView`过滤器来解决延迟加载和Hibernate会话关闭的矛盾：
```xml
  <filter>
      <filter-name>openSessionInView</filter-name>
      <filter-class>
         org.springframework.orm.hibernate3.support.OpenSessionInViewFilter
      </filter-class>
  </filter>

  <filter-mapping>
      <filter-name>openSessionInView</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
```
③ 配置会话超时时间为10分钟：
```xml
  <session-config>
      <session-timeout>10</session-timeout>
  </session-config>
```
④ 配置404和Exception的错误页面：
```xml
  <error-page>
      <error-code>404</error-code>
      <location>/error.jsp</location>
  </error-page>

  <error-page>
      <exception-type>java.lang.Exception</exception-type>
      <location>/error.jsp</location>
  </error-page>
```
⑤ 配置安全认证方式：
```xml
  <security-constraint>
      <web-resource-collection>
          <web-resource-name>ProtectedArea</web-resource-name>
          <url-pattern>/admin/*</url-pattern>
          <http-method>GET</http-method>
          <http-method>POST</http-method>
      </web-resource-collection>
      <auth-constraint>
          <role-name>admin</role-name>
      </auth-constraint>
  </security-constraint>

  <login-config>
      <auth-method>BASIC</auth-method>
  </login-config>

  <security-role>
      <role-name>admin</role-name>
  </security-role>
```
说明：对Servlet（小服务）、Listener（监听器）和Filter（过滤器）等Web组件的配置，Servlet 3规范提供了基于注解的配置方式，可以分别使用@WebServlet、@WebListener、@WebFilter注解进行配置。 

补充：如果Web提供了有价值的商业信息或者是敏感数据，那么站点的安全性就是必须考虑的问题。安全认证是实现安全性的重要手段，认证就是要解决“Are you who you say you are?”的问题。认证的方式非常多，简单说来可以分为三类： 
A. What you know? — 口令 
B. What you have? — 数字证书（U盾、密保卡） 
C. Who you are? — 指纹识别、虹膜识别 
在Tomcat中可以通过建立安全套接字层（Secure Socket Layer, SSL）以及通过基本验证或表单验证来实现对安全性的支持。

#### 二、Tomcat总结
##### 1.Tomcat架构
![Tomcat架构](http://upload-images.jianshu.io/upload_images/8926909-6b8166af025c93d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Tomcat中最顶层的容器是Server，代表着整个服务器，从上图中可以看出，一个Server可以包含至少一个Service，用于具体提供服务。
Service主要包含两个部分：`Connector`和`Container`。从上图中可以看出 Tomcat 的心脏就是这两个组件，他们的作用如下：
- Connector用于处理连接相关的事情，并提供Socket与Request和Response相关的转化; 
- Container用于封装和管理Servlet，以及具体处理Request请求；

一个Tomcat中只有一个Server，一个Server可以包含多个Service，一个Service只有一个Container，但是可以有多个Connectors，这是因为一个服务可以有多个连接，如同时提供Http和Https链接，也可以提供向相同协议不同端口的连接,示意图如下
![Tomcat组成](http://upload-images.jianshu.io/upload_images/8926909-efed3d1d9d12ac59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**小结：**
（1）Tomcat中只有一个Server，一个Server可以有多个Service，一个Service可以有多个Connector和一个Container； 
（2）Server掌管着整个Tomcat的生死大权； 
（4）Service 是对外提供服务的； 
（5）Connector用于接受请求并将请求封装成Request和Response来具体处理； 
（6）Container用于封装和管理Servlet，以及具体处理request请求；

**Connector和Container的微妙关系**
由上述内容我们大致可以知道一个请求发送到Tomcat之后，首先经过Service然后会交给我们的Connector，Connector用于接收请求并将接收的请求封装为Request和Response来具体处理，Request和Response封装完之后再交由Container进行处理，Container处理完请求之后再返回给Connector，最后在由Connector通过Socket将处理的结果返回给客户端，这样整个请求的就处理完了！

Connector最底层使用的是Socket来进行连接的，Request和Response是按照HTTP协议来封装的，所以Connector同时需要实现TCP/IP协议和HTTP协议！
>[《四张图带你了解Tomcat系统架构--让面试官颤抖的Tomcat回答系列！》](http://blog.csdn.net/xlgen157387/article/details/79006434)

##### 2.解释什么是Jasper?
Jasper是Tomcat的JSP引擎，它解析JSP文件，将它们编译成JAVA代码作为servlet。在运行时，Jasper允许自动检测JSP文件的更改并重新编译它们

##### 3.请解释Tomcat的默认端口是什么?
Tomcat的默认端口是8080。在本地机器上初始化Tomcat之后，可以验证Tomcat是否正在运行URL:http://localhost:8080

##### 4.请解释Tomcat中使用的连接器是什么?
在Tomcat中，使用了两种类型的连接器：
HTTP连接器:它有许多可以更改的属性，以确定它的工作方式和访问功能，如重定向和代理转发
AJP连接器:它以与HTTP连接器相同的方式工作，但是他们使用的是HTTP的AJP协议。AJP连接器通常通过插件技术mod_jk在Tomcat中实现

##### 5.解释如何使用WAR文件部署web应用程序?
在Tomcat的web应用程序目录下，jsp、servlet和它们的支持文件被放置在适当的子目录中。你可以将web应用程序目录下的所有文件压缩到一个压缩文件中，以.war文件扩展名结束。你可以通过在webapps目录中放置WAR文件来执行web应用程序。当一个web服务器开始执行时，它会将WAR文件的内容提取到适当的webapps子目录中。

##### 6.解释什么是Tomcat Valve?说明Tomcat配置了多少个Valve?
Tomcat Valve——Tomcat 4引入的新技术，它允许您将Java类的实例链接到一个特定的Catalina容器。Tomcat配置了四种类型的Valve：
- 访问日志
- 远程地址过滤
- 远程主机过滤器
- 客户请求记录器

##### 7.解释servlet如何完成生命周期?
在Tomcat上运行的典型servlet生命周期如下：
- Tomcat通过它的其中一个连接器接收来自客户端的请求
- 进程请求Tomcat将此请求映射为适当的Servlet
- 一旦请求被定向到适当的servlet，Tomcat就会验证servlet类是否已经加载。如果不是,Tomcat将servlet包装成Java字节码，这是由JVM执行的，并形成servlet的实例
- Tomcat通过调用它的init来启动servlet，它包含能够筛选Tomcat配置文件并相应地采取行动的代码，并声明它可能需要的任何资源
- 一旦servlet启动，Tomcat就可以调用servlet的服务方法来进行请求
- 在servlet的生命周期中，Tomcat和servlet可以通过使用侦听器类来进行协调或通信，从而跟踪各种状态变化的servlet
- 删除servlet，Tomcat调用servlet销毁方法

##### 8.Tomcat优化经验
一、关掉对web.xml的监视，把jsp提前编辑成Servlet。有富余物理内存的情况，加大tomcat使用的jvm的内存

二、服务器资源。服务器所能提供CPU、内存、硬盘的性能对处理能力有决定性影响。
(1) 对于高并发情况下会有大量的运算，那么CPU的速度会直接影响到处理速度。
(2) 内存在大量数据处理的情况下，将会有较大的内存容量需求，可以用-Xmx -Xms -XX:MaxPermSize等参数对内存不同功能块进行划分。我们之前就遇到过内存分配不足，导致虚拟机一直处于full GC，从而导致处理能力严重下降。
(3) 硬盘主要问题就是读写性能，当大量文件进行读写时，磁盘极容易成为性能瓶颈。最好的办法还是利用下面提到的缓存。

三、利用缓存和压缩
对于静态页面最好是能够缓存起来，这样就不必每次从磁盘上读。这里我们采用了Nginx作为缓存服务器，将图片、css、js文件都进行了缓存，有效的减少了后端tomcat的访问。另外，为了能加快网络传输速度，开启gzip压缩也是必不可少的。但考虑到tomcat已经需要处理很多东西了，所以把这个压缩的工作就交给前端的Nginx来完成。

除了文本可以用gzip压缩，其实很多图片也可以用图像处理工具预先进行压缩，找到一个平衡点可以让画质损失很小而文件可以减小很多。曾经我就见过一个图片从300多kb压缩到几十kb，自己几乎看不出来区别。

 四、采用集群
单个服务器性能总是有限的，最好的办法自然是实现横向扩展，那么组建tomcat集群是有效提升性能的手段。我们还是采用了Nginx来作为请求分流的服务器，后端多个tomcat共享session来协同工作。可以参考之前写的《利用nginx+tomcat+memcached组建web服务器负载均衡》。

五、 优化tomcat参数
这里以tomcat7的参数配置为例，需要修改conf/server.xml文件，主要是优化连接配置，关闭客户端dns查询。
```xml
<Connector port="8080"   
           protocol="org.apache.coyote.http11.Http11NioProtocol"  
           connectionTimeout="20000"  
           redirectPort="8443"   
           maxThreads="500"   
           minSpareThreads="20"  
           acceptCount="100" 
           disableUploadTimeout="true" 
           enableLookups="false"   
           URIEncoding="UTF-8" /> 
```

