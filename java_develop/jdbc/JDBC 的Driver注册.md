​	

#  JDBC 的Driver注册



首先我们来看看com.mysql.jdbc.Driver()的源代码

由源代码可以看出在mysql的Driver类中有一个静态代码块，静态代码块中已经通过DriverManager注册了这个驱动！

大家都知道静态代码快是在类加载器加载这个类的字节码文件的时候就已经执行了的，也就是说，在DriverManager.registerDriver(new com.mysql.jdbc.Driver())这段代码中，一旦new了这个驱动，这个驱动就已经被加载了！

所以如果DriverManager.registerDriver(new com.mysql.jdbc.Driver())，实际上Mysql的Driver会被加载两次！！

所以只要new com.mysql.jdbc.Driver();实际上就已经注册驱动了！

但是我们能new com.mysql.jdbc.Driver()这样子来注册驱动吗？可以！但是还是不提倡，为什么呢？因为这样的话还有一个问题，这样写十分依赖Jar包，一旦jar包找不到，编译时期就会报错。所以在开发过程中通常写成:

``` monkey
Class.forName(“com.mysql.jdbc.Driver”);
```


这样的话，通过类加载的方式来加载Driver类，照样能够执行static代码块中的注册驱动的方法。而且由于将Driver的位置写成了字符串的形式，对jar包的依赖就降低了，也易于使用配置文件加载这个类，所以下次如果换成连接Oracle或者其他数据库，改一下配置文件再填一个jar包就可以了。

