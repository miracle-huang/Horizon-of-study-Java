# HTTP概述
HTTP 超文本传输协议 Hyper Text Transfer Protocol
当我们在浏览器的地址栏输入一个地址的时候，就能够访问服务器的某个页面。
这个过程本身就是两个应用程序之间的交互，一个应用程序是浏览器，另一个应用程序是服务器。
协议是什么？ 协议就是不同的应用程序之间按照事先做好的约定进行的通信。 这样就能互相读懂对方的意思。
浏览器和WEB服务器之间，使用的就是一种叫做HTTP的协议。 这样是BS (Browser Server )架构模型的基础。
也可以理解为资源在网络中的传输方式。
### 屏蔽底层
http是一个运行在**应用层**的上层协议。
我们在**屏蔽底层的情况下**， 可以认为两台计算机是依据http协议交互的。
![屏蔽底层进行交互](https://upload-images.jianshu.io/upload_images/21824483-d61f4cca6548a754.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 从浏览器角度进行分析
1. 分析出url, 分离请求的地址, 端口 服务器内部路径
2. 分析出请求方式
3. 分析出请求的参数 (get, post)
4. 分析状态码 
5. 能分析返回值

# 工作流程
**页面产生的基本逻辑**
1. 域名解析
2. 发起tcp三次握手
3. 发起http请求
4. 响应http请求(html相关的代码)
5. 解析html代码
6. 加载静态资源

![一次完整的HTTP事务过程](https://upload-images.jianshu.io/upload_images/21824483-fe043e36ea4368f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 域名解析
![域名解析图解](https://upload-images.jianshu.io/upload_images/21824483-85aadc48aca426cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**域名解析步骤**
1. 浏览器, 检查自身缓存, 
2. 检查操作系统的缓存
3. 检查hosts文件, 看是否配置的有对应IP地址
4. 请求域名解析服务: 递归查询, 迭代查询
![域名解析过程](https://upload-images.jianshu.io/upload_images/21824483-cf30407dbf0dc4d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### http请求
![http请求的发生地点](https://upload-images.jianshu.io/upload_images/21824483-8024ef71e84bf097.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
http这个协议的本质就是把一个网络请求在应用层转化为**报文**格式
###### 请求行
```
请求方式     url                               协议
GET          http://www.cskaoyan.com/forum.php HTTP/1.1
```
请求行用于描述客户端的请求方式、请求的资源名称，以及使用的HTTP协议版本号。

###### 请求方式
语义化的区别
Get	   :  获取数据, 参数一般url之后(?分割, &), 1kb, 安全性, 
Post: 提交数据,   正文里,  没限制,  不可见
Put    : 传输文件
Delete  : 删除操作
Options:  测试连接(询问后台是否支持某个方法, url, 接口)
Head:   获取报文首部

###### 请求头部 
请求头部信息提供了如下信息:
Host: 主机名
User-Agent: 浏览器基本资料
Accept: 浏览器能够识别的响应类型
Accept-Language: 浏览器默认语言
Accept-Encoding: 浏览器能够识别的压缩方式
Referer: 来路页面， /addHero 这个路径是通过addHero.html这个页面跳转过来的。
Connecton：是否保持连接

这些信息，也可以在HeroAddServlet中，通过 [request对象获取](https://how2j.cn/k/servlet/servlet-request/555.html#step1609)

常用请求头：
- Accept:浏览器可接受的    MIME类型 */*   (大类型)/(小类型)
- Accept-Charset: 浏览器通过这个头告诉服务器，它支持哪种字符集
- Accept-Encoding:浏览器能够进行解码的数据编码方式，比如gzip 
- Accept-Language: 浏览器所希望的语言种类，当服务器能够提供一种以上的语言版本时要用到。 可以在浏览器中进行设置。
- Host:初始URL中的主机和端口 
域名--ip
Aaa
Bbb
- Referer:包含一个URL，用户从该URL代表的页面出发访问当前请求的页面 （防盗链）
qq空间, 别的网站 (qq空间内部图片, 外部不可访问)

Content-Type:内容类型
- If-Modified-Since: Wed, 02 Feb 2011 12:04:56 GMT 服务器利用这个头与服务器的文件进行比对，如果一致，则告诉浏览器从缓存中直接读取文件。
- User-Agent:浏览器类型.
- Content-Length:表示请求消息正文的长度 
- Connection:表示是否需要持久连接。如果服务器看到这里的值为“Keep -Alive”，或者看到请求使用的是HTTP 1.1（HTTP 1.1默认进行持久连接
http 1.0 1.1 是否保持长连接,持久连接 
- Cookie:这是最重要的请求头信息之一 
- Date：Date: Mon, 22 Aug 2011 01:55:39 GMT请求时间GMT

### http响应
一个HTTP响应代表服务器向客户端回送的数据。
一个完整的HTTP响应包括如下内容：
- 一个状态行
- 若干消息头
- 空格及响应正文

其中的一些消息头和正文都是可选的，消息头和正文内容之间要用空行隔开。
###### 响应状态行
```
协议        状态码   原因短语(没有任何意义, 可选) 
HTTP/1.1    200        OK
```
###### 状态码
>200 (正常)
表示一切正常，返回的是正常请求结果 
206 表示分段的请求OK
301、302/307 (临时重定向)
指出被请求的文档已被临时移动到别处，此文档的新的URL在Location响应头中给出。
304(未修改)
表示客户机缓存的版本是最新的，客户机可以继续使用它，无需到服务器请求。
404(找不到)
>404 服务器上不存在客户机所请求的资源。
400  服务器不支持这种请求方式
500 (服务器内部错误)
服务器端的程序发生错误
###### 响应消息头
- Location: [<u>http://www.cskaoyan.com/</u>](http://www.cskaoyan.com/指示新的资源的位置)[<u>指示新的资源的位置</u>](http://www.cskaoyan.com/指示新的资源的位置) 
- Server: apache tomcat 指示服务器的类型
- Content-Encoding: gzip 服务器发送的数据采用的编码类型
- Content-Length: 80 告诉浏览器正文的长度
- Content-Language: zh-cn服务发送的文本的语言
- Content-Type: text/html;  服务器发送的内容的MIME类型
- Refresh: 1;url=http://www.cskaoyan.com指示客户端刷新频率。单位是秒
- Set-Cookie: SS=Q0=5Lb_nQ; path=/search服务器端发送的Cookie
- **Expires: 0**关于浏览器缓存
- **Cache-Control**: no-cache (1.1)   关于浏览器缓存
- Connection: close/Keep-Alive   
- Date: Tue, 11 Jul 2000 18:23:51 GMT
### 解析html代码
浏览器拿到HTML文档后，开始解析HTML代码(dom解析)。
当遇到JS/CSS/图片等静态资源时，会自动向服务器端再(再发http请求)请求下载。
![下载静态资源](https://upload-images.jianshu.io/upload_images/21824483-5e5d186a61de6cdb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 加载静态资源
最后一步，浏览器利用自己内部的工作机制，把请求到的HTML代码和静态资源进行渲染，渲染最后，呈现给用户。
# http和https
[HTTP1.0、HTTP1.1 和 HTTP2.0 的区别](https://www.cnblogs.com/heluan/p/8620312.html)
### https
>https是一个加密传输的http协议
证书: 是由权威机构办法的证书 (包含很多内容)
>加密: 混合加密, 对称加密+非对称加密
完整性保障: 中间人攻击

>对称加密:   加密和解密是同一种方式, 知道加密方式,就知道解密方式
非对称加密:  单向过程,  
数据经过一种加密方式加密之后, 要通过别的途径解密
			公钥加密  私钥解密

![两种加密方式的比较](https://upload-images.jianshu.io/upload_images/21824483-8adf46a21512d202.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





