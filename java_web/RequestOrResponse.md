### Request Response是什么？
当Web容器收到客户端的发送过来http请求，会针对每一次请求，分别创建一个用于代表此次请求的HttpServletRequest对象(request)对象、和代表响应的HTTPServletResponse对象(response)。
request负责获取客户机提交过来的数据，response负责向客户机输出数据。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/20161206100434669.png)<br>


### ----------------------------------------Request对象------------------------------------------------
##### 常用方法
request.getMethod() 请求方式 <br>
request.getContextPath()，getServletPath()…获取请求路径<br>
request.getScheme() 获取请求协议<br>
request.getRequestURL方法返回客户端发出请求时的完整URL<br>
request.getRequestURI方法返回请求行中的资源名部分<br>
request.getQueryString 方法返回请求行中的参数部分<br>
request.getRemoteAddr方法返回发出请求的客户机的IP地址<br>
request.getRemoteHost方法返回发出请求的客户机的完整主机名<br>
request.getRemotePort方法返回客户机所使用的网络端口号<br>
request.getLocalAddr方法返回WEB服务器的IP地址<br>
request.getLocalName方法返回WEB服务器的主机名<br>

###### 防盗链
防盗链：
有些下载系统的下载地址会有一个下载跳转，用户在下载页面上看到的下载地址可能是 http://www.abc.com/down.asp?id=xxxx
有人直接将这个地址复制到其他地方，即可盗链，直接利用下载的地址就可以实现下载，而不是通过我自己的网站去下载，这时我们就需要判断下是否是通过我自己的网站点击下载的。
所以我们需要在 down.asp 这个页面做下简单的来源判断。如果不是来自本站的连接，则直接拒绝访问。
String referer = request.getHeader("referer");//获取请求头，判断这个头是否为空，或这个头的首地址是否为http://localhost，如果不是则为盗链。

##### 处理编码问题
服务器响应时的数据，即服务器向浏览器传递的数据的编码格式由服务器决定：
编码时使用的编码表，使用getByte("编码表")设置，或者使用response.setCharacterEncoding(编码表)设置。两者的区别在于 ，前者设置字节流码表，后者设置字符流码表
解码时使用的编码表，使response.setHeader("ContentType","text/html;charset=utf-8")指定http响应头来设置。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/20161206213759060.png)<br>

客户端发送请求时的乱码解决:<br>
GET提交，参数在URL中，设置URL的解码配置，服务器默认使用IOS-8855-1拉丁码表解码URL，我们可以通过如 tomcat/config/server.xml配置文件中:
``` <Connector
     connectionTimeout="20000"
     port="8080"
     protocol="HTTP/1.1"
     redirectPort="8443"/>
```
添加属性URIEncoding="UTF-8"即可将服务器默认的解码url的方式设置为utf-8

或者在doget方法中:
将接收的乱码文字使用新的码表转换：
```
String name = request.getParamter("name"); // 获取乱码文字
byte[] bs = name.getBytes("IOS-8859-1"); // 根据乱码码表，将文字转换为字节数组
String s = new String(bs, "UTF-8"); // 将字节数组按照新的码表解码，生成文字
```
![](https://github.com/TrueOr/java/raw/master/java_web/picture/20161208171242792.png)<br>

POST提交：
与GET提交解码的区别：解码事件不同，GET因为参数在URL中，所以服务器一旦接受请求就会立刻解码参数，而POST在Servlet调用获取参数的方法时才会解码。
所以，解决post请求的乱码很简单，只需要在参数调用前使用request.setCharacterEncoding("utf-8"),设置请求解码表即可。


##### 请求转发
![](https://github.com/TrueOr/java/raw/master/java_web/picture/20161208173848590.png)<br>
浏览器发送请求，servlet处理request和response部分业务，但是无法全部处理，也无法简单的显示到页面中，所以将处理过得request和response发送给jsp，jsp进行一些业务操作，并响应给浏览器展示。
AServlet doGet():
```
//todo AServlet进行业务处理，比如从数据库获取数据
// 处理结束后：=>
// 发送给Bservlet
BServlet bServlet = new BServlet();
bServlet.doGet(request, response);

//*****标准转发
// 使用request域(一个请求内有效，主要用于请求转发) 保存数据库的信息，发送给BServlet，即 AServlet和BServlet使用request域共享数据，request域是request对象中的一个Map。
request.setAttribute("name","Feathers");  // 向request域中存入一个键值对
request.getRequestDispatcher("/servlet/BServlet").forword(request.response);
```
BServlet doGet()：
```
// todo 负责输出显示
// 从request域中取出值
System.out.println((String)request.getAttribute("name"));
```
注意：
不能在转发的Servlet中向浏览器输出任何响应正文的内容，但是可以添加响应头。
因为Servlet中，即使你添加了响应体，也会被清空。

##### 重定向和转发的区别
    * 重定向是一次请求，转发一次请求
    * 重定向可以访问项目之外的地址，而转发不能。
    * 重定向不可以使用request域，而转使用request域，一次请求，一个request对象
    * 重定向地址栏可能会发生会发生变化，而转发一定不会
    * 重定向请求方式有可能发生改变（重定后的请求一定是get请求，只有显示设置post才是post请求，否则是get请求），而转发不会
    * 重定向时response的方法（通过修改response头来完成重定向），而请求转发是resquest的方法（一次请求，转发给不同的Servlet）

##### 请求包含
请求转发，转发的Servlet不能修改请求体，而请求包含中，同请求转发类似，但是可以修改请求体。
用途：用来解决重复操作。
将重复的操作提取到这个Servlet中，统一进行处理。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/20161208182400985.png)<br>
往往在多个JSP中使用，一个jsp用于处理相同的操作，其余的jsp根据是否需要处理，进行依次处理。

实现：
AServlet doGet()：
```
response.setContentType("text/html;charset=utf-8");
// 包含
request.getRequestDispatcher("/servlet/BServlet").include(request,response);
response.getWriter().write("Aservlet 处理");
```
BServlet doGet():
```
response.getWriter().write("Bservlet 处理");
```
结果：
`
Aservlet 处理Bservlet 处理
`


### ---------------------------------------------------------------------------------------------------


### ----------------------------------------Response对象------------------------------------------------
HttpServletResponse对象代表服务器的响应。这个对象中封装了向客户端发送数据、发送响应头，发送响应状态码的方法。<br>
##### 设置响应状态码
    * setStatus(int sc)：设置正常的响应状态码 status code
    * setStatus(int sc， String sm)：设置正常的响应状态码,状态码描述 status message,过时，因为正常状态下，状态码信息不会显示给用户，所以没有必要设置
    * sendError(int sc):设置错误的状态码
    * sendError(int sc, String sm)：设置错误的状态码,包含错误信息
##### 设置响应头
    * setHeader(String name, String value):设置一个键值对，值为string
    * setDateHeader(String name, long date)：设置一个键值对，值为long，long常用于毫秒的表示
    * setIntHeader(String name, int value)：设置一个键值对，值为int类型
    * setHeader(String name, String value):添加一个键值对，值为string
    * setDateHeader(String name, long date)：添加一个键值对，值为long，long常用于毫秒的表示
    * setIntHeader(String name, int value)：添加一个键值对，值为int类型
    * add 和 set 区别在于，前置直接添加（key是可以重复的），后者会修改原来的，没有才会添加。
    响应头实例：
        ContentType:text/html;charset=utf-8 设置页面内容是html编码格式是utf-8
        Refresh:5;url=https://www.baidu.com 5秒后跳转
        html中meta标签的作用就是用于向响应头中添加信息。
##### 设置响应正文
    response.getWriter()
    response.getOutputStream()
getOutputStream和getWriter方法分别用于得到输出二进制数据、输出文本数据的ServletOuputStream、Printwriter对象,但是两则不能结合在一起使用,原因是：<br>
两个对象的流不一样,getOutputStream：方法用于返回Servlet引擎创建的字节输出流对象,getWriter：返回Servlet引擎创建的字符输出流对象,
这两个方法互斥，调用了其中的一个方法之后，就不能在调用另外一个。<br>
Serlvet的service方法结束后，Servlet引擎将检查getWriter或getOutputStream方法返回的输出流对象是否已调用过close方法，如果没有，Servlet引擎将调用close方法关闭该输出流对象。

##### 实现重定向
请求重定向指：一个web资源收到客户端请求后，通知客户端去访问另外一个web资源，这称之为请求重定向。地址栏会变，并发送2次请求，增加服务器负担。
实现方式：response.sendRedirect("http://www.baidu.com")
实现原理：<br>
    在响应头中添加302状态码，告诉浏览器需要进行重定向:response.setStatus(302),在响应头中添加Location;<br>
    指定重定向的位置:response.setHeader("Location", "http://www.baidu.com");
##### 给浏览器传递一个图片
```
// 获取图片输入流
 InputStream is = getServletContext().getResourceAsStream("/WEB-INFO/xxxx.jpg");
 // 获取浏览器的输出流
 byte[] buffer = new byte[1024];
 // 将图片篇输入流写出到浏览器中
 int len = -1;
 while((len = is.readBuffer(buffer)) != -1){
     os.write(buffer, 0, len);
     os.flush();
 }
 ```
 ##### 实现文件下载
 ```
 protected void doGet(HttpServletRequest req, HttpServletResponse resp)
             throws ServletException, IOException {
         this.doPost(req, resp);
     }


     protected void doPost(HttpServletRequest req, HttpServletResponse response)
             throws ServletException, IOException {
         //获取到下载资源的绝对路径getRealPath("e://aa/aa.jpg");
         String path = "e://aa/aa.jpg";
         String filename = path.substring(path.lastIndexOf("\\")+1);

         //对于中文文件名，如果想正常，一定要经过url编码
         response.setHeader("content-disposition", "attachment;filename=" + URLEncoder.encode(filename, "UTF-8"));

         FileInputStream in = new FileInputStream(path);
         OutputStream out = response.getOutputStream();
         try {
             int len = 0;
             byte buffer[] = new byte[1024];
             while ((len = in.read(buffer)) > 0) {
                 out.write(buffer, 0, len);
             }
         } finally {
             if (in != null)
                 try {
                     in.close();
                 } catch (Exception e) {
                 }
             ;
             if (out != null)
                 try {
                     out.close();
                 } catch (Exception e) {
                 }
             ;
         }
```

