#cgdb安装使用

cgdb很好使，分为代码窗口和命令窗口，可以同步看到执行的代码，且有代码高亮。

也可以打开某个文件看代码，比gdb的list好使得多。

项目主页：<http://cgdb.github.io/>

常用命令：
ESC：切换焦点到源码模式，在该界面中可以使用vi的常用命令
i：切换焦点到gdb模式
o：打开文件对话框，选择要显示的代码文件，按ESC取消
空格：在当前行设置一个断点

更多参考使用手册：<http://cgdb.github.io/docs/cgdb.html>

源码编译安装，源码包见附件
$ ./configure --prefix=/usr/local
$ make
$ sudo make install
如果./configure报错configure: error: Please install makeinfo before installing
则安装texinfo
yum install -y texinfo 