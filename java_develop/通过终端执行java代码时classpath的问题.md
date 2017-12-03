# 通过终端执行java代码时classpath的问题

有的时候需要在终端或者dos的操作界面通过调用javac 和java 命令来编译和运行java文件，这个时候就涉及到java 的搜索路径的问题；

我们知道，java的搜索路径实在环境变量classpath中设置的，但是有时候需要临时在某一个文件夹下运行java命令，没有必要跑到classpath的地方永久的设置环境变量，通过java -cp参数就可以解决；

```
-cp 和 -classpath 一样，是指定类运行所依赖其他类的路径，通常是类库，jar包之类，需要全路径到jar包，window上分号“;”  
```

使用方法：

```
java -cp .;myClass.jar packname.mainclassname
```

如果要添加多个搜索的路径，每一个路径之间要用分隔符区分开，windows的分隔符是“;”；

这里同时说一下，如果你的java文件中已经将该文件放到某个package中了，那么再执行的时候需要将该文件放置在package 执行的文件路径下面，及将package 中设定路径中的 `.`修改为实际的路径，然户在该package的基路径下执行java + packagename.classname 的限定名执行文件；

总结：

1. 要将编译过的class文件放到package指定的文件路径下；
2. 在输入 java 命令时，需要使用完整的packagename.classname的形式；



参考资料：

[1]http://ivan0513.iteye.com/blog/982445

[2]http://blog.csdn.net/liang0000zai/article/details/50516002