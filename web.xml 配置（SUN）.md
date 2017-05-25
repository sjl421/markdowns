# web.xml 配置（SUN）

## 启动

当要启动某个j2ee项目时，服务器软件或容器如（tomcat）会第一步加载项目中的web.xml文件，通过其中的各种配置来启动项目，只有其中配置的各项均无误时，项目才能正确启动。web.xml有多项标签，在其加载的过程中顺序依次为：context-param >> listener >> fileter >> servlet。（同类多个节点以出现顺序依次加载）

1、web.xml先读取context-param和listener这两种节点；

2、然后容器创建一个ServletContext(上下文)，应用于整个项目；

3、容器会将读取到的context-param转化为键值对并存入servletContext；

4、根据listener创建监听；

5、容器会读取，根据指定的类路径来实例化过滤器；

6、此时项目初始化完成；

7、在发起第一次请求是，servlet节点才会被加载实例化。

## web.xml 配置详解

1. `<context-param>`

* context-param节点是web.xml中用于配置应用于整个web项目的上下文,

```xml
<context-param>  
	<param-name>contextConfigLocation</param-name>  
  <!-- spring 配置文件所在位置，启动 spring 时会去该路径下查找该配置文件 -->
	<param-value>
      classpath:applicationContext.xml
      classpath:mvc-dispatcher-servlet.xml
      classpath:spring-datasource.xml
  </param-value> 
</context-param>  
```

web.xml中配置spring必须使用listener节点，但context-param节点可有可无，如果缺省则默认contextConfigLocation路径为“/WEB-INF/applicationContext.xml”；如果有多个xml文件，使用”,“分隔,可以使用上面的那样一个文件一行的方式；

2. `ContextLoaderListener`

* spring的上下文监听器配置，`ContextLoaderListener`实现了`ServletContextListener`接口，当容器加载时启动spring容器。`ServletContextListener`在`contextInitialized`方法中初始化spring容器。有几种办法可以加载spring容器，通过在web.xml的`<context-param>`标签中配置spring的applicationContext.xml路径，文件名可以任意取，如果没有配置，将在/WEB-INF/路径下查找默认的applicationContext.xml文件。

```xml
<listener>
   <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

listener 节点说明：

解释一： 事件监听，js里应用广泛，各种事件函数的实现，Android和java se也是广泛的应用，各种点击事件的监听。当触发某个事件时，会触发监听在该事件上的所有监听器。spring 的 `ContextLoaderListener `就是实现了 ServletContextListener 接口的监听器，该监听器会在容器（tomcat，jetty）启动的时候触发，然后就可以启动 spring 相应的配置信息。

解释二： 为web应用程序定义监听器，监听器用来监听各种事件，比如：application和session事件，所有的监听器按照相同的方式定义，功能取决去它们各自实现的接口，常用的Web事件接口有如下几个：

* ServletContextListener：用于监听Web应用的启动和关闭；

* ServletContextAttributeListener：用于监听ServletContext范围（application）内属性的改变；
* ServletRequestListener：用于监听用户的请求；
* ServletRequestAttributeListener：用于监听ServletRequest范围（request）内属性的改变；
* HttpSessionListener：用于监听用户session的开始和结束；
* HttpSessionAttributeListener：用于监听HttpSession范围（session）内属性的改变。

配置Listener只要向Web应用注册Listener实现类即可，无序配置参数之类的东西，因为Listener获取的是Web应用ServletContext（application）的配置参数。为Web应用配置Listener的两种方式：

1. 使用@WebListener修饰Listener实现类即可。![图片描述](http://img.mukewang.com/56dfacec0001cff506090068.png)
2. 在web.xml文档中使用进行配置。![图片描述](http://img.mukewang.com/56dfacf60001246f05970096.png)

> 以上`ContextLoaderListener` 和`ContextLoaderListener`的配置隶属于spring容器的初始化；



**Spring 如何处理请求（servlet ）**

一个HTTP请求路径根据web.xml配置的拦截路径匹配后会被相应的servlet处理（在处理之前会被配置的过滤器处理），在这个servlet中能够拿到请求的数据信息，然后进行相应的处理，处理完成后再响应给浏览器。 spring 的`org.springframework.web.servlet.DispatcherServlet`就是一个 servlet，不过这个 servlet 是 spring 自己实现的，它处理的请求路径在 servlet-mapping 下的 url-pattern 中进行配置，配置完成后会将所有该配置拦截到的请求交给 spring 的 DispatcherServlet 进行处理，这个 spring 核心的 servlet 我将它理解为一个路由的作用，它会将拦截到的请求根据请求路径和请求方式进一步的分发下去，分发到 spring 的 @Controller 下的`@RequestMapping(value={"/xxx"}, method=RequestMethod.xox )`下的方法下进行处理。



**spring容器的核心servlet，拦截的请求路径**

servlet即配置所需用的servlet，用于处理及响应客户的请求。容器的Context对象对请求路径(URL)做出处理，去掉请求URL的上下文路径后，按路径映射规则和Servlet映射路径（）做匹配，如果匹配成功，则调用这个Servlet处理请求。

创建Servlet实例有两个时机：

1. 客户端第一次请求某个Servlet时，系统创建该Servlet的实例，大部分Servlet都是这种Servlet。

2. Web应用启动时立即创建Servlet实例，即load-on-start Servlet；

   说明：的内容可以为空，或者是一个整数。这个值表示由Web容器载入内存的顺序。举个例子：如果有两个Servlet元素都含有子元素，则子元素值较小的Servlet将先被加载。如果子元素值为空或负值，则由Web容器决定什么时候加载Servlet。如果两个Servlet的子元素值相同，则由Web容器决定先加载哪一个Servlet。 1表示启动容器时，初始化Servlet。

spring的核心servlet配置，该servlet会将在这里配置拦截的路径转发到spring的controller拦截的路径进行处理，这个servlet相当于一个spring的路由中心，将spring拦截的请求对应的转发下去进行处理。

```xml
<!-- spring 核心转发器，拦截指定目录下的请求，分配到配置的拦截路径下处理 -->
<servlet>
   <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
     <param-name>contextConfigLocation</param-name>
       <param-value>/META-INF/spring-servlet.xml</param-value>
   </init-param>
   <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
   <servlet-name>dispatcher</servlet-name>
   <!-- -->
   <url-pattern>/</url-pattern>
</servlet-mapping>
```

解释二:

```xml
<!--使用Spring MVC,配置DispatcherServlet是第一步。DispatcherServlet是一个Servlet,,所以可以配置多个DispatcherServlet-->  
<!--DispatcherServlet是前置控制器，配置在web.xml文件中的。拦截匹配的请求，Servlet拦截匹配规则要自已定义，把拦截下来的请求，依据某某规则分发到目标Controller(我们写的Action)来处理。-->  
<servlet>  
  <servlet-name>DispatcherServlet</servlet-name>
  <!--在DispatcherServlet的初始化过程中，框架会在web应用的 WEB-INF文件夹下寻找名为[servlet-name]-servlet.xml 的配置文件，生成文件中定义的bean。-->  
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
  <!--指明了配置文件的文件名，不使用默认配置文件名，而使用dispatcher-servlet.xml配置文件。-->  
  <init-param>  
    <param-name>contextConfigLocation</param-name>  
    <!--其中<param-value>**.xml</param-value> 这里可以使用多种写法-->  
    <!--1、不写,使用默认值:/WEB-INF/<servlet-name>-servlet.xml-->  
    <!--2、<param-value>/WEB-INF/classes/dispatcher-servlet.xml</param-value>-->  
    <!--3、<param-value>classpath*:dispatcher-servlet.xml</param-value>-->  
    <!--4、多个值用逗号分隔-->  
    <param-value>classpath:spring/dispatcher-servlet.xml</param-value>  
  </init-param>  
  <load-on-startup>1</load-on-startup><!--是启动顺序，让这个Servlet随Servletp容器一起启动。-->  
</servlet>  
<servlet-mapping>  
<!--这个Servlet的名字是dispatcher，可以有多个DispatcherServlet，是通过名字来区分的。每一个DispatcherServlet有自己的WebApplicationContext上下文对象。同时保存的ServletContext中和Request对象中.-->  
<!--ApplicationContext是Spring的核心，Context我们通常解释为上下文环境，我想用“容器”来表述它更容易理解一些，ApplicationContext则是“应用的容器”了:P，Spring把Bean放在这个容器中，在需要的时候，用getBean方法取出-->  
	<servlet-name>DispatcherServlet</servlet-name>  
<!--Servlet拦截匹配规则可以自已定义，当映射为@RequestMapping("/user/add")时，为例,拦截哪种URL合适？-->  
<!--1、拦截*.do、*.htm， 例如：/user/add.do,这是最传统的方式，最简单也最实用。不会导致静态文件（jpg,js,css）被拦截。-->  
<!--2、拦截/，例如：/user/add,可以实现现在很流行的REST风格。很多互联网类型的应用很喜欢这种风格的URL。弊端：会导致静态文件（jpg,js,css）被拦截后不能正常显示。 -->  
	<url-pattern>/</url-pattern> <!--会拦截URL中带“/”的请求。-->  
</servlet-mapping>  
```

**对静态资源的访问规则**

```xml
<!--如果你的DispatcherServlet拦截"/"，为了实现REST风格，拦截了所有的请求，那么同时对*.js,*.jpg等静态文件的访问也就被拦截了。-->  
<!--方案一：激活Tomcat的defaultServlet来处理静态文件-->  
<!--要写在DispatcherServlet的前面， 让 defaultServlet先拦截请求，这样请求就不会进入Spring了，我想性能是最好的吧。-->  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.css</url-pattern>  
</servlet-mapping>  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.swf</url-pattern>  
</servlet-mapping>  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.gif</url-pattern>  
</servlet-mapping>  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.jpg</url-pattern>  
</servlet-mapping>  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.png</url-pattern>  
</servlet-mapping>  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.js</url-pattern>  
</servlet-mapping>  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.html</url-pattern>  
</servlet-mapping>  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.xml</url-pattern>  
</servlet-mapping>  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.json</url-pattern>  
</servlet-mapping>  
<servlet-mapping>  
    <servlet-name>default</servlet-name>  
    <url-pattern>*.map</url-pattern>  
</servlet-mapping>  
```

**Spring filter配置方法**

filter节点配置过滤器，主要用于对用户请求request进行预处理，也可以对Response进行后处理，是个典型的处理链。
使用Filter的完整流程是：Filter对用户请求进行预处理，接着将请求HttpServletRequest交给Servlet进行处理并生成响应，最后Filter再对服务器响应HttpServletResponse进行后处理。Filter与Servlet具有完全相同的生命周期，且Filter也可以通过来配置初始化参数，获取Filter的初始化参数则使用FilterConfig的getInitParameter()。

一个HTTP请求就是一次浏览器客户端与服务器的交互，在这次交互中有浏览器向服务器发送数据的过程，还有服务器接收到请求数据后处理完将处理结果返回的过程，当返回结果成功就完成了一次HTTP请求（其中的握手，路由等就不细说了）。在浏览器与服务器一来一回的过程中我们可以做一些事情，例如将请求数据编码方式统一，添加IP校验，session校验等相关servlet处理前的工作，在servlet处理后响应给浏览器客户端的过程中我们也可以进行过滤工作。spring 的`org.springframework.web.filter.CharacterEncodingFilter`就是一个过滤器，它在请求未到达servlet之前将请求编码转换为我们在 `<param-value>UTF-8</param-value>`中配置的编码方式，过滤的路径是 filter-mapping 的 url-pattern 配置的路径。

**Spring 的编码过滤器**

```xml
<!-- spring 编码过滤器 -->
<filter>
   <filter-name>characterEncodingFilter</filter-name>
   <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
   <init-param>
     <param-name>encoding</param-name>
     <param-value>UTF-8</param-value>
   </init-param>
   <init-param>
     <param-name>forceEncoding</param-name>
     <param-value>true</param-value>
   </init-param>
</filter>
<!-- 编码过滤器过滤的路径 -->
<filter-mapping>
   <filter-name>characterEncodingFilter</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>
```

**使用注解配置过滤器**

![56dfad540001855906100171](G:\TempPlace\56dfad540001855906100171.png)

> 需要注意的一点就是filter链过滤的时候是按照filter-mapping先后顺序执行的，所以一般来说会把编码处理的filer放到最前面；

**url-pattern配置讲解**

在 servlet 和 filter 中我们都需要配置 url-pattern，但这个配置的解析规则有哪几种我们接下来就详细的说一下。

1、精确匹配：如 /xxx.html 就只会匹配 xxx.html。

2、路径匹配：如 /xxx/ 会匹配以 xxx 为前缀的 url。

3、后缀匹配：如 .html 会匹配所有以 html 为后缀的 url。

但是对于 url-pattern 的匹配来说可能会存在冲突的情况，这种情况下就需要排个优先级了，以上三者的优先级为 精确匹配 > 路径匹配 > 后缀匹配 。



**其他的一些配置**

```xml
<welcome-file-list><!--指定欢迎页面-->  
     <welcome-file>login.html</welcome-file>  
</welcome-file-list>  
<error-page> <!--当系统出现404错误，跳转到页面nopage.html-->  
    <error-code>404</error-code>  
	<location>/nopage.html</location>  
</error-page>  
<error-page> <!--当系统出现java.lang.NullPointerException，跳转到页面error.html-->  
    <exception-type>java.lang.NullPointerException</exception-type>  
    <location>/error.html</location>  
</error-page>  
<session-config><!--会话超时配置，单位分钟-->  
    <session-timeout>360</session-timeout>  
</session-config>  
```



---

Questions:

#### 1. 加载顺序 

web.xml 文件中一般包括 servlet, spring, filter, listenr的配置。那么他们是按照一个什么顺序加载呢？

加载顺序会影响对spring bean 的调用。

​    比如filter 需要用到 bean ，但是加载顺序是 先加载filter 后加载spring，则filter中初始化操作中的bean为null；

首先可以肯定 加载顺序与他们在web.xml 文件中的先后顺序无关。

web.xml 中 listener 和 serverlet 的加载顺序为 先 listener 后serverlet

最终得出结果：先 listener >> filter >> servlet >>  spring

 所以，如果过滤器中要使用到 bean，可以将spring 的加载 改成 Listener的方式

```xml
<listener>
	<listener-class>
		org.springframework.web.context.ContextLoaderListener
	</listener-class>
</listener>
```

