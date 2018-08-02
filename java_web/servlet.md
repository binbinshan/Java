#### 名词解释
servlet容器:
* Servlet规范定义了一个API标准，这一标准的实现通常称为Servlet容器，比如开源的Tomcat、JBoss。web容器准确的说应该叫web服务器，它是来管理和部署web应用的。<br>

应用服务器:
* 它的功能比web服务器要强大的多，因为它可以部署EJB应用，可以实现容器管理的事务，一般的应用服务器 有weblogic和websphere等，它们都是商业服务器，功能强大但都是收费的。
#### 什么是servlet？
Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。
由其来处理请求和发送响应，使用Servlet可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170214192940050-671180063.png)<br>

#### servlet请求过程
![](https://github.com/TrueOr/java/raw/master/java_web/picture/20140119021418375.jpg)<br>
* Web服务器接收到HTTP请求
* Web服务器将请求转发给servlet容器
* 如果容器中不存在所需的servlet，容器就会检索servlet，并将其加载到容器的地址空间中
* 容器调用servlet的init()方法对servlet进行初始化（该方法只会在servlet第一次被载入时调用）
* 容器调用servlet的service()方法来处理HTTP请求，即，读取请求中的数据，创建一个响应。servlet会被保留在容器的地址空间中，继续处理其他的HTTP请求
* Web服务器将动态生成的结果返回到正确的地址。

#### servlet生命周期
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216103737254-1072057229.png)<br>
* 服务器启动时(web.xml中配置load-on-startup=1，默认为0)或者第一次请求该servlet时，就会初始化一个Servlet对象，也就是会执行初始化方法init(ServletConfig conf)
* 该servlet对象去处理所有客户端请求，在service(ServletRequest req，ServletResponse res)方法中执行
* 最后服务器关闭时，才会销毁这个servlet对象，执行destroy()方法。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/1352204684_1548.jpg)<br>

#### 第一个servlet
* 创建一个MyServlet继承HttpServlet，重写doGet和doPost方法。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216094438629-1196159083.png)<br>
* 在web.xml中配置MyServlet
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216094000972-1276129522.png)<br>
* 浏览器是通过我们配置的信息来找到对应的servlet
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216094053504-915571176.png)<br>
按照步骤，首先浏览器通过http://localhost:8080/test01/MyServlet 来找到web.xml中的url-pattern，这就是第一步，匹配到了url-pattern后，就会找到第二步servlet的名字MyServlet，知道了名字，就可以通过servlet-name找到第三步，到了第三步，也就能够知道servlet的位置了。然后到其中找到对应的处理方式进行处理。 

#### 为什么创建的servlet是继承自httpServlet，而不是直接实现Servlet接口？
首先httpServlet继承GenericServlet(通用servlet),而GenericServlet实现了Servlet接口和ServletConfig接口。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/servlet-servletconfig.PNG)<br>

Servlet接口有生命周期的三个关键方法，init、service、destroy。还有另外两个方法，一个getServletConfig()方法来获取ServletConfig对象，ServletConfig对象可以获取到Servlet的一些信息，ServletName、ServletContext、InitParameter、InitParameterNames，通过servletConfig接口就可以看出。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216142610660-322166979.png)
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216143011050-806963452.png)<br>
其中ServletContext对象是servlet上下文对象，功能有很多，获得了ServletContext对象，就能获取大部分我们需要的信息，比如获取servlet的路径，等方法。

Servlet接口中的内容和作用，总结起来就是，三个生命周期运行的方法和获取ServletConfig，而通过ServletConfig又可以获取到ServletContext。而GenericServlet实现了Servlet接口后，也就说明我们可以直接继承GenericServlet，就可以使用上面我们所介绍Servlet接口中的那几个方法了，能拿到ServletConfig，也可以拿到ServletContext，不过那样太麻烦，不能直接获取ServletContext，所以GenericServlet除了实现Servlet接口外，还实现了ServletConfig接口，那样，就可以直接获取ServletContext了。<br>
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216145502191-411577839.png)<br>
#### servlet的生命周期中，可以看出，执行的是service方法，为什么我们就只需要写doGet和doPost方法呢？
在GenericServlet中有一个抽象方法service(ServletRequest req, ServletResponse res)。在GenericServlet类中并没有实现该内容，那么我们想到的是，在它上面肯定还有一层，也就是还有一个子类继承它，实现该方法，要是让我们自己写的Servlet继承GenericServlet，需要自己写service方法，那岂不是累死，并且我们可以看到，service方法中的参数还是ServletRequest，ServletResponse。并没有跟http相关对象挂钩，所以我们接着往下面看。<br>
HttpServlet继承了GenericServlet类，通过我们上面的推测，这个类主要的功能肯定是实现service方法的各种细节和设计。并且通过类名可以知道，该类就跟http挂钩了。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216152220082-1786372762.png)<br>
service(ServletRequest req, ServletResponse res)方法
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216152409566-1602415910.png)<br>
该方法中就做一件事情，就是将ServletRequest和ServletResponse这两个对象强转为HttpServletRequest和HttpServletResponse对象，转换为httpServletRequest和HttpServletResponse对象之后。<br>
在调用service(HttpServletRequest req, HttpServletResponse resp)方法,这个方法就是判断浏览器过来的请求方式是哪种，每种的处理方式不一样，我们常用的就是get，post，并且，我们处理的方式可能有很多的内容，所以，在该方法内会将get，post等其他5种请求方式提取出来，变成单个的方法，然后我们需要编写servlet时，就可以直接重写doGet或者doPost方法就行了，而不是重写service方法，更加有针对性。所以这里就回到了我们上面编写servlet时的情况，继承httpServlet，而只要重写两个方法，一个doGet，一个doPost，其实就是service方法会调用这两个方法中的一个(看请求方式)

#### ServletConfig ServletContext
##### ServletConfig对象
获取途径：getServletConfig();<br>
  getServletName();  //获取servlet的名称，也就是我们在web.xml中配置的servlet-name 
  getServletContext(); //获取ServletContext对象，该对象的作用看下面讲解
  getInitParameter(String); //获取在servlet中初始化参数的值。这里注意与全局初始化参数的区分。这个获取的只是在该servlet下的初始化参数
  ![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170216195140550-371826071.png)<br>
  getInitParameterNames(); //获取在Servlet中所有初始化参数的名字，也就是key值，可以通过key值，来找到各个初始化参数的value值。注意返回的是枚举类型
  ![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170217091747144-810654839.png)<br>
  ![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170217091914597-1623014325.png)<br>
  ![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170217091935300-1981582648.png)<br>
注意：在上面我们所分析的源码过程中，我们就知道，其实可以不用先获得ServletConfig，然后在获取其各种参数，可以直接使用其方法，比如上面我们用的ServletConfig().getServletName();可以直接写成getServletName();而不用在先获取ServletConfig();了，原因就是在GenericServlet中，已经帮我们获取了这些数据，我们只需要直接拿就行。

##### ServletContext对象
获取途径：getServletContext(); getServletConfig().getServletContext();<br>　　
第一种是直接拿，在GenericServlet中已经帮我们用getServletConfig().getServletContext();拿到了ServletContext。我们只需要直接获取就行了，第二种就相当于我们自己在获取一遍，两种读是一样的。<br>
功能：tomcat为每个web项目都创建一个ServletContext实例，tomcat在启动时创建，服务器关闭时销毁，在一个web项目中共享数据，管理web项目资源，为整个web配置公共信息等，通俗点讲，就是一个web项目，就存在一个ServletContext实例，每个Servlet读可以访问到它。<br>

* web项目中共享数据，getAttribute(String name)、setAttribute(String name, Object obj)、removeAttribute(String name),
、setAttribute(String name, Object obj) 在web项目范围内存放内容，以便让在web项目中所有的servlet读能访问到。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/874710-20170217100049254-1938338032.png)<br>
* 整个web项目初始化参数 //这个就是全局初始化参数，每个Servlet中都能获取到该初始化值。getInitPatameter(String name)//通过指定名称获取初始化值 getInitParameterNames()//获得枚举类型 web.xml配置整个web项目的初始化。
![](https://github.com/TrueOr/java/raw/master/java_web/picture/initparment.PNG)<br>
* 获取web项目资源<br>
获取web项目下指定资源的路径：getServletContext().getRealPath("/WEB-INF/web.xml")<br>
获取web项目下指定资源的内容，返回的是字节输入流。InputStream getResourceAsStream(java.lang.String path)<br>






