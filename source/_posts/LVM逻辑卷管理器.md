---
title: LVM逻辑卷管理器
date: 2020-02-08 16:30:23
tags: Linux
---

磁盘管理（进阶篇）

<!--more-->

# LVM逻辑卷管理器  
- 前面的磁盘管理（基础篇）介绍的磁盘管理技术虽然能够有效地提高硬盘设备的读写速度以及数据的安全性，但是在硬盘分好区后，再想修改硬盘分区大小就不容易了。当用户想要随着实际需求的变化调整硬盘分区的大小时，会受到硬盘“灵活性”的限制。这时就需要用到另外一项非常普及的硬盘设备资源管理技术了—**LVM（逻辑卷管理器）**。LVM可以允许用户对硬盘资源进行动态调整。  
- LVM技术是在硬盘分区和文件系统之间添加了一个逻辑层，它提供了一个抽象的卷组，可以把多块硬盘进行卷组合并。这样一来，用户不必关心物理硬盘设备的底层架构和布局，就可以实现对硬盘分区的动态调整。  
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/LVMstructure.JPG"/>
**PV,Physical Volume**  
**VG,Volume Group**  
**LV,Logical Volume** 

- 物理卷处于LVM中的最底层。卷组建立在物理卷之上，一个卷组可以包含多个物理卷，而且在卷组创建之后也可以继续向其中添加新的物理卷。逻辑卷是用卷组中空闲的资源建立的，并且逻辑卷在建立后可以动态地扩展或缩小空间。这就是LVM的核心理念。 

## 部署逻辑卷  
该实验在VMware中进行，首先**拍摄快照**，以便在实验结束后恢复原有状态。  
**依次创建物理卷PV、卷组VG、逻辑卷LV**  

1. 在虚拟机中添加两块新硬盘设备。我们先对这两块新硬盘进行创建物理卷的操作，可以将该操作简单理解成让硬盘设备支持LVM技术。

```bash
fdisk -l #此时可以看出多出了/dev/sdb /dev/sdc的信息
pvcreate /dev/sdb /dev/sdc #让硬盘设备支持LVM技术
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pvcreate.JPG"/>


2. 把两块硬盘设备加入到newvg卷组中，然后查看卷组状态  

```bash
vgcreate newvg /dev/sdb /dev/sdc
vgdisplay
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/vgcreate.JPG"/>
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/vgdispaly.JPG"/>

3. 切割出一个约160M的逻辑卷LV  
> 在对逻辑卷进行切割时有两种计量单位。第一种是以容量为单位，所使用的参数为-L。例如，使用-L 150M生成一个大小为150MB的逻辑卷。另外一种是以基本单元的个数为单位，所使用的参数为-l。每个基本单元的大小默认为4MB。例如，使用-l 37可以生成一个大小为37×4MB=148MB的逻辑卷。

```bash
lvdisplay #查看当前逻辑卷情况
lvcreate -n newlv -l 40 newvg
lvdisplay #创建逻辑卷后查看逻辑卷情况
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/lvcre.dis..JPG"/>

4. 把生成的逻辑卷格式化，然后挂载使用
> Linux系统会把LVM中的逻辑卷设备存放在/dev设备目录中（实际上是做了一个符号链接），同时会以卷组的名称来建立一个目录，其中保存了逻辑卷的设备映射文件（即/dev/卷组名称/逻辑卷名称）。  

```bash
mkfs.xfs /dev/newvg/newlv
mkdir /newlv
mount /dev/newvg/newlv /newlv
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/lvmkfsmount.JPG"/>

5. 查看挂载状态，并写入配置文件  

```bash
df -h
echo "/dev/newvg/newlv /newlv xfs defaults 0 0" >> /etc/fstab
df -h
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/df_h.JPG"/>

## 扩容逻辑卷   
> 用户在使用存储设备时感知不到设备底层的架构和布局，更不用关心底层是由多少块硬盘组成的，只要卷组中有足够的资源，就可以一直为逻辑卷扩容。扩展前请一定要记得卸载设备和挂载点的关联。  
 
```bash
umount /newlv
```

1. 把上诉逻辑卷newlv扩展至30G  

```bash
lvextend -L 30G /dev/newvg/newlv
mount -a
df -h
#xfs文件系统需要执行xfs_growfs操作，需要先挂载，所以上面先执行了mount -a命令
xfs_growfs /dev/newvg/newlv
df -h
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/lvextend.JPG"/>
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/lvextendresult.JPG"/>

**注意**  
<font color=blue>使用 resize2fs或xfs_growfs 对挂载目录在线扩容</font>  
<font color=blue>resize2fs 针对文件系统ext2 ext3 ext4</font>  
<font color=blue>xfs_growfs 针对文件系统xfs</font>

**xfs文件系统是不支持缩减逻辑卷操作，所以为了演示缩小逻辑卷操作，先将xfs文件系统重新格式化成ext4文件系统** 

1. ext4文件系统的扩容操作  

```bash
umount /newlv
mkfs.ext4 /dev/newvg/newlv
vim /etc/fstab #将/dev/newvg/newlv对应的xfs改成ext4,改变文件系统类型并不会影响磁盘上的数据
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/trans..JPG"/>

```bash
lvextend -L 35G /dev/newvg/newlv
e2fsck -f /dev/newvg/newlv #检查硬盘完整型，针对ext4执行
mount -a
df -h
resize2fs /dev/newvg/newlv
df -h
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/ext4lvextend.JPG"/>





## 缩小逻辑卷  
**执行缩容前一定确保最终的大小要大于现有数据所占的空间**  
执行缩容操作前记得先把文件系统卸载掉  

```bash
umount /newlv
```

1. 把逻辑卷newlv的容量减小到100M  

```bash
umount /newlv
df -h
e2fsck -f /dev/newvg/newlv
resize2fs /dev/newvg/newlv 5G  
lvreduce -L 5G /dev/newvg/newlv ##奇怪，我之前运行的时候没有加此操作，依然能够成功
mount -a
df -h
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/ext4lvreduce.JPG"/>

## 逻辑卷快照  
可以对某一个逻辑卷设备做一次快照，如果日后发现数据被改错了，就可以利用之前做好的快照卷进行覆盖还原。LVM的快照卷功能有两个特点：   
> 快照卷的容量必须等同于逻辑卷的容量；  
 快照卷仅一次有效，一旦执行还原操作后则会被立即自动删除。

查看卷组信息  
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/vgdisplaysnap1.JPG"/>   
卷组输出结果显示已经使用5G的容量，空闲容量还有34.99GB.  

```bash
echo "This is a test about LVNSnap" > /LVMtest/file  
```

1. 使用-s参数生成一个快照卷，使用-L参数指定切割的大小。另外，还要在命令后面写上是针对哪个逻辑卷执行的快照操作。  

```bash
lvcreate -L 5G -s -n SNAP /dev/volumegroup/logicalvolume
vgdispaly
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/vgdisplaysnap2.JPG"/>

2. 在逻辑卷所挂载的目录中创建一个100MB的垃圾文件，然后再查看快照卷的状态。可以发现存储空间占的用量上升了。  

```bash
dd if=/dev/zero of=/LVMtest/FILE count=1 bs=100M
vgdisplay
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/vgdisplaysnap3.JPG"/>

3. 为了校验SNAP快照卷的效果，需要对逻辑卷进行快照还原操作。在此之前记得先卸载掉逻辑卷设备与目录的挂载

```bash
df -h
umount /LVWtest
lvconvert --merge /dev/volumegroup/SNAP 
mount -a
ls /LVMtest 
#此时我们之前创建的100M大小的垃圾文件FILE不见了，因为这个文件是LVM快照之后创建的文件
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SNAPlvm.JPG"/>


## 删除逻辑卷  
**依次删除逻辑卷、卷组、物理卷设备，顺序不可颠倒**  

```bash
umount /newlv
vim /etc/fstab #删除掉/dev/newvg/newlv相关信息
lvremove /dev/newvg/newlv
vgremove newvg
pvremove /dev/sdb /dev/sdc
#执行下面的命令分别查看LV、VG、PV的情况，发现/dev/newvg/newlv相关信息消失
lvdisplay
vgdisplay
pvdisplay
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/removelvm.JPG"/>



## 参考资料
https://blog.csdn.net/qq_33932782/article/details/76612965