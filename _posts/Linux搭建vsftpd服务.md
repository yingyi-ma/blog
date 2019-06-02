---
title: Linux搭建vsftpd服务
date: 2019-04-19 14:16:43
tags: Linux
---

#### 介绍

>vsftpd是 “very secure FTP deamon”的缩写，是一个完全免费，开源的ftp服务器软件。

#### 特点

小巧轻快，安全易用，支持虚拟用户、支持带宽限制等功能。

#### 背景

安装环境:centos7.2
安装工具:yum

##### 1.检测是否已经安装vsftp

```
[root@localhost ~]# rpm -qa | grep vsftpd

vsftpd-3.0.2-22.el7.x86_64
```

如果显示了vsftpd版本号说明安装了vsftpd

##### 2.安装vsftp

```
yum -y install vsftpd
```
配置文件默认路径：
 /etc/vsftpd/vsftpd.conf
 
 ##### 3.创建虚拟用户
 创建ftp目录
```
 mkdir /ftpfile
```
 ###### 创建用户
```
 useradd ftpuser -d /ftpfile -s /sbin/nologin
```
参数说明：
-d 意为该用户应该使用的文件路径  

 -s 意为该创建的用户没有登录linux系统的权限
 
 ###### 给ftp目录授权给新创建用户
 
```
chown -R ftpuser.ftpuser /ftpfile  
```
 

参数说明 -R 注意，是大写的R，意为遍历参数中所有的路径，统统都赋权限给ftpuser

 ###### 给创建用户设置密码
```
passwd ftpuser
```


##### 4.配置
```
cd /etc/vsftpd
sudo vim chroot_list
```
把刚才新增的虚拟用户（ftpuser）添加此配置文件中，后续要引用

```
sudo vim /etc/selinux/config
```
修改为SELINUX=disabled
:wq保存退出

注：如果一会验证的时候碰到550欲绝访问请执行：
sudo setsebool -P ftp_home_dir 1
然后重启linux服务器，执行reboot命令

```
vim /etc/vsftpd/vsftpd.conf  
```

```
chroot_local_user=NO # 防止FTP用户访问上级（安全问题）
chroot_list_enable=YES # 添加虚拟用户名单生效
```

#### 问题
###### 响应:  500 OOPS: vsftpd: refusing to run with writable root inside chroot()

```
vim /etc/vsftpd/vsftpd.conf
# 添加一行配置 allow_writeable_chroot=YES
```

###### 启停命令:
```
systemctl start vsftpd.service
systemctl restart vsftpd.service
systemctl stop vsftpd.service
systemctl status vsftpd.service
```

