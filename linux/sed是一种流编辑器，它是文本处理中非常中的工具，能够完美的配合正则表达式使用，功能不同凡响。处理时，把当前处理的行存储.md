sed是一种流编辑器，它是文本处理中非常中的工具，能够完美的配合正则表达式使用，功能不同凡响。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等。 sed的选项、命令、替换标记 命令格式 sed [options] 'command' file(s) sed [options] -f scriptfile file(s) 选项 -e

```

```





```
将 this is a test line 追加到以test开头的行前面： 
sed '/^test/i\this is a test line' file 
在test.conf文件第5行之前插入this is a test line： sed -i '5i\this is a test line' test.conf
```

