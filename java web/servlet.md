#### 名词解释
servlet容器:
* Servlet规范定义了一个API标准，这一标准的实现通常称为Servlet容器，比如开源的Tomcat、JBoss。web容器准确的说应该叫web服务器，它是来管理和部署web应用的。<br>

应用服务器:
* 它的功能比web服务器要强大的多，因为它可以部署EJB应用，可以实现容器管理的事务，一般的应用服务器 有weblogic和websphere等，它们都是商业服务器，功能强大但都是收费的。
#### 什么是servlet？
Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。
由其来处理请求和发送响应，使用Servlet可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。
![](https://github.com/TrueOr/java/raw/master/java web/picture/874710-20170214192940050-671180063.png)<br>

#### servlet请求过程
![](https://github.com/TrueOr/java/raw/master/java web/picture/20140119021418375.jpg)<br>
* Web服务器接收到HTTP请求
* Web服务器将请求转发给servlet容器
* 如果容器中不存在所需的servlet，容器就会检索servlet，并将其加载到容器的地址空间中
* 容器调用servlet的init()方法对servlet进行初始化（该方法只会在servlet第一次被载入时调用）
* 容器调用servlet的service()方法来处理HTTP请求，即，读取请求中的数据，创建一个响应。servlet会被保留在容器的地址空间中，继续处理其他的HTTP请求
* Web服务器将动态生成的结果返回到正确的地址。

#### servlet生命周期
![](https://github.com/TrueOr/java/raw/master/java web/picture/874710-20170216103737254-1072057229.png)<br>
* 服务器启动时(web.xml中配置load-on-startup=1，默认为0)或者第一次请求该servlet时，就会初始化一个Servlet对象，也就是会执行初始化方法init(ServletConfig conf)
* 该servlet对象去处理所有客户端请求，在service(ServletRequest req，ServletResponse res)方法中执行
* 最后服务器关闭时，才会销毁这个servlet对象，执行destroy()方法。
![](https://github.com/TrueOr/java/raw/master/java web/picture/1352204684_1548.jpg)<br>

#### 第一个servlet
* 创建一个MyServlet继承HttpServlet，重写doGet和doPost方法。
![](https://github.com/TrueOr/java/raw/master/java web/picture/874710-20170216094438629-1196159083.png)<br>
* 在web.xml中配置MyServlet
![](https://github.com/TrueOr/java/raw/master/java web/picture/874710-20170216094000972-1276129522.png)<br>
* 浏览器是通过我们配置的信息来找到对应的servlet
![](https://github.com/TrueOr/java/raw/master/java web/picture/874710-20170216094053504-915571176.png)<br>
按照步骤，首先浏览器通过http://localhost:8080/test01/MyServlet 来找到web.xml中的url-pattern，这就是第一步，匹配到了url-pattern后，就会找到第二步servlet的名字MyServlet，知道了名字，就可以通过servlet-name找到第三步，到了第三步，也就能够知道servlet的位置了。然后到其中找到对应的处理方式进行处理。 



