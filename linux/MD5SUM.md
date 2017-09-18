# MD5SUM

md5sum 使用生成文件的md5值的命令,同时他也可以校验文件和它对应的md5值是否符合;

1. 使用:

```shell
md5sum filename > filename.md5
###
[root@adb_ftp md5]# md5sum hello > md5
[root@adb_ftp md5]# ls
hello  md5  sun
[root@adb_ftp md5]# cat md5 
63f45e40258c7ff37c1a08c1cc89a1f2  hello


## 同时计算多个文件的md5
[root@adb_ftp md5]# md5sum  hello sun > md5
[root@adb_ftp md5]# ls
hello  md5  sun
[root@adb_ftp md5]# cat md5 
63f45e40258c7ff37c1a08c1cc89a1f2  hello
e4955d532dfb2ba5134962c2e8f0dc78  sun
```



### 同时md5sum也可以用来验证文件和给出的md5值是否符合

```shell
hellO 和 sun 是对应的文件, 文件md5中记录的该文件的md5的值;

[root@adb_ftp md5]# md5sum -c md5 
hello: OK
sun: OK
#检验结果显示md5值和文件符合;

# 当修改hello文件后
[root@adb_ftp md5]# md5sum -c md5 
hello: FAILED
sun: OK
md5sum: WARNING: 1 computed checksum did NOT match	#文件和md5值已经对不上了;
```



