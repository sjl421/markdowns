# D13安装centos7.2说明文档

## 1.D13机型硬盘配置

- 12块800G的SSD

## 2.安装方案

- 2块SSD做RAID 1 （group0）
- 10块SSD做RAID 5 （group1）
- 操作系统安装到group0的SSD上

## 3.安装步骤

#### 3.1启动盘准备

- 大于4G的U盘，里面写入centos7.2的iso文件

#### 3.2进入RAID BIOS配置

1. 开机，在LSI mega RAID界面按CTRL+H进入WEBBIOS ![img1](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios1.jpg)
2. WEBBIOS界面如下，点击start ![img2](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios2.jpg)
3. 如下可显示磁盘的logic view，点击Configuration Wizard进入RAID配置 ![img3](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios3.jpg)
4. 选择New Configuration（提示会清掉原来硬盘中的数据） ![img4](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios4.jpg)
5. 确认之后选择Manual Configuration进行手动配置 ![img5](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios5.jpg)
6. 在如下界面，点击选择左侧列表中的前两块硬盘，按CTRL进行多选 ![img6](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios6.jpg)
7. 点击Add To Array，添加到group0 ![img7](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios7.jpg)
8. 点击Accept DG，出现如下界面，点击Next ![img8](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios8.jpg)
9. 点击Add to SPAN，添加到SPAN盘 ![img9](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios9.jpg)
10. 点击Next，出现如下界面，在RAID Level中选择RAID级别，这里选择RAID 1 ![img10](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios10.jpg)
11. 点击update size，更新虚拟磁盘大小 ![img11](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios11.jpg)
12. 点击Accept同意RAID组建立，之后点击BACK进行剩下10块SSD的RAID5组建
13. 同理添加10块SSD到group1，配置组建成RAID 5阵列
14. 最终组建完成如图所示，group0是2块SSD的RAID 1，group1是10块SSD的RAID 5 ![img12](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios12.jpg)
15. 重启使配置生效

#### 3.3 进入BIOS从U盘引导

1. 重启后在启动界面按F2/DEL，进入BIOS启动项选择
2. 查看是否检测到USB引导
3. 如果没有，进入BIOS设置
4. 在boot选项卡下设置UEFI Boot为enable
5. 设置启动顺序为从USB引导优先 ![img13](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios13.jpg) ![img14](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios14.jpg)
6. 保存设置并重启从USB引导 ![img15](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios15.jpg)
7. 出现如下界面开始安装操作系统 ![img16](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios16.jpg)

#### 3.4安装centos7.2操作系统

1. 选择install centos
2. 选择分区，如下图，安装操作系统到容量为800G的group0中的SSD中 ![img17](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios17.jpg)
3. 其他配置参见centos7.2的安装教程
4. 安装完成后重启

#### 3.5检查RAID卡

- 如下图在开机引导时可见RAID 1和RAID 5已经组建完成 ![img18](https://gitlab03.dtdream.com/BigData/DTCubeADB/raw/master/doc/modules/attachment/webbios1.jpg)
- 也可以在webbios下查看RAID卡信息
- 进入centos操作系统后使用如下命令可以查看软RAID信息 `cat /proc/mdstat`
- 使用如下命令查看RAID的厂商、型号、级别等，无法查看各个硬盘信息 `cat /proc/scsi/scsidmesg|grep -i raid`

## 4.安装中可能遇到的问题

#### 4.1无法检测到U盘

- 在安装界面按E/TAB进入编辑命令，将原来的启动目录修改成 `vmlinuz initrd=initrd.img linux dd quiet`
- 会让你选择盘符，记住U盘的盘符，如sda4，重启系统在安装界面按E/TAB进入编辑命令，修改成如下设备位置，按CTRL+X继续安装 `vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sda4 quiet`

#### 4.2启动顺序

- 如果在BIOS配置中指定了启动顺序，要改为优先从硬盘启动(因为操作系统安装在了group0的硬盘中)

## 5.备注

- group0的硬盘在安装操作系统时显示大小为800G左右，因为group0做了RAID 1，一块硬盘用来备份，故空间显示为800G
- group1的硬盘显示大小为7.2T左右，因为group1做了RAID 5，一块硬盘用来做校验盘，故空间显示为7.2T