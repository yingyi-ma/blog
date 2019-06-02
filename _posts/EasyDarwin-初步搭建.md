---
title: EasyDarwin 初步搭建
date: 2019-05-10 16:18:57
tags: 框架
keywords: EasyDarwin
description:
---
### 前言
**EasyDarin**是一款基于**RTSP**协议的**流媒体服务器**开源框架。

#### 什么是流媒体


>**流媒体（streaming media）** 是指将一连串的媒体数据压缩后，经过网上分段发送数据，在网上即时传输影音以供观赏的一种技术与过程，此技术使得数据包得以像流水一样发送；如果不使用此技术，就必须在使用前下载整个媒体文件。
流媒体文件一般定义在bit层次结构，因此流数据包并不一定必须按照字节对齐，虽然通常的媒体文件都是按照这种字节对齐的方式打包的。流媒体的三大操作平台是微软公司、RealNetworks、苹果公司提供的。

#### RTSP 协议



>**RTSP（Real Time Streaming Protocol）**，RFC2326，实时流传输协议，是TCP/IP协议体系中的一个应用层协议，由哥伦比亚大学、网景和RealNetworks公司提交的IETF RFC标准。该协议定义了一对多应用程序如何有效地通过IP网络传送多媒体数据。RTSP在体系结构上位于RTP和RTCP之上，它使用TCP或UDP完成数据传输。HTTP与RTSP相比，HTTP请求由客户机发出，服务器作出响应；使用RTSP时，客户机和服务器都可以发出请求，即RTSP可以是双向的。RTSP是用来控制声音或影像的多媒体串流协议，并允许同时多个串流需求控制，传输时所用的网络通讯协定并不在其定义的范围内，服务器端可以自行选择使用TCP或UDP来传送串流内容，它的语法和运作跟HTTP 1.1类似，但并不特别强调时间同步，所以比较能容忍网络延迟。而前面提到的允许同时多个串流需求控制（Multicast），除了可以降低服务器端的网络用量，更进而支持多方视讯会议（Video Conference）。因为与HTTP1.1的运作方式相似，所以代理服务器〈Proxy〉的快取功能〈Cache〉也同样适用于RTSP，并因RTSP具有重新导向功能，可视实际负载情况来转换提供服务的服务器，以避免过大的负载集中于同一服务器而造成延迟。
![](/images/04_easydarwin/0.png)
### [EasyDarwin](http://www.easydarwin.org/) - 最简单的流媒体平台框架


EasyDarwin是一款高性能开源RTSP流媒体服务器，基于go语言研发，维护和优化：RTSP推模式转发、RTSP拉模式转发、录像、检索、回放、关键帧缓存、秒开画面、RESTful接口、WEB后台管理、分布式负载均衡，基于EasyDarwin构建出了一套基础的流媒体云视频平台架构！


EasyDarwin 核心流媒体服务！RTSP开源流媒体服务，高效、稳定、可靠、功能齐全，支持RTSP流媒体协议，支持安防行业需要的摄像机流媒体转发功能、支持互联网行业需要的多平台(PC、Android、IOS)RTSP直播（H264/MJPEG/MPEG4、AAC/PCMA/PCMU/G726）功能，底层(Select/Epoll网络模型、无锁队列调度)和上层(RESTful接口、WEB管理、多平台编译)、关键帧索引（秒开画面）、远程运维等方面优化，这些都是全代码完全开源的，具体接口调用方法和流程见：[https://github.com/EasyDarwin/EasyDarwin](https://github.com/EasyDarwin/EasyDarwin)；


#### EasyDarwin推流过程
![](/images/04_easydarwin/1.png)



#### 部署环境

>1. EasyDarwin安装
>2. ffmpeg安装
>3. 配置文件（EasyDarwin与Ffmpeg关联、启动开关等）

##### EasyDarwin安装
下载地址（解压即用）
[https://github.com/EasyDarwin/EasyDarwin/releases](https://github.com/EasyDarwin/EasyDarwin/releases) 
![](/images/04_easydarwin/2.png)




##### ffmpeg安装
下载地址（解压即用）
[http://www.ffmpeg.org/download.html](http://www.ffmpeg.org/download.html) 

linux
![](/images/04_easydarwin/3.png)
windows
![](/images/04_easydarwin/4.png)


##### 配置文件（EasyDarwin与Ffmpeg关联、启动开关等）
![](/images/04_easydarwin/5.png)

```
; 打开本地转储（1为开启 0为关闭）
save_stream_to_local=1 

;easydarwin使用ffmpeg工具来进行存储。这里表示ffmpeg的可执行程序的路径
ffmpeg_path=C:\Users\Administrator\Desktop\EasyDarwin\ffmpeg-20190508-06ba478-win64-static\bin\ffmpeg

;本地存储所将要保存的根目录。如果不存在，程序会尝试创建该目录。
m3u8_dir_path=C:\Users\Administrator\Desktop\EasyDarwin\EasyDarwin-windows-8.1.0-1901141151Downloads\EasyDarwinGoM3u8

```


#### 测试
>1. ffmpeg 推流（直播）
>2. ffmpeg 拉流（接收直播）
>3. 查看回放目录
>4. 查看回放文件
>5. 查看回放

在本系统ffmpeg根目录下执行以下命令

**测试MP4文件**：
[http://cdn-media.jingzong.org/mp4/R01/R01-001/R01-001-0040.mp4](http://cdn-media.jingzong.org/mp4/R01/R01-001/R01-001-0040.mp4)

##### ffmpeg 推流
文件路径：R01-001-0040.mp4
协议类型：tcp
vcodec：h264
rtsp地址（默认本机IP端口号554、test是自定义名称）：rtsp://192.168.3.20:554/test
```
ffmpeg -re -i R01-001-0040.mp4 -rtsp_transport tcp -vcodec h264 -f rtsp rtsp://192.168.3.20:554/test
```
![](/images/04_easydarwin/6.png)
![](/images/04_easydarwin/7.png)


##### ffmpeg 拉流
根目录下执行播放
```
ffplay rtsp://192.168.3.20:554/test
```
![](/images/04_easydarwin/8.png)




##### 查看录像文件夹
```
http://192.168.3.20:10008/api/v1/record/folders
```
![](/images/04_easydarwin/9.png)


##### 查看录像文件
```
http://192.168.3.20:10008/api/v1/record/files?folder=t10
```
![](/images/04_easydarwin/10.png)


##### 查看回放
播放生成的切分文件
```
http://192.168.3.20:10008/record/t10/20190510/out0.ts
```

#### 存储文件的播放兼容性处理以及自定义配置
有些流保存成m3u8后，无法在浏览器直接播放，比如H265的流。这时候就需要转码了。ffmpeg提供的转码功能可解决这个问题。
EasyDarwin在存储时，相当于调用ffmpeg指令如下：

```
ffmpeg -fflags genpts -rtsp_transport tcp -i rtsp://localhost:554/stream_265 -c:v copy -c:a copy -hls_time 6 -hls_list_size 0 /Users/ze/Downloads/EasyDarwinGoM3u8/stream_265/20190106/out.m3u8
```
注意这里的-c:v和-c:a参数使用了copy，相当于没有转码。当需要转码时，我们把这两个字段去掉就可以了。
方法是在配置文件里面的rtsp小节，加上一个配置项，key为拉流的自定义路径，值为default,如下所示：

```
;ffmpeg转码格式，比如可设置为-c:v copy -c:a copy，表示copy源格式；default表示使用ffmpeg内置的输出格式，可能需要转码
/stream_265=default
```

这时，再进行存储，ffmpeg的指令变为：

```
ffmpeg -fflags genpts -rtsp_transport tcp -i rtsp://localhost:554/stream_265 -hls_time 6 -hls_list_size 0 /Users/ze/Downloads/EasyDarwinGoM3u8/stream_265/20190106/out.m3u8

```
去掉了-c:v copy -c:a copy字段，ffmpeg存储时就开始转码了。这样会提高了存储文件的兼容性，但是同时会提高cpu的占用率。






参考来源：[https://blog.csdn.net/jyt0551/article/details/84189498](https://blog.csdn.net/jyt0551/article/details/84189498)
