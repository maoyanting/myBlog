---
title: Git-win10下Git闪退
date: 2018-03-19 20:09:16
categories: 
- Git
tags:
- Git
---

win10下Git闪退

今天第一天上班，装Git就遇到了很大的问题……忧伤

<!--more-->

配置：win10专业版  Git 2.5.2

下载 Git for windows，安装后打开Git Bash闪退，打开Git GUI显示

![https://pic-1256291953.cos.ap-shanghai.myqcloud.com/WechatIMG304.jpeg](https://pic-1256291953.cos.ap-shanghai.myqcloud.com/WechatIMG304.jpeg)

先介绍下网上找到的方法

>参考链接：[win10 专业版出现 open /dev/null](https://answers.microsoft.com/zh-hans/windows/forum/windows_10-hardware/win10/59c6662f-188d-49f3-bbbd-67b70d79f4d8)

***

### 1、启动非即插即用驱动程序

右击微软徽标，选择设备管理器，选择显示隐藏的设备，然后找到非即插即用驱动程序，展开，找到目录下面的Null，右键点击Null-属性-驱动程序-如果已经启动则点击停止，再点击启动，类型选择为启动。

>我没有非即插即用驱动程序……
>
>这里可以参考：[git报错 fatal:open /dev/null or dup failed: No such file or directory ](http://blog.csdn.net/itachi85/article/details/47172375)

***

### 2、修复操作

在C:\Windows\System32位置，找到cmd，以管理员运行cmd，粘贴输入sfc /scannow命令，进行修复操作。

![](https://pic-1256291953.cos.ap-shanghai.myqcloud.com/WechatIMG312.png)

>修复到40%左右的时候跳掉了……

***

### 3、替换null.sys

这里是有可能你的null.sys有问题，找一个新的null.sys替换一下就行

>然而并没有……

***

### 4、使用低版本的Git

安装成功后能打开，但是还是显示no such file or directory

![1](https://pic-1256291953.cos.ap-shanghai.myqcloud.com/WechatIMG309.jpeg)

***

### 5、更新你的win10

如小标题，没啥用

***

### 6、重设null的启动配置

1. 以管理员模式运行cmd
2. 如下

`sc config Null start= system`
`sc start Null`

3. 关闭cmd并重启电脑

参考资料：[Null Service Defaults in Windows 10](http://servicedefaults.com/10/null/)

***

### 解决方案

我先替换了null.sys，然后修复操作，最后重设null的启动配置，重启，成功。







