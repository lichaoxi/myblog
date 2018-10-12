---
title: 简单说说HTTP协议
tags: [HTTP协议,网络]
categories: 网络
copyright: true
---
#### 一、Http协议
HTTP协议（HyperText Transfer Protocol，超文本传输协议）是用于从3W服务器传输超文本到本地浏览器的传送协议。它可以使浏览器更加高效，使网络传输减少。它不仅保证计算机正确快速地传输超文本文档，还确定传输文档中的哪一部分，以及哪部分内容首先显示(如文本先于图形)等。
<!--more-->
HTTP是一个应用层协议，基于请求与响应模式的、无状态的、应用层的协议，常基于TCP的连接方式。HTTP使用统一资源标识符（Uniform Resource Identifiers, URI）来传输数据和建立连接。URL是一种特殊类型的URI，包含了用于查找某个资源的足够的信息URL，全称是`UniformResourceLocator`, 中文叫统一资源定位符,是互联网上用来标识某一处资源的地址。
![请求响应模型](http://upload-images.jianshu.io/upload_images/8926909-55c27c7d4394a5a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
HTTP协议定义Web客户端如何从Web服务器请求Web页面，以及服务器如何把Web页面传送给客户端。HTTP协议采用了请求/响应模型。客户端向服务器发送一个请求报文，请求报文包含请求的方法、URL、协议版本、请求头部和请求数据。服务器以一个状态行作为响应，响应的内容包括协议的版本、成功或者错误代码、服务器信息、响应头部和响应数据。以下是 HTTP 请求/响应的步骤：

###### 1、客户端连接到Web服务器

一个HTTP客户端，通常是浏览器，与Web服务器的HTTP端口（默认为80）建立一个TCP套接字连接。例如，[https://www.jianshu.com/yitaicloud](https://www.jianshu.com/u/abcf0e8f851a)。

###### 2、发送HTTP请求

通过TCP套接字，客户端向Web服务器发送一个文本的请求报文，一个请求报文由请求行、请求头部、空行和请求数据4部分组成。

###### 3、服务器接受请求并返回HTTP响应

Web服务器解析请求，定位请求资源。服务器将资源复本写到TCP套接字，由客户端读取。一个响应由状态行、响应头部、空行和响应数据4部分组成。

###### 4、释放连接[TCP连接](http://www.jianshu.com/p/ef892323e68f)

若connection 模式为close，则服务器主动关闭[TCP连接](http://www.jianshu.com/p/ef892323e68f)，客户端被动关闭连接，释放[TCP连接](http://www.jianshu.com/p/ef892323e68f);若connection 模式为keepalive，则该连接会保持一段时间，在该时间内可以继续接收请求;

###### 5、客户端浏览器解析HTML内容

客户端浏览器首先解析状态行，查看表明请求是否成功的状态代码。然后解析每一个响应头，响应头告知以下为若干字节的HTML文档和文档的字符集。客户端浏览器读取响应数据HTML，根据HTML的语法对其进行格式化，并在浏览器窗口中显示。

例如：在浏览器地址栏键入URL，按下回车之后会经历以下流程：
1、浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址;
2、解析出 IP 地址后，根据该 IP 地址和默认端口 80，和服务器建立[TCP连接](http://www.jianshu.com/p/ef892323e68f);
3、浏览器发出读取文件(URL 中域名后面部分对应的文件)的HTTP 请求，该请求报文作为 [TCP 三次握手](http://www.jianshu.com/p/ef892323e68f)的第三个报文的数据发送给服务器;
4、服务器对浏览器请求作出响应，并把对应的 html 文本发送给浏览器;
5、释放 [TCP连接](http://www.jianshu.com/p/ef892323e68f);
6、浏览器将该 html 文本并显示内容;
>[《关于HTTP协议，一篇就够了》](http://www.cnblogs.com/ranyonsue/p/5984001.html)

[如果TCP有点生疏了，请戳下这里](http://blog.csdn.net/zhuolou1208/article/details/78106420)

![Http协议在TCP/IP协议栈中位置](http://upload-images.jianshu.io/upload_images/8926909-9f9e6a0cb61243cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 1.常用的HTTP方法有哪些？
GET： 用于请求访问已经被URI（统一资源标识符）识别的资源，可以通过URL传参给服务器。
POST：用于传输信息给服务器，主要功能与GET方法类似，但一般推荐使用POST方式。
PUT： 传输文件，报文主体中包含文件内容，保存到对应URI位置。
DELETE：删除文件，与PUT方法相反，删除对应URI位置的文件。
HEAD： 获得报文首部，与GET方法类似，只是不返回报文主体，一般用于验证URI是否有效。
OPTIONS：查询相应URI支持的HTTP方法。

##### 2.GET方法与POST方法的区别
**区别一：**
get重点在从服务器上获取资源，post重点在向服务器发送数据；
**区别二：**
- get传输数据是通过URL请求，以field（字段）= value的形式，置于URL后，并用"?"连接，多个请求数据间用"&"连接，如http://127.0.0.1/Test/login.action?name=admin&password=admin，这个过程用户是可见的；
- post传输数据通过Http的post机制，将字段与对应值封存在请求实体中发送给服务器，这个过程对用户是不可见的；

**区别三：**
Get传输的数据量小，因为受URL长度限制，但效率较高；
Post可以传输大量数据，所以上传文件时只能用Post方式；
**区别四：**
get是不安全的，因为URL是可见的，可能会泄露私密信息，如密码等(页面可以被缓存，或者获得历史访问记录)；
post较get安全性较高；
**区别五：**
get方式只能支持ASCII字符，向服务器传的中文字符可能会乱码。
post支持标准字符集，可以正确传递中文字符。

##### 3.HTTP请求报文与响应报文格式
请求报文包含三部分：
a、请求行：包含请求方法、URI、HTTP版本信息
b、请求首部字段
c、请求内容实体
响应报文包含三部分：
a、状态行：包含HTTP版本、状态码、状态码的原因短语
b、响应首部字段
c、响应内容实体

>**返回的状态**
1xx：指示信息--表示请求已接收，继续处理
2xx：成功--表示请求已被成功接收、理解、接受
3xx：重定向--要完成请求必须进行更进一步的操作
4xx：客户端错误--请求有语法错误或请求无法实现
5xx：服务器端错误--服务器未能实现合法的请求

>**200：请求被正常处理**
204：请求被受理但没有资源可以返回
206：客户端只是请求资源的一部分，服务器只对请求的部分资源执行GET方法，相应报文中通过Content-Range指定范围的资源。
301：永久性重定向
**302：临时重定向**
303：与302状态码有相似功能，只是它希望客户端在请求一个URI的时候，能通过GET方法重定向到另一个URI上
304：发送附带条件的请求时，条件不满足时返回，与重定向无关
307：临时重定向，与302类似，只是强制要求使用POST方法
400：请求报文语法有误，服务器无法识别
401：请求需要认证
403：请求的对应资源禁止被访问
**404：服务器无法找到对应资源**
>**500：服务器内部错误**
>503：服务器正忙

##### 4.常见HTTP首部字段
![Http请求截图](http://upload-images.jianshu.io/upload_images/8926909-b5c977f5065f41a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

a、通用首部字段（请求报文与响应报文都会使用的首部字段）
Date：创建报文时间
Connection：连接的管理
Cache-Control：缓存的控制
Transfer-Encoding：报文主体的传输编码方式

b、请求首部字段（请求报文会使用的首部字段）
Host：请求资源所在服务器
Accept：可处理的媒体类型
Accept-Charset：可接收的字符集
Accept-Encoding：可接受的内容编码
Accept-Language：可接受的自然语言

c、响应首部字段（响应报文会使用的首部字段）
Accept-Ranges：可接受的字节范围
Location：令客户端重新定向到的URI
Server：HTTP服务器的安装信息

d、实体首部字段（请求报文与响应报文的的实体部分使用的首部字段）
Allow：资源可支持的HTTP方法
Content-Type：实体主类的类型
Content-Encoding：实体主体适用的编码方式
Content-Language：实体主体的自然语言
Content-Length：实体主体的的字节数
Content-Range：实体主体的位置范围，一般用于发出部分请求时使用

##### 5.HTTP1.1版本新特性
先上答案，然后再后面详细解释：
a、默认持久连接，节省通信量，只要客户端服务端任意一端没有明确提出断开TCP连接，就一直保持连接，可以发送多次HTTP请求
b、管线化，客户端可以同时发出多个HTTP请求，而不用一个个等待响应
c、断点续传原理

HTTP 1.1支持持久连接，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟。一个包含有许多图像的网页文件的多个请求和应答可以在一个连接中传输，但每个单独的网页文件的请求和应答仍然需要使用各自的连接。HTTP 1.1还允许客户端不用等待上一次请求结果返回，就可以发出下一次请求，但服务器端必须按照接收到客户端请求的先后顺序依次回送响应结果，以保证客户端能够区分出每次请求的响应内容，这样也显著地减少了整个下载过程所需要的时间。基于HTTP 1.1协议的客户机与服务器的信息交换过程。
![Http1.1工作](http://upload-images.jianshu.io/upload_images/8926909-cfd0517c7223404b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>[【HTTP 1.1与HTTP 1.0的比较HTTP 1.1与HTTP 1.0的比较】](http://blog.csdn.net/elifefly/article/details/3964766/)

##### 6.HTTP的缺点与HTTPS
a、通信使用明文不加密，内容可能被窃听
b、不验证通信方身份，可能遭到伪装
c、无法验证报文完整性，可能被篡改
HTTPS就是HTTP加上加密处理（一般是SSL安全通信线路）+认证+完整性保护

##### 7.HTTP优化
利用负载均衡优化和加速HTTP应用
利用HTTP Cache来优化网站

补充一点**Cache**的细节：
缓存是一个到处都存在的用空间换时间的例子。通过使用多余的空间，我们能够获取更快的速度。用户在浏览网站的时候，浏览器能够在本地保存网站中的图片或者其他文件的副本，这样用户再次访问该网站的时候，浏览器就不用再下载全部的文件，减少了下载量意味着提高了页面加载的速度。
缓存非常有用，但是也带来了一定的缺陷。当我们的网站发生了更新的时候，比如说Logo换了，浏览器本地仍保存着旧版本的Logo，那么浏览器如何来确定使用本地文件还是使用服务器上的新文件？下面来介绍几种判断的方法。
- Caching Method 1：Last-Modified
服务器为了通知浏览器当前文件的版本，会发送一个上次修改时间的标签，例如：
Last-modified: Fri, 16 Mar 2007 04:00:25 GMT
- Caching Method 2: ETag
通常情况下，通过修改时间来比较文件是可行的。但是在一些特殊情况，例如服务器的时钟发生了错误，服务器时钟进行修改，夏时制DST到来后服务器时间没有及时更新，这些都会引起通过修改时间比较文件版本的问题。
ETag可以用来解决这种问题。ETag是一个文件的唯一标志符。就像一个哈希或者指纹，每个文件都有一个单独的标志，只要这个文件发生了改变，这个标志就会发生变化。如同 Last-modified 一样，ETag 解决了文件版本比较的问题。只不过 ETag 的级别比 Last-Modified 高一些。
- Caching Method 3：Expires
缓存一个文件，并且与服务器确认版本的方式非常好，但是仍有一个缺点，我们必须连接服务器。每次使用前都进行一次比较，这种方法很安全，但还不是最好的。我们可以使用 Expiration Date 来减少这种请求。
- Caching Method 4：Max-age
Expires的方法很好，但是我们每次都得算一个精确的时间。max-age 标签可以让我们更加容易的处理过期时间。我们只需要说，这份资料你只能用一个星期就可以了。
>[【利用HTTP Cache来优化网站】](http://www.cnblogs.com/cocowool/archive/2011/08/22/2149929.html)

##### 8.跨域问题
浏览器的同源策略会导致跨域，这里同源策略又分为以下两种
- DOM同源策略：禁止对不同源页面DOM进行操作。这里主要场景是iframe跨域的情况，不同域名的iframe是限制互相访问的。
- XmlHttpRequest同源策略：禁止使用XHR对象向不同源的服务器地址发起HTTP请求。

只要协议、域名、端口有任何一个不同，都被当作是不同的域，之间的请求就是跨域操作。
要解决跨域问题可以参考以下文章：
>[《跨域的那些事儿》](https://zhuanlan.zhihu.com/p/28562290)
[《跨域资源共享 CORS 详解》](http://www.ruanyifeng.com/blog/2016/04/cors.html) 阮一峰大神的精品

另外玉刚写了一篇不错的，角度是站在前端来看的
[《HTTP 必知必会的那些》](https://mp.weixin.qq.com/s?src=11&timestamp=1534170599&ver=1058&signature=NRB3iKd0Se5un2PbMTA2IjwRKl56ROP2rapEPAUXnW*zwI0itn4bonQ4ieM*-qxl0nLLyYQYkCkMrcpB5Bl1j8ujt6spFS3BEPfdIXVZvx5o6LXKHjDi74Zu6Mn2xsvB&new=1)