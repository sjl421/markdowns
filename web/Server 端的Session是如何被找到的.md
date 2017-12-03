# Server 端的Session是如何被找到的

关于Session有几个问题需要搞明白：

* Session是什么？
* Session的做什么用？
* Session是如何工作的？

并且关于这几个问题都能延伸出很多比较小的问题。

其中的一个小问题是：

Session是存贮在Server端的，那么浏览器（客户端）处的请求所对应的session是如何在server处被找到？

说起session就要说起cookie（想要搞懂这个问题可以去搜索：session和cookie的关系，搞明白联系和区别）

浏览器对server的每次请求都会带着cookie（加入cookie没有被禁用），其中cookie中有一项信息是JSSESSIONID（一下简称JSID），browser的每次请求在到达最终的controller之前都会被tomcat处理，tomcat首先会根据请求cookie中的JSID的值在当前的管理的Session的内存中（HashMap）查找当前ID的Session是否存在，

（以下是我个人的猜测，尚未找到相关的资料确认）：

当发现hashmap中有对应id的session存在时，就将其从hashmap中读取出来装载进request中，然后(中间的具体省略），最终交代给controller，我们就能从controller中通过request.getSession()得到当前requeset的session。











延伸问题：

1. Session ID是如何共server 返回给browser的；

2. Session 和 cookie的关系；

3. Session共享；

4. Session是如何被管理的？

   ## Session 是如何被管理的

   结论： Session是存贮在ServletContext中的（以tomcat为例），即session是存贮在tomcat的内存中的，以CocurrentHashMap的存储形式保存；