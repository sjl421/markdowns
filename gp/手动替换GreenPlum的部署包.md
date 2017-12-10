# 手动替换GreenPlum的部署包

1. 修改 ~/.bashrc

```shell
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
# gp的二进制文件的位置
export MASTER_DATA_DIRECTORY=/home/gpadmin/data/master/gpseg-1
# 指定greenplum包的路径
source /usr/local/adb_core/greenplum_path.sh
```

2. 补齐git上没有的python包

在已经部署好的环境上copy以下几个包,文件的位置时:源码目录/lib/python文件夹下:

```
Crypto
figleaf
yaml
在 pygresql 文件夹下创建空的 __init__.py 文件
```

3. 将部署包拷贝到所有的物理机节点上相同的位置


