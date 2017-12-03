# PostgreSql gpfdist 导入导出

## gpfdist 介绍

gpfdist是linux系统中的一个工具，详情请百度



这其中需要涉及到的权限包括：

insert，select和create external table (read and write)的权限；



## 导入

## 1. 启动gpfdist

用户提前配置好gpfdist；

配置好放置好源文件；

### 2. 创建read 权限的外部表

用户通过命令创建外部表，此时需要用like 来创建外部表，表示外部表和最终的目的表是一致的；

### 3. 拷贝数据 

执行数据拷贝命令（其实就是一条insert语句）

### 4. 在导入完成之后就可以删除外部表了



## 导出

### 1. 和导入操作的第一步一样的；

### 2. 创建具有写权签的外部表

### 3. 执行语句

具体是什么语句还需要进一步了解；

![粘贴图片](E:\DingShare\粘贴图片.png)