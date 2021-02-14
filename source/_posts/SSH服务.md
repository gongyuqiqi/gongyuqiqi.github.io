---
title: SSH服务
date: 2020-02-24 
tags: Linux
---

远程控制服务：配置sshd服务、 安全密钥验证、远程传输命令  
不间断会话服务：管理远程会话、会话共享功能

<!--more-->

# 远程控制服务
- linux中一切皆是文件，配置服务即是对配置文件进行修改
- 主配置文件：最重要的配置文件  
  一般情况下在/etc/服务名称/服务名称.config  
- 普通文件：主要被调用，常规参数
  

## （一）配置sshd服务

**想要使用SSH协议来远程管理Linux系统，则需要部署配置sshd服务程序。**
> 一般的服务程序并不会在配置文件修改之后立即获得最新的参数。如果想让新配置文件生效，则需要手动**重启相应的服务程序**。最好也**将这个服务程序加入到开机启动项**中，这样系统在下一次启动时，该服务程序便会自动运行，继续为用户提供服务。  


1. 查看主配置文件  

```bash
cat /etc/ssh/sshd_config
```

2. Xshell远程登陆服务器,此时是需要密码验证的。 

```R
ssh 192.168.12.136
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/PermitRootLogin_yes.JPG"/>



3. 修改主配置文件内容如下，使得不可以通过root用户方式远程登陆服务器。

```bash
vim /etc/ssh/sshd_config
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/beforemodification.JPG"/>
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/modification.JPG"/>

4. 在此远程连接服务器，发现依然可以通过root用户方式远程登陆服务器。

```bash
ssh 192.168.12.64
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/PermitRootLogin_no_without_systemctl_restart_sshd.JPG"/>


5. **重启服务**

```bash
#这里以sshd服务为例
systemctl restart sshd
systemctl enable sshd
```

6. 再次进行远程登陆

```R
ssh 192.168.12.64
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/PermitRootLogin_no.JPG"/>

每次输入密码后，会再次弹出界面框要求输入密码，进入死循环。。。。。。

## （二）安全密钥验证

1. 客户端主机中生成“密钥对”

```bash
ssh-keygen
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/ssh-keygen.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sshkeygenfile.JPG"/>


2. 把客户端主机生成的**公钥文件**传送至远程主机

```bash
ssh-copy-id 116.63.129.7
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/ssh-copy-id.JPG"/>

3. 查看服务器端认证的文件

```
cat /root/.ssh/authorized_keys
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/cat_root_.ssh_authorized_key.JPG"/>

4. 客户端登陆服务端

```bash
ssh 116.63.129.7
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/ssh116.JPG"/>

此时就会发现，客户端远程访问服务器就**不需要密码**登陆了。因为密钥的优先级是高于密码的。这样是不是很方便呢？嘿嘿😁 

而且，**密钥方式登陆**更安全哦！😊

5. 修改**服务器端配置文件**，使其只支持密钥方式登陆

```bash
#注意啦，一定要是服务器端的配置文件
vim /etc/ssh/sshd_config
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/server_passwordauthentication_yes.JPG"/>
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/server_passwordauthentication_no.JPG"/>

6. 重启相应服务

```bash
systemctl restart sshd
```

7. Xshell远程登陆服务器（验证）

```bash
ssh 116.63.129.7
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/xshell116with_sysytemctl___.JPG"/>

登陆界面默认无法输入密码！

**注意啦，有对应私钥的客户端无需密码即可登陆拥有对应公钥的服务端**

## 远程传输命令

scp（secure copy）是一个基于SSH协议在网络之间进行安全传输的命令。

1. 将客户端文件传给服务器端

```bash
#一下操作在客户端进行
echo "this is a test about scp">scptest
scp /root/scptest 116.63.129.7:/home
```
2. 进入服务器端进行查看

```bash
cat /home/scptest
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scptest.JPG"/>


3. 将服务器端文件传给客户端

```bash
#以下操作在服务器端进行
echo "This is the message from server">fromserver
```

4. 进入客户端进行文件下载和查看

```bash
scp 116.63.129.7:home/fromserver /root
cat /root/fromserver
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scpservertohost.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/catfromserver.JPG"/>


# 不间断会话服务

**[首先配置yum仓库](https://www.linuxprobe.com/chapter-09.html#92)**

## 管理远程会话

screen命令能做的事情非常多：可以用-S参数创建会话窗口；用-d参数将指定会话进行离线处理；用-r参数恢复指定会话；用-x参数一次性恢复所有的会话；用-ls参数显示当前已有的会话；以及用-wipe参数把目前无法使用的会话删除

1. screen服务介绍

```bash
#创建一个会话框test
screen -S test
```

```bash
#查看系统当前的会话
screen -ls
```

```bash
#退出当前会话
exit
```

2. screen实战一

```bash
#创建TEST会话框,会发现屏幕闪跳了以下
screen -S TEST
```

```bash
ls
cd /bin
cd /roo/biosoft
ls
```
当前界面如下
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/screenTEST1.JPG"/>

此时将终端kill掉，模拟远程连接过程中网络出错

```bash
screen -ls
screen -r TEST #恢复离线的对话框
```

此时就会恢复到上一次发生事故时的界面

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/screenTEST2.JPG"/>

3. screen实战二

有时候在编辑文件时，文件内容还未及时保存，远程连接出现问题。这个时候screen服务就会🌟发光发热🔥

```bash
screen vim test.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/vim.JPG"/>

此时将终端kill掉，模拟远程连接过程中网络出错

```bash
screen -ls
screen -r 10777
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/screen-r10777.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/vim.JPG"/>

## 会话共享功能

1. 终端A远程连接服务器+创建会话
   
   ssh服务将终端A远程连接到服务器（这里假装连接了，其实就是连接到本地的虚拟机，为了方便展示）

```bash
#连接
ssh 192.168.12.136
#创建会话框
screen -S test
```

2. 终端B远程连接服务器+同步

```bash
ssh 192.168.12.136
screen -x
```
此后，终端A和终端B就同步画面啦

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/synchronization.JPG"/>


## 参考资料

[ssh配置文件ssh_config和sshd_config区别](https://www.cnblogs.com/xiaochina/p/5802008.html)

[远程控制服务_文字版](https://www.linuxprobe.com/chapter-09.html#92)

[远程控制服务_视频版](https://www.bilibili.com/video/av71365003?p=12)

